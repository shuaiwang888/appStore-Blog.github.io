> **文章摘要：本文主要分析Producer核心流程**

本文分析使用kafka-0.10.2.0的源码，书接上文[kafka生产者源码分析2-Producer初始化](http://121.199.72.143:8090/archives/kafka-sheng-chan-zhe-yuan-ma-fen-xi-2-producer-chu-shi-hua)，本篇重点分析Producer核心流程。

## 一、Producer发送流程

### 1.1 整体流程图
![image.png](/upload/2022/01/image-493ab7cc70554d72a97823857a8be0ee.png)

### 1.2 Producer的send方法

```java
@Override
 public Future<RecordMetadata> send(ProducerRecord<K, V> record) {
     return send(record, null);
 }

@Override
 public Future<RecordMetadata> send(ProducerRecord<K, V> record, Callback callback) {
     // intercept the record, which can be potentially modified; this method does not throw exceptions
     // 使用ProducerInterceptor对消息进行拦截或修改
     ProducerRecord<K, V> interceptedRecord = this.interceptors == null ? record : this.interceptors.onSend(record);
     // TODO 核心代码
     return doSend(interceptedRecord, callback);
 }
```

数据发送的最终实现还是调用了Producer的 `doSend()` 接口。

### 1.3 Producer的doSend方法

```java
private Future<RecordMetadata> doSend(ProducerRecord<K, V> record, Callback callback) {
    TopicPartition tp = null;
    try {
        // first make sure the metadata for the topic is available
        /**
         * 步骤一：
         *      同步等待拉取元数据。
         *  maxBlockTimeMs 最多能等待多久。
         */
        ClusterAndWaitTime clusterAndWaitTime = waitOnMetadata(record.topic(), record.partition(), maxBlockTimeMs);
        long remainingWaitMs = Math.max(0, maxBlockTimeMs - clusterAndWaitTime.waitedOnMetadataMs);
        Cluster cluster = clusterAndWaitTime.cluster;
        /**
         * 步骤二：
         *  对消息的key和value进行序列化。
         */
        byte[] serializedKey;
        try {
            serializedKey = keySerializer.serialize(record.topic(), record.key());
        } catch (ClassCastException cce) {
            throw new SerializationException("Can't convert key of class " + record.key().getClass().getName() +
                    " to class " + producerConfig.getClass(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG).getName() +
                    " specified in key.serializer");
        }
        byte[] serializedValue;
        try {
            serializedValue = valueSerializer.serialize(record.topic(), record.value());
        } catch (ClassCastException cce) {
            throw new SerializationException("Can't convert value of class " + record.value().getClass().getName() +
                    " to class " + producerConfig.getClass(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG).getName() +
                    " specified in value.serializer");
        }
        /**
         * 步骤三：
         *  根据分区器选择消息应该发送的分区。
         *
         *  因为前面我们已经获取到了元数据
         *  这儿我们就可以根据元数据的信息
         *  计算一下，我们应该要把这个数据发送到哪个分区上面。
         */
        int partition = partition(record, serializedKey, serializedValue, cluster);
        /**
         *
      * 计算消息记录的总大小
      * Records.LOG_OVERHEAD = SIZE_LENGTH（值为4） + OFFSET_LENGTH（值为8）
      * Records.LOG_OVERHEAD有SIZE_LENGTH和OFFSET_LENGTH两个字段，分别表示存放消息长度和消息偏移量所需要的字节数
      */
        int serializedSize = Records.LOG_OVERHEAD + Record.recordSize(serializedKey, serializedValue);

        /**
         * 步骤四：
         *  确认一下消息的大小是否超过了最大值。
         *  KafkaProdcuer初始化的时候，指定了一个参数，代表的是Producer这儿最大能发送的是一条消息能有多大
         *  默认最大是1M，我们一般都会去修改它。
         */
        ensureValidRecordSize(serializedSize);
        /**
         * 步骤五：
         *  根据元数据信息，封装分区对象
         */
        tp = new TopicPartition(record.topic(), partition);
        long timestamp = record.timestamp() == null ? time.milliseconds() : record.timestamp();
        log.trace("Sending record {} with callback {} to topic {} partition {}", record, callback, record.topic(), partition);
        // producer callback will make sure to call both 'callback' and interceptor callback
        /**
         * 步骤六：
         *  给每一条消息都绑定他的回调函数。因为我们使用的是异步的方式发送的消息。
         */
        Callback interceptCallback = this.interceptors == null ? callback : new InterceptorCallback<>(callback, this.interceptors, tp);
        /**
         * 步骤七：
         *  把消息放入accumulator（32M的一个内存）
         *  然后有accumulator把消息封装成为一个批次一个批次的去发送。
         */
        RecordAccumulator.RecordAppendResult result = accumulator.append(tp, timestamp, serializedKey, serializedValue, interceptCallback, remainingWaitMs);
        // 如果达到批次要求
        if (result.batchIsFull || result.newBatchCreated) {
            log.trace("Waking up the sender since topic {} partition {} is either full or getting a new batch", record.topic(), partition);
            /**
             * 步骤八:
             *  唤醒sender线程。他才是真正发送数据的线程。
             */
            this.sender.wakeup();
        }
        return result.future;
        // handling exceptions and record the errors;
        // for API exceptions return them in the future,
        // for other exceptions throw directly
    } catch (ApiException e) {
        log.debug("Exception occurred during message send:", e);
        if (callback != null)
            callback.onCompletion(null, e);
        this.errors.record();
        if (this.interceptors != null)
            this.interceptors.onSendError(record, tp, e);
        return new FutureFailure(e);
    } catch (InterruptedException e) {
        this.errors.record();
        if (this.interceptors != null)
            this.interceptors.onSendError(record, tp, e);
        throw new InterruptException(e);
    } catch (BufferExhaustedException e) {
        this.errors.record();
        this.metrics.sensor("buffer-exhausted-records").record();
        if (this.interceptors != null)
            this.interceptors.onSendError(record, tp, e);
        throw e;
    } catch (KafkaException e) {
        this.errors.record();
        if (this.interceptors != null)
            this.interceptors.onSendError(record, tp, e);
        throw e;
    } catch (Exception e) {
        // we notify interceptor about all exceptions, since onSend is called before anything else in this method
        if (this.interceptors != null)
            this.interceptors.onSendError(record, tp, e);
        throw e;
    }
}
```

**doSend() 大致流程分为如下的8个步骤：**

1. 确认数据要发送到的 topic 的 metadata 是可用的（如果该 partition 的 leader 存在则是可用的，如果开启权限时，client 有相应的权限），如果没有 topic 的 metadata 信息，就需要获取相应的 metadata；
2. 序列化 record 的 key 和 value；
3. 获取该 record 要发送到的 partition（可以指定，也可以根据算法计算）；
4. 确认一下消息的大小是否超过最大值（默认是1M）；
5. 根据元数据信息，封装分区对象；
6. 给每一个消息都绑定回调函数；
7. 向 accumulator 中追加 record 数据，数据会先进行缓存（默认32M）；
8. 如果追加完数据后，对应的 RecordBatch 已经达到了 batch.size 的大小（或者batch 的剩余空间不足以添加下一条 Record），则唤醒 sender 线程发送数据。

数据发送主要分为上面的八个步骤，下面对着几部分进行消息分析。

## 二、发送过程详解

### 2.1 同步阻塞获取metadata信息

> **如果你要往一个topic里发送消息，必须是得有这个topic的元数据的，你必须要知道这个topic有哪些分区，然后根据Partitioner组件去选择一个分区，然后知道这个分区对应的leader所在的broker，才能跟那个broker建立连接，发送消息。调用同步阻塞的方法，去等待先得获取到那个topic对应的元数据，如果此时客户端还没缓存那个topic的元数据，那么一定会发送网络请求到broker去拉取那个topic的元数据过来，但是下一次就可以直接根据缓存好的元数据来发送了，这块后面讲有一篇文章进行详细讲解。**

### 2.2 对消息的key和value序列化

发送消息的key和value可以是各种各样的类型，比如说String、Double、Boolean，或者是自定义的对象，但是如果要发送消息到broker，必须对这个key和value进行序列化，把那些类型的数据转换成byte[]字节数组的形式。kafka内部提供序列化和反序列化如下：

![image.png](/upload/2022/01/image-3b454a26fbd74c63aeb3408966dc58fb.png)

如果这些序列化不能满足我们的需求，可以自定义序列化和反序列化。

### 2.3 获取分区 partition 值

**关于 partition 值的计算，分为三种情况：**

1. 指明 partition 的情况下，直接将指明的值直接作为 partiton 值；
2. 没有指明 partition 值但有 key 的情况下，将 key 的 hash 值与 topic 的 partition 数进行取余得到 partition 值；
3. 既没有 partition 值又没有 key 值的情况下，第一次调用时随机生成一个整数（后面每次调用在这个整数上自增），将这个值与 topic 可用的 partition 总数取余得到 partition 值，也就是常说的 round-robin 算法。

具体实现如下：

```java
private int partition(ProducerRecord<K, V> record, byte[] serializedKey, byte[] serializedValue, Cluster cluster) {
   // 如果你的这个消息已经分配了分区号，那直接就用这个分区号就可以了
   // 但是正常情况下，消息是没有分区号的。
   Integer partition = record.partition();
   return partition != null ?
        partition :
       // 使用分区器进行选择合适的分区
        partitioner.partition(
                record.topic(), record.key(), serializedKey, record.value(), serializedValue, cluster);
}
```

Producer 默认使用的 partitioner 是 `org.apache.kafka.clients.producer.internals.DefaultPartitioner`，用户也可以自定义 partition 的策略，下面是这个类两个方法的具体实现：

```java
public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
    // 获取集群中指定topic的分区信息
    List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
    int numPartitions = partitions.size();
    if (keyBytes == null) {
        //策略一： 如果发送消息的时候，没有指定key 轮询
        // 获取counter并自增，counter是个原子类
        int nextValue = nextValue(topic);
        // 获取可用分区
        List<PartitionInfo> availablePartitions = cluster.availablePartitionsForTopic(topic);
        if (availablePartitions.size() > 0) {
            int part = Utils.toPositive(nextValue) % availablePartitions.size();
            return availablePartitions.get(part).partition();
        } else {
            // no partitions are available, give a non-available partition
            // 没有可用分区，直接给一个不可用分区
            return Utils.toPositive(nextValue) % numPartitions;
        }
    } else {
        /** 策略二：这个地方就是指定了key
         *  hash the keyBytes to choose a partition
         *  直接对key取一个hash值 % 分区的总数取模
         *  如果是同一个key，计算出来的分区肯定是同一个分区。
         *  如果我们想要让消息能发送到同一个分区上面，那么我们就
         *  必须指定key. 这一点非常重要
         *  murmur2是一种高效率低碰撞的Hash算法
        */
        return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
    }
}   
```

### 2.4 校验消息是否超过最大值

```java
private void ensureValidRecordSize(int size) {
    //如果一条消息的大小超过了 1M，那么就会报错
    if (size > this.maxRequestSize)
        throw new RecordTooLargeException("The message is " + size +
                                          " bytes when serialized which is larger than the maximum request size you have configured with the " +
                                          ProducerConfig.MAX_REQUEST_SIZE_CONFIG +
                                          " configuration.");
    //如果你的一条消息的大小超过32M也会报错。
    if (size > this.totalMemorySize)
        throw new RecordTooLargeException("The message is " + size +
                                          " bytes when serialized which is larger than the total memory buffer you have configured with the " +
                                          ProducerConfig.BUFFER_MEMORY_CONFIG +
                                          " configuration.");
}
```

**maxRequestSize 和 totalMemorySize这两个参数都是可以配置的!**

### 2.5 封装TopicPartition分区对象

就是根据分区封装 `TopicPartition` 对象，不再赘述。

### 2.6 设置回调函数

**设置好自定义的callback回调函数以及对应的interceptor拦截器的回调函数!**

### 2.7 向accumulator 写数据

```java
public RecordAppendResult append(TopicPartition tp,
                                     long timestamp,
                                     byte[] key,
                                     byte[] value,
                                     Callback callback,
                                     long maxTimeToBlock) throws InterruptedException {
        // We keep track of the number of appending thread to make sure we do not miss batches in
        // abortIncompleteBatches().
        // 统计正在向RecordAccumulator中追加数据的线程数
        appendsInProgress.incrementAndGet();
        try {
            // check if we have an in-progress batch

            /**
             * 步骤一：先根据分区找到应该插入到哪个队列里面。
             * 如果有已经存在的队列，那么我们就使用存在队列
             * 如果队列不存在，那么我们新创建一个队列
             *
             * 我们肯定是有了存储批次的队列，但是大家一定要知道一个事
             * 我们代码第一次执行到这儿，获取其实就是一个空的队列。
             *
             * 现在代码第二次执行进来。
             * 假设 分区还是之前的那个分区。
             *
             * 这个方法里面我们之前分析，里面就是针对batchs进行的操作
             * 里面kafka自己封装了一个数据结构：CopyOnWriteMap (这个数据结构本来就是线程安全的)
             */
            Deque<RecordBatch> dq = getOrCreateDeque(tp);
            // 同步操作，以Deque为锁
            /**
             * 假设我们现在有线程一，线程二，线程三
             */
            synchronized (dq) {
                // 检查生产者是否已经关闭了
                //首先进来的是第一个线程
                if (closed)
                    throw new IllegalStateException("Cannot send after the producer is closed.");
                /**
                 * 步骤二：
                 *      尝试往队列里面的批次里添加数据
                 *
                 *      一开始添加数据肯定是失败的，我们目前只是有了队列
                 *      数据是需要存储在批次对象里面（这个批次对象是需要分配内存的）
                 *      我们目前还没有分配内存，所以如果按场景驱动的方式，
                 *      代码第一次运行到这儿其实是不成功的。
                 */
                RecordAppendResult appendResult = tryAppend(timestamp, key, value, callback, dq);
                //第一次进来的时候appendResult的值就为null
                if (appendResult != null)
                    return appendResult;
            }//释放锁

            // we don't have an in-progress record batch try to allocate a new batch
            /**
             * 步骤三：计算一个批次的大小
             * 在消息的大小和批次的大小之间取一个最大值，用这个值作为当前这个批次的大小。
             * 有可能我们的一个消息的大小比一个设定好的批次的大小还要大。
             * 默认一个批次的大小是16K。
             * 所以我们看到这段代码以后，应该给我们一个启示。
             * 如果我们生产者发送数的时候，如果我们的消息的大小都是超过16K，
             * 说明其实就是一条消息就是一个批次，那也就是说消息是一条一条被发送出去的。
             * 那如果是这样的话，批次这个概念的设计就没有意义了
             * 所以大家一定要根据自己公司的数据大小的情况去设置批次的大小。
             */
            int size = Math.max(this.batchSize, Records.LOG_OVERHEAD + Record.recordSize(key, value));
            log.trace("Allocating a new {} byte message buffer for topic {} partition {}", size, tp.topic(), tp.partition());
            /**
             * 步骤四：
             *  根据批次的大小去分配内存
             *
             *  线程一，线程二，线程三，执行到这儿都会申请内存
             *  假设每个线程 都申请了 16k的内存。
             */
            ByteBuffer buffer = free.allocate(size, maxTimeToBlock);
            synchronized (dq) {
                //假设线程一 进来了。
                //线程二就进来了
                // Need to check if producer is closed again after grabbing the dequeue lock.
                if (closed)
                    throw new IllegalStateException("Cannot send after the producer is closed.");
                /**
                 * 步骤五：
                 *      尝试把数据写入到批次里面。
                 *      代码第一次执行到这儿的时候 依然还是失败的（appendResult==null）
                 *      目前虽然已经分配了内存
                 *      但是还没有创建批次，那我们向往批次里面写数据
                 *      还是不能写的。
                 *
                 *   线程二进来执行这段代码的时候，是成功的。
                 */
                RecordAppendResult appendResult = tryAppend(timestamp, key, value, callback, dq);
                //失败的意思就是appendResult 还是会等于null
                if (appendResult != null) {
                    // Somebody else found us a batch, return the one we waited for! Hopefully this doesn't happen often...
                    //释放内存

                    //线程二到这儿，其实他自己已经把数据写到批次了。所以
                    //他的内存就没有什么用了，就把内存个释放了（还给内存池了。
                    free.deallocate(buffer);
                    return appendResult;
                }

                /**
                 * 步骤六：
                 *  根据内存大小封装批次
                 *
                 *  线程一到这儿 会根据内存封装出来一个批次。
                 */
                MemoryRecordsBuilder recordsBuilder = MemoryRecords.builder(buffer, compression, TimestampType.CREATE_TIME, this.batchSize);
                // 使用传入的TopicPartition参数和records新创建一个RecordBatch
                RecordBatch batch = new RecordBatch(tp, recordsBuilder, time.milliseconds());
                //尝试往这个批次里面写数据，到这个时候 我们的代码会执行成功。

                //线程一，就往批次里面写数据，这个时候就写成功了。
                FutureRecordMetadata future = Utils.notNull(batch.tryAppend(timestamp, key, value, callback, time.milliseconds()));
                /**
                 * 步骤七：
                 *  把这个批次放入到这个队列的队尾
                 *
                 *  线程一 把批次添加到队尾
                 */
                dq.addLast(batch);
                incomplete.add(batch);
                return new RecordAppendResult(future, dq.size() > 1 || batch.isFull(), true);
            }
        } finally {
            // 将记录正在追加消息的线程数的计数器减1
            appendsInProgress.decrementAndGet();
        }
    }
```

（1）首先，我们先看 `getOrCreateDeque(tp)` 方法

```java
private Deque<RecordBatch> getOrCreateDeque(TopicPartition tp) {
    //直接从batches里面获取当前分区对应的存储队列
    Deque<RecordBatch> d = this.batches.get(tp);
    if (d != null)
        return d;
    d = new ArrayDeque<>();
    Deque<RecordBatch> previous = this.batches.putIfAbsent(tp, d);
    if (previous == null)
        return d;
    else
        return previous;
}
```

> **多个线程会并发的执行putIfAbsent方法，在这个方法里可以保证线程安全的，除非队列不存在才会设置进去，在put方法的时候是有synchronized，可以保证同一时间只有一个线程会来更新这个值，同时batches 使用自定义并发线程安全的数据结构CopyOnWriteMap<>(),详细代码如下：**

```java
/**
 * A simple read-optimized map implementation that synchronizes only writes and does a full copy on each modification
 *
 * * 1） 这个数据结构是在高并发的情况下是线程安全的。
 *  * 2)  采用的读写分离的思想设计的数据结构
 *  *      每次插入（写数据）数据的时候都开辟新的内存空间
 *  *      所以会有个小缺点，就是插入数据的时候，会比较耗费内存空间。
 *  * 3）这样的一个数据结构，适合写少读多的场景。
 *  *      读数据的时候性能很高。
 *  *
 *  * batchs这个对象存储数据的时候，就是使用的这个数据结构。
 *  * 对于batches来说，它面对的场景就是读多写少的场景。
 *  *
 *  *batches：
 *  *   读数据：
 *  *      每生产一条消息，都会从batches里面读取数据。
 *  *      假如每秒中生产10万条消息，是不是就意味着每秒要读取10万次。
 *  *      所以绝对是一个高并发的场景。
 *  *   写数据：
 *  *     假设有100个分区，那么就是会插入100次数据。
 *  *     并且队列只需要插入一次就可以了。
 *  *     所以这是一个低频的操作。
 */
public class CopyOnWriteMap<K, V> implements ConcurrentMap<K, V> {

    /**
     * 核心的变量就是一个map
     * 这个map有个特点，它的修饰符是volatile关键字。
     * 在多线程的情况下，如果这个map的值发生变化，其他线程也是可见的。
     *
     * get
     * put
     */
    private volatile Map<K, V> map;

    ...省略部分代码

    /**
     * 没有加锁，读取数据的时候性能很高（高并发的场景下，肯定性能很高）
     * 并且是线程安全的。
     * 因为人家采用的读写分离的思想。
     * @param k
     * @return
     */
    @Override
    public V get(Object k) {
        return map.get(k);
    }

    /**
     * 1):
     *      整个方法使用的是synchronized关键字去修饰的，说明这个方法是线程安全。
     *      即使加了锁，这段代码的性能依然很好，因为里面都是纯内存的操作。
     * 2）
     *        这种设计方式，采用的是读写分离的设计思想。
     *        读操作和写操作 是相互不影响的。
     *        所以我们读数据的操作就是线程安全的。
     * 3）
     *      最后把值赋给了map，map是用volatile关键字修饰的。
     *      说明这个map是具有可见性的，这样的话，如果get数据的时候，这儿的值发生了变化，也是能感知到的。
     */
    @Override
    public synchronized V put(K k, V v) {
        // 新的内存空间
        Map<K, V> copy = new HashMap<K, V>(this.map);
        // 插入操作
        V prev = copy.put(k, v);
        this.map = Collections.unmodifiableMap(copy);
        return prev;
    }
}
```

（2）整体流程采用分段加锁的设计，提高并发能力

（3）采用内存池设计BufferPool

**既可以循环使用内存，又可以减少GC的次数。对应的流程图为：**

![image.png](/upload/2022/01/image-e0f8dda4c99040ceb617062dac8043ba.png)

### 2.8 唤醒Sender线程

唤醒sender线程，主要用来负责发送数据。后面我们再详细的分析。

## 三、最后

**从Producer 的 send 方法可以学到非常多的东西：**

- Producer初始化没有去拉取集群的元数据，而是在后面根据你发送消息时候的需要，要给哪个topic发送消息，再去拉取那个topic对应的元数据，这就是懒加载的设计思想，按需加载思想;
- 一个高并发、高吞吐、高性能的消息系统的客户端的设计，他的核心流程是如何来搭建和设计的，可扩展，通过拦截器的模式预留一些扩展点给其他人来扩展，在消息发送之前或者发送之后，都可以进行自定义的扩展;
- 设计一个通用框架的时候，必须得有一个序列化的过程，因为key和value可能是各种各样的类型，但是必须要保证把key和value都转换成通用的byte[]字节数组的格式，才可以来进行跟broker的通信;
- 基于一个独立封装的组件来进行分区的选择和路由，可以用默认的，也可以用自定义的分区器，留下给用户自己扩展的空间;
- 对消息的大小，是否超出请求的最大大小，是否会填满 内存缓冲导致内存溢出，对一些核心的请求数据必然要进行严格的检查;
- 异步发送请求，通过先进入内存缓冲，同时设置一个callback回调函数的思路，在发送完成之后来回调你的函数通知你消息发送的结果，异步运行的后台线程配合起来使用，基于异步线程来发送消息;

本篇文章主要探索了Producer的发送数据的主流程，其中分段加锁，内存池设计，并发安全数据结构CopyOnWriteMap<>() 值得我们去思考和学习。下篇我们将讲解元数据的更新操作。




