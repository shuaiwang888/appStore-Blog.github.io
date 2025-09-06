### [参考资料：图解 Redis 数据结构](https://mp.weixin.qq.com/s/qptE172slg_6Tl1yuzdbfw)

## 一、Redis的两层数据结构简介

> **redis的性能高的原因之一是它每种数据结构都是经过专门设计的，并都有一种或多种数据结构来支持，依赖这些灵活的数据结构，来提升读取和写入的性能。** 如果要了解redis的数据结构，可以从两个不同的层面来讨论它：

### 1、第一个层面，是从使用者的角度，这一层面也是Redis暴露给外部的调用接口
- string
- list
- hash
- set
- sorted set
### 2、第二个层面，是从内部实现的角度，属于更底层的实现，比如：

- dict
- sds
- ziplist
- quicklist
- skiplist
- intset

![](https://img-blog.csdnimg.cn/ad0a040a93cf4435b055ca8fa663ab5a.png)

**本文的重点在于讨论第二个层面:**

1. Redis数据结构的内部实现
2. 这两个层面的数据结构之间的关系：Redis如何通过组合第二个层面的各种基础数据结构来实现第一个层面的更高层的数据结构

在讨论任何一个系统的内部实现的时候，我们都要先明确它的设计原则，这样我们才能更深刻地理解它为什么会进行如此设计的真正意图。在本文接下来的讨论中，我们主要关注以下几点：

1. **存储效率:** Redis是专用于存储数据的，它对计算机资源的主要消耗就在于内存，因此节省内存是它非常非常重要的一个方面。这意味着Redis一定是非常精细地考虑了压缩数据、减少内存碎片等问题。
2. **快速响应时间:** 与快速响应时间相对的，是高吞吐量。Redis是用于提供在线访问的，对于单个请求的响应时间要求很高，因此，快速响应时间是比高吞吐量更重要的目标。
3. **单线程:** Redis的性能瓶颈不在于CPU资源，而在于内存访问和网络IO。而采用单线程的设计带来的好处是，极大简化了数据结构和算法的实现。相反，Redis通过异步IO和pipelining等机制来实现高速的并发访问。显然，单线程的设计，对于单个请求的快速响应时间也提出了更高的要求。
## 二、redisObject：两层数据结构的桥梁

> redisObject数据结构详解：http://zhangtielei.com/posts/blog-redis-robj.html

### 1、什么是redisObject：

**从Redis的使用者的角度来看**，一个Redis节点包含多个database（非cluster模式下默认是16个，cluster模式下只能是1个），而一个database维护了从`key space`到`object space`的映射关系，这个映射关系的key是string类型，而value可以是多种数据类型，比如：string, list, hash、set、sorted set等。

**而从Redis内部实现的角度来看**，database内的这个映射关系是用一个dict来维护的。dict的key固定用一种数据结构来表达就够了，这就是动态字符串sds；而value则比较复杂，为了在同一个dict内能够存储不同类型的value，这就需要一个通用的数据结构，这个通用的数据结构就是robj，全名是`redisObject`。

**举个例子：**

- 如果value是list类型，那么它的内部存储结构是一个quicklist或者是一个ziplist
- 如果value是string类型，那么它的内部存储结构一般情况下是一个sds。但如果string类型的value的值是一个数字，那么Redis内部还会把它转成long型来存储，从而减小内存使用。
- 所以，一个robj既能表示一个sds，也能表示一个quicklist，甚至还能表示一个long型。

### 2、redis的数据结构定义：（基于Redis3.2版本）
```cpp
/* Object types */
#define OBJ_STRING 0
#define OBJ_LIST 1
#define OBJ_SET 2
#define OBJ_ZSET 3
#define OBJ_HASH 4
 
/* Objects encoding. Some kind of objects like Strings and Hashes can be
 * internally represented in multiple ways. The 'encoding' field of the object
 * is set to one of this fields for this object. */
#define OBJ_ENCODING_RAW 0     /* Raw representation */
#define OBJ_ENCODING_INT 1     /* Encoded as integer */
#define OBJ_ENCODING_HT 2      /* Encoded as hash table */
#define OBJ_ENCODING_ZIPMAP 3  /* Encoded as zipmap */
#define OBJ_ENCODING_LINKEDLIST 4 /* Encoded as regular linked list */
#define OBJ_ENCODING_ZIPLIST 5 /* Encoded as ziplist */
#define OBJ_ENCODING_INTSET 6  /* Encoded as intset */
#define OBJ_ENCODING_SKIPLIST 7  /* Encoded as skiplist */
#define OBJ_ENCODING_EMBSTR 8  /* Embedded sds string encoding */
#define OBJ_ENCODING_QUICKLIST 9 /* Encoded as linked list of ziplists */
 
#define LRU_BITS 24
typedef struct redisObject {
    unsigned type:4;
    unsigned encoding:4;
    unsigned lru:LRU_BITS; /* lru time (relative to server.lruclock) */
    int refcount;
    void *ptr;
} robj;
```

1. 一个robj包含如下5个字段：

- type: 对象的数据类型。占4个bit。可能的取值有5种：OBJ_STRING, OBJ_LIST, OBJ_SET, OBJ_ZSET, OBJ_HASH，分别对应Redis对外暴露的5种数据结构
- encoding: 对象的内部表示方式（也可以称为编码）。占4个bit。可能的取值有10种，即前面代码中的10个OBJ_ENCODING_XXX常量。
- lru: 做LRU替换算法用，占24个bit。这个不是我们这里讨论的重点，暂时忽略。
- refcount: 引用计数。它允许robj对象在某些情况下被共享。
- ptr: 数据指针。指向真正的数据。比如，一个代表string的robj，它的ptr可能指向一个sds结构；一个代表list的robj，它的ptr可能指向一个quicklist。

2. encoding字段的说明：

这里特别需要仔细察看的是encoding字段。对于同一个type，还可能对应不同的encoding，这说明同样的一个数据类型，可能存在不同的内部表示方式。而不同的内部表示，在内存占用和查找性能上会有所不同。

当type = OBJ_STRING的时候，表示这个robj存储的是一个string，这时encoding可以是下面3种中的一种：

- OBJ_ENCODING_RAW: string采用原生的表示方式，即用sds来表示。
- OBJ_ENCODING_INT: string采用数字的表示方式，实际上是一个long型。
- OBJ_ENCODING_EMBSTR: string采用一种特殊的嵌入式的sds来表示。

当type = OBJ_HASH的时候，表示这个robj存储的是一个hash，这时encoding可以是下面2种中的一种：

- OBJ_ENCODING_HT: hash采用一个dict来表示
- OBJ_ENCODING_ZIPLIST: hash采用一个ziplist来表示
3. 10种encoding的取值说明：

- OBJ_ENCODING_RAW: 最原生的表示方式。其实只有string类型才会用这个encoding值（表示成sds）。
- OBJ_ENCODING_INT: 表示成数字。实际用long表示。
- OBJ_ENCODING_HT: 表示成dict。
- OBJ_ENCODING_ZIPMAP: 是个旧的表示方式，已不再用。在小于Redis 2.6的版本中才有。
- OBJ_ENCODING_LINKEDLIST: 也是个旧的表示方式，已不再用。
- OBJ_ENCODING_ZIPLIST: 表示成ziplist。
- OBJ_ENCODING_INTSET: 表示成intset。用于set数据结构。
- OBJ_ENCODING_SKIPLIST: 表示成skiplist。用于sorted set数据结构。
- OBJ_ENCODING_EMBSTR: 表示成一种特殊的嵌入式的sds。
- OBJ_ENCODING_QUICKLIST: 表示成quicklist。用于list数据结构。


### 3、robj的作用：

- redisObject就是Redis对外暴露的第一层面的数据结构：string, list, hash, set, sorted set，而每一种数据结构的底层实现所对应的是哪些第二层面的数据结构（dict, sds, ziplist, quicklist, skiplist等），则通过不同的encoding来区分。可以说，robj是联结两个层面的数据结构的桥梁。
- 为多种数据类型提供一种统一的表示方式。
- 允许同一类型的数据采用不同的内部表示，从而在某些情况下尽量节省内存。
- 支持对象共享和引用计数。当对象被共享的时候，只占用一份内存拷贝，进一步节省内存。
## 三、第一层数据结构
### 1、String：

String是最简单的数据类型，一般用于复杂的计数功能的缓存：微博数，粉丝数等。

> 底层实现方式：动态字符串 sds 或者 long

**String的内部存储结构一般是sds（Simple Dynamic String），但是如果一个String类型的value的值是数字，那么Redis内部会把它转成long类型来存储，从而减少内存的使用。**

- 确切地说，String在Redis中是用一个robj来表示的。
- 用来表示String的robj可能编码成3种内部表示：OBJ_ENCODING_RAW，OBJ_ENCODING_EMBSTR，OBJ_ENCODING_INT。其中前两种编码使用的是sds来存储，最后一种OBJ_ENCODING_INT编码直接把string存成了long型。
- 在对string进行incr, decr等操作的时候，如果它内部是OBJ_ENCODING_INT编码，那么可以直接进行加减操作；如果它内部是OBJ_ENCODING_RAW或OBJ_ENCODING_EMBSTR编码，那么Redis会先试图把sds存储的字符串转成long型，如果能转成功，再进行加减操作。
- 对一个内部表示成long型的string执行append, setbit, getrange这些命令，针对的仍然是string的值（即十进制表示的字符串），而不是针对内部表示的long型进行操作。比如字符串”32”，如果按照字符数组来解释，它包含两个字符，它们的ASCII码分别是0x33和0x32。当我们执行命令setbit key 7 0的时候，相当于把字符0x33变成了0x32，这样字符串的值就变成了”22”。而如果将字符串”32”按照内部的64位long型来解释，那么它是0x0000000000000020，在这个基础上执行setbit位操作，结果就完全不对了。因此，在这些命令的实现中，会把long型先转成字符串再进行相应的操作。


### 2、Hash：

Hash适合用于存储对象，因为一个对象的各个属性，正好对应一个hash结构的各个field，可以方便地操作对象中的某个字段。

> 底层实现方式：压缩列表ziplist 或者 字典dict

当数据量较少的情况下，hash底层会使用压缩列表ziplist进行存储数据，也就是同时满足下面两个条件的时候：

- hash-max-ziplist-entries 512：当hash中的数据项（即filed-value对）的数目小于512时
- hash-max-ziplist-value 64：当hash中插入的任意一个value的长度小于64字节

当不能同时满足上面两个条件的时候，底层的ziplist就会转成dict，之所以这样设计，是因为当ziplist变得很大的时候，它有如下几个缺点：

- 每次插入或修改引发的realloc操作会有更大的概率造成内存拷贝，从而降低性能。
- 一旦发生内存拷贝，内存拷贝的成本也相应增加，因为要拷贝更大的一块数据。
- 当ziplist数据项过多的时候，在它上面查找指定的数据项就会性能变得很低，因为ziplist上的查找需要进行遍历。
- 总之，ziplist本来就设计为各个数据项挨在一起组成连续的内存空间，这种结构并不擅长做修改操作。一旦数据发生改动，就会引发内存realloc，可能导致内存拷贝。

### 3、List：

list 的实现为一个双向链表，经常被用作队列使用，支持在链表两端进行push和pop操作，时间复杂度为O(1)；同时也支持在链表中的任意位置的存取操作，但是需要对list进行遍历，时间复杂度为O(n)。

> list 的应用场景非常多，比如微博的关注列表，粉丝列表，消息列表等功能都可以用Redis的 list 结构来实现。可以利用lrange命令，做基于redis的分页功能。

1. Redis3.2之前的底层实现方式：压缩列表ziplist 或者 双向循环链表linkedlist

当list存储的数据量较少时，会使用ziplist存储数据，也就是同时满足下面两个条件：

- 列表中数据个数少于512个
- list中保存的每个元素的长度小于 64 字节

当不能同时满足上面两个条件的时候，list就通过双向循环链表linkedlist来实现了

2. Redis3.2及之后的底层实现方式：quicklist

quicklist是一个双向链表，而且是一个基于ziplist的双向链表，quicklist的每个节点都是一个ziplist，结合了双向链表和ziplist的优点

### 4、Set：

set是一个存放不重复值的无序集合，可做全局去重的功能，提供了判断某个元素是否在set集合内的功能，这个也是list所不能提供的。基于set可以实现交集、并集、差集的操作，计算共同喜好，全部的喜好，自己独有的喜好等功能。

> 底层实现方式：有序整数集合intset 或者 字典dict

当存储的数据同时满足下面这样两个条件的时候，Redis 就采用整数集合intset来实现set这种数据类型：

- 存储的数据都是整数
- 存储的数据元素个数小于512个


当不能同时满足这两个条件的时候，Redis 就使用dict来存储集合中的数据

### 5、Sorted Set：

Sorted set 相比 set 多了一个权重参数score，集合中的元素能够按score进行排列。可以做排行榜应用，取TOP N操作。另外，sorted set可以用来做延时任务。最后一个应用就是可以做范围查找。

> 底层实现方式：压缩列表ziplist 或者 zset

当 sorted set 的数据同时满足下面这两个条件的时候，就使用压缩列表ziplist实现sorted set

- 元素个数要小于 128 个，也就是ziplist数据项小于256个
- 集合中每个数据大小都小于 64 字节

当不能同时满足这两个条件的时候，Redis 就使用zset来实现sorted set，这个zset包含一个dict + 一个skiplist。dict用来查询数据到分数(score)的对应关系，而skiplist用来根据分数查询数据（可能是范围查找）。

## 四、第二层的数据结构
### 1、sds：
> sds数据结构详解：http://zhangtielei.com/posts/blog-redis-sds.html

1. 什么是sds：sds全称是Simple Dynamic String，具有如下显著的特点：

- 可动态扩展内存。sds表示的字符串其内容可以修改，也可以追加。

- 采用预分配冗余空间的方式来减少内存的频繁分配，从而优化字符串的增长操作

- 二进制安全（Binary Safe）。sds能存储任意二进制数据，而不仅仅是可打印字符。

- 与传统的C语言字符串类型兼容。

2. sds的数据结构：

sds是Binary Safe的，它可以存储任意二进制数据，不能像C语言字符串那样以字符’\0’来标识字符串的结束，因此它必然有个长度字段。但这个长度字段在哪里呢？实际上sds还包含一个header结构：
```cpp
struct __attribute__ ((__packed__)) sdshdr5 {
    unsigned char flags; /* 3 lsb of type, and 5 msb of string length */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len; /* used */
    uint8_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len; /* used */
    uint16_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr32 {
    uint32_t len; /* used */
    uint32_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; /* used */
    uint64_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
```

sds一共有5种类型的header。之所以有5种，是为了能让不同长度的字符串可以使用不同大小的header。这样，短字符串就能使用较小的header，从而节省内存。

所以，sds字符串的完整结构，由在内存地址上前后相邻的header和字符数组两部分组成：

- header：除了sdshdr5，一个header结构都包含3个字段：字符串的真正长度len、字符串的最大容量alloc和flags，flags总是占用一个字节。其中的最低3个bit用来表示header的类型。

- 字符数组：字符数组的长度等于最大容量+1，存放了真正有效的字符串数据。在真正的字符串数据之后，是空余未用的字节（一般以字节0填充），允许在不重新分配内存的前提下让字符串数据向后做有限的扩展。在真正的字符串数据之后，还有一个NULL结束符，即ASCII码为0的’\0’字符。这是为了和传统C字符串兼容。之所以字符数组的长度比最大容量多1个字节，就是为了在字符串长度达到最大容量时仍然有1个字节存放NULL结束符。

header的类型共有5种，在sds.h中有常量定义：
```cpp
#define SDS_TYPE_5  0
#define SDS_TYPE_8  1
#define SDS_TYPE_16 2
#define SDS_TYPE_32 3
#define SDS_TYPE_64 4
```

sds字符串的header，其实隐藏在真正的字符串数据的前面（低地址方向），这样的一个定义，有如下几个好处：

- header和数据相邻，从而不需要分成两块内存空间来单独分配，有利于减少内存碎片，提高存储效率
- 虽然header有多个类型，但sds可以用统一的char *来表达。且它与传统的C语言字符串保持类型兼容。如果一个sds里面存储的是可打印字符串，那么我们可以直接把它传给C函数，比如使用strcmp比较字符串大小，或者使用printf进行打印。

3. String robj的编码与解码过程：

- OBJ_STRING类型的字符串对象的编码和解码过程在Redis里非常重要，应用广泛。

- 当我们执行Redis的set命令的时候，Redis首先将接收到的value值（string类型）表示成一个type = OBJ_STRING并且encoding = OBJ_ENCODING_RAW的robj对象，然后在存入内部存储之前先执行一个编码过程，试图将它表示成另一种更节省内存的encoding方式。这一过程的核心代码，是object.c中的tryObjectEncoding函数。

- 当我们需要获取字符串的值，比如执行get命令的时候，我们需要执行与前面讲的编码过程相反的操作——解码。这一解码过程的核心代码，是object.c中的getDecodedObject函数。

- 对于编码的核心代码tryObjectEncoding函数和解码的核心代码getDecodedObject函数的详细说明，样请读者移步这篇文章：Redis内部数据结构详解(3)——robj - 铁蕾的个人博客

4. 为什么 Redis 的 String 底层数据结构使用 sds：

看到这里，我们就可以大概明白，为什么 String 底层数据结构要使用 sds：

- **性能高：** sds 数据结构主要由 len、alloc、buf[] 三个属性组成，其中 buf[] 为实际保存字符串的 char 类型数组；len 表示 buf[] 数组所保存的字符串的长度。由于使用 len 记录了保存的字符串长度，所以在获取字符串长度的时候，不需要从前往后遍历数组，直接获取 len 的值就可以了

- **内存预分配，优化字符串的增长操作：** 当需要修改数据时，首先会检查 sds 空间 len 是否满足，不满足则自动扩容空间，然后再执行修改。sds 在分配空间时，还会分配额外的未使用空间 free，下次再修改就先检查未使用空间 free 是否满足，如果满足则不用再扩展空间。通过空间预分配策略，有效较少在字符串连续增长情况下产生的内存重分配次数。

> 额外分配 free 空间的规则：
如果 sds 字符串修改后，len 值小于 1M，则额外分配未使用空间 free 的大小为 len 
如果 sds 字符串修改后，len 值大于等于 1M，则额外分配未使用空间 free 的大小为1M

- **惰性空间回收，优化字符串的缩短操作：** 当缩短 SDS 字符串后，并不会立即执行内存重分配来回收多余的空间，如果后续有增长操作，则可直接使用。

### 2、dict：
> dict数据结构详解：http://zhangtielei.com/posts/blog-redis-dict.html

1. dict是一个用于维护key-value映射关系的数据结构。Redis的一个database中所有key到value的映射，就是使用一个dict来维护的。不过，这只是它在Redis中的一个用途而已，它在Redis中被使用的地方还有很多。比如，Redis的hash结构，当它的field较多时，便会采用dict来存储。再比如，Redis配合使用dict和skiplist来共同维护一个sorted set。

2. dict本质上是为了解决算法中的查找问题，是一个基于哈希表的算法，在不要求数据有序存储，且能保持较低的哈希值冲突概率的前提下，查询的时间复杂度接近O(1)。它采用某个哈希函数并通过计算key从而找到在哈希表中的位置，采用拉链法解决冲突，并在装载因子（load factor）超过预定值时自动扩展内存，引发重哈希（rehashing），为了避免扩容时一次性对所有key进行重哈希，Redis采用了一种称为增量式重哈希（incremental rehashing）的方法，将重哈希的操作分散到对于dict的各个增删改查的操作中去。这种方法能做到每次只对一小部分key进行重哈希，而每次重哈希之间不影响dict的操作。dict之所以这样设计，是为了避免重哈希期间单个请求的响应时间剧烈增加，这与前面提到的“快速响应时间”的设计原则是相符的。

3. 当装载因子大于 1 的时候，Redis 会触发扩容，将散列表扩大为原来大小的 2 倍左右；当数据动态减少之后，为了节省内存，当装载因子小于 0.1 的时候，Redis 就会触发缩容，缩小为字典中数据个数的大约2 倍大小。

### 3、ziplist：
> ziplist数据结构详解：http://zhangtielei.com/posts/blog-redis-ziplist.html

1. ziplist是一个经过特殊编码的双向链表，可以用于存储字符串或整数，其中整数是按真正的二进制表示进行编码的，而不是编码成字符串序列。它能以O(1)的时间复杂度在表的两端提供push和pop操作。

2. 一个普通的双向链表，链表中每一项都占用独立的一块内存，并通过地址指针连接起来，但这种方式会带来大量的内存碎片，而且地址指针也会占用额外的内存。而ziplist使用了一整块连续的内存，将表中每一项存放在前后连续的地址空间内，类似于一个数组。另外，ziplist为了在细节上节省内存，对于值的存储采用了变长的编码方式，大概意思是说，对于大的整数，就多用一些字节来存储，而对于小的整数，就少用一些字节来存储。

3. 总的来说，ziplist使用一块连续的内存空间来存储数据，并采用可变长的编码方式，支持不同类型和大小的数据的存储，更加节省内存，而且数据存储在一片连续的内存空间，读取的效率也非常高。

### 4、quicklist：
> quicklist数据结构详解：http://zhangtielei.com/posts/blog-redis-quicklist.html

1. 什么是quicklist：

- quicklist是一个基于ziplist的双向链表，quicklist的每个节点都是一个ziplist，比如，一个包含3个节点的quicklist，如果每个节点的ziplist又包含4个数据项，那么对外表现上，这个list就总共包含12个数据项。quicklist的结构为什么这样设计呢？总结起来，大概又是一个空间和时间的折中：

- 双向链表便于在表的两端进行push和pop操作，但是它的内存开销比较大。首先，它在每个节点上除了要保存数据之外，还要额外保存两个指针；其次，双向链表的各个节点是单独的内存块，地址不连续，节点多了容易产生内存碎片。
ziplist由于是一整块连续内存，所以存储效率很高。但是，它不利于修改操作，每次数据变动都会引发一次内存的realloc。特别是当ziplist长度很长的时候，一次realloc可能会导致大批量的数据拷贝，进一步降低性能。
于是，结合了双向链表和ziplist的优点，quicklist就应运而生了。

2. quicklist中每个ziplist长度的配置：

不过，这也带来了一个新问题：到底一个quicklist节点包含多长的ziplist合适呢？比如，同样是存储12个数据项，既可以是一个quicklist包含3个节点，而每个节点的ziplist又包含4个数据项，也可以是一个quicklist包含6个节点，而每个节点的ziplist又包含2个数据项。

这又是一个需要找平衡点的难题。我们只从存储效率上分析一下：

- 每个quicklist节点上的ziplist越短，则内存碎片越多。内存碎片多了，有可能在内存中产生很多无法被利用的小碎片，从而降低存储效率。这种情况的极端是每个quicklist节点上的ziplist只包含一个数据项，这就蜕化成一个普通的双向链表了。
- 每个quicklist节点上的ziplist越长，则为ziplist分配大块连续内存空间的难度就越大。有可能出现内存里有很多小块的空闲空间（它们加起来很多），但却找不到一块足够大的空闲空间分配给ziplist的情况。这同样会降低存储效率。这种情况的极端是整个quicklist只有一个节点，所有的数据项都分配在这仅有的一个节点的ziplist里面。这其实蜕化成一个ziplist了


可见，一个quicklist节点上的ziplist要保持一个合理的长度。那到底多长合理呢？这可能取决于具体应用场景。实际上，Redis提供了一个配置参数list-max-ziplist-size，就是为了让使用者可以来根据自己的情况进行调整。
```
list-max-ziplist-size -2.   // 这个参数可以取正值，也可以取负值。
```

当取正值的时候，表示按照数据项个数来限定每个quicklist节点上的ziplist长度。比如，当这个参数配置成5的时候，表示每个quicklist节点的ziplist最多包含5个数据项。

当取负值的时候，表示按照占用字节数来限定每个quicklist节点上的ziplist长度。这时，它只能取-1到-5这五个值，每个值含义如下：

- -5: 每个quicklist节点上的ziplist大小不能超过64 Kb。（注：1kb => 1024 bytes）
- -4: 每个quicklist节点上的ziplist大小不能超过32 Kb。
- -3: 每个quicklist节点上的ziplist大小不能超过16 Kb。
- -2: 每个quicklist节点上的ziplist大小不能超过8 Kb。（-2是Redis给出的默认值）
- -1: 每个quicklist节点上的ziplist大小不能超过4 Kb。
### 5、intset：
> intset数据结构详解：http://zhangtielei.com/posts/blog-redis-intset.html

1. 什么是intset：


intset是一个由整数组成的有序集合，从而便于进行二分查找，用于快速地判断一个元素是否属于这个集合。它在内存分配上与ziplist有些类似，是连续的一整块内存空间，而且对于大整数和小整数（按绝对值）采取了不同的编码，尽量对内存的使用进行了优化。

2. intset的数据结构：
```cpp
typedef struct intset {
    uint32_t encoding;
    uint32_t length;
    int8_t contents[];
} intset;
 
#define INTSET_ENC_INT16 (sizeof(int16_t))
#define INTSET_ENC_INT32 (sizeof(int32_t))
#define INTSET_ENC_INT64 (sizeof(int64_t))
```
各个字段含义如下：

- encoding: 数据编码，表示intset中的每个数据元素用几个字节来存储。它有三种可能的取值：INTSET_ENC_INT16表示每个元素用2个字节存储，INTSET_ENC_INT32表示每个元素用4个字节存储，INTSET_ENC_INT64表示每个元素用8个字节存储。因此，- 
- intset中存储的整数最多只能占用64bit。
- length: 表示intset中的元素个数。encoding和length两个字段构成了intset的头部（header）。
- contents: 是一个柔性数组（flexible array member），表示intset的header后面紧跟着数据元素。这个数组的总长度（即总字节数）等于encoding * length。柔性数组在Redis的很多数据结构的定义中都出现过（例如sds, quicklist, skiplist），用于表达一个偏移量。contents需要单独为其分配空间，这部分内存不包含在intset结构当中。

其中需要注意的是，intset可能会随着数据的添加而改变它的数据编码：

- 最开始，新创建的intset使用占内存最小的INTSET_ENC_INT16（值为2）作为数据编码。
- 每添加一个新元素，则根据元素大小决定是否对数据编码进行升级。
3. intset与ziplist相比：

- ziplist可以存储任意二进制串，而intset只能存储整数。
- ziplist是无序的，而intset是从小到大有序的。因此，在ziplist上查找只能遍历，而在intset上可以进行二分查找，性能更高。
- ziplist可以对每个数据项进行不同的变长编码（每个数据项前面都有数据长度字段len），而intset只能整体使用一个统一的编码（encoding）。
### 6、skiplist：
> skiplist数据结构详解：http://zhangtielei.com/posts/blog-redis-skiplist.html

1. 什么是跳表：

跳表是一种可以进行二分查找的有序链表，采用空间换时间的设计思路，跳表在原有的有序链表上面增加了多级索引（例如每两个节点就提取一个节点到上一级），通过索引来实现快速查找。跳表是一种动态数据结构，支持快速的插入、删除、查找操作，时间复杂度都为O(logn)，空间复杂度为 O(n)。跳表非常灵活，可以通过改变索引构建策略，有效平衡执行效率和内存消耗。

- 跳表的删除操作：除了要删除原始链表中的节点，还要删除索引中的节点。

- 插入元素后，索引的动态更新：不停的往跳表里面插入数据，如果不更新索引，就有可能出现某两个索引节点之间的数据非常多的情况，甚至退化成单链表。针对这种情况，我们在添加元素的时候，通过一个随机函数，同时选择将这个数据插入到部分索引层。比如随机函数生成了值 K，那我们就将这个结点添加到第一级到第K级这K级的索引中

![](https://img-blog.csdnimg.cn/ad0a040a93cf4435b055ca8fa663ab5a.png)
