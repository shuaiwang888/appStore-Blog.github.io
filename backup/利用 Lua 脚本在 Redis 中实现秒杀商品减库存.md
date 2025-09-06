
### 一、情景描述 

公司开展商品秒杀活动，共有 A、B、C、D 四个商品提供秒杀抢购，每件商品限售 5 件，售完即止。

其实这与我之前做的秒杀系统处理相似，只是这里使用的另一种解决方案：使用lua脚本；而我之前的逻辑是使用一个本地标识符，在redis库存减为0时将其标识为true，表示已经售完，这样变可以阻止商品数量外的无效请求进入redis代码逻辑，而若有人取消或超时未支付，便回补库存：1⃣️回补数据库库存；2⃣️回补redis库存；3⃣️将本地标识符设为false，表未售完！

### 二、实现方案

**1）使用 redis 保存四件商品的库存数量，如：**

```java
set goods_A 5
set goods_B 5
set goods_C 5
set goods_D 5
```

**2）利用 lua 脚本操作库存数量：**

```java-- 判断商品是否存在
local isExist = redis.call('exists', KEYS[1])                 -- 判断商品是否存在
    if isExist == 1 then
        local goodsNumber = redis.call('get', KEYS[1])        -- 获取商品的数量
        if goodsNumber > "0" then
            redis.call('decr', KEYS[1])                       -- 如果商品数量大于0，则库存减1
            return "success"
        else
            redis.call("del", KEYS[1])                        -- 商品数量为0则从秒杀活动删除该商品
            return "fail"
        end
    else return "notfound"
end
```

**3）支付成功则调用 lua 脚本：**
```java
if is_pay :
    redis-cli --eval reduceGood.lua goods_A            # 假设某用户支付A商品成功，则执行shell命令
```

**注：执行 lua 脚本解释：**

```java
# 对于只有5件的商品A，秒杀系统同时收到7个秒杀请求：
 
redis-cli --eval reduceGood.lua goods_A           # 返回success，库存剩余数量：4
 
redis-cli --eval reduceGood.lua goods_A           # 返回success，库存剩余数量：3
 
redis-cli --eval reduceGood.lua goods_A           # 返回success，库存剩余数量：2
 
redis-cli --eval reduceGood.lua goods_A           # 返回success，库存剩余数量：1
 
redis-cli --eval reduceGood.lua goods_A           # 返回success，库存剩余数量：0
 
redis-cli --eval reduceGood.lua goods_A           # 返回fail，该商品无库存
 
redis-cli --eval reduceGood.lua goods_A           # 返回notfound，该商品已下架
```

**商城实际处理部分代码：**
```java
            // 进行实际的sku 库存扣减
            String stockKey = String.format(SkuStorage.KEY, orderParam.getSkuId());
            if (!redisUtil.minusStock(stockKey, orderParam.getItemNum())) {
                log.error("库存扣减失败, {}", GsonUtil.toJson(orderParam));
                return Result.fail(ResultCode.STOCK_NOT_ENOUGH.getCode(), ResultCode.STOCK_NOT_ENOUGH.getMessage());
            }

            log.info("库存扣减成功：sku:{}, num:{}", orderParam.getSkuId(), orderParam.getItemNum());
```

### 三、总结 lua 脚本调用 Redis 的优势

- 多个命令合并到脚本统一处理，减少多个命令的网络开销

- 脚本能确保操作的原子性，不会受到其他客户端命令的影响

- 代码复用，redis将永久存放客户端发送的脚本


