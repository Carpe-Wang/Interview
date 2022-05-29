# 1:String(最大存储512M)：
* 根据字符串的各式不同，可以分为三种：
  * String，如："Hello,World", "Carpe-Wang"
  * int 如：10
  * float如：92.5
```
   set:添加或者修改已经存在的一个String类型的键值对
   get:根据key来获取相关的value
   mset:批量添加多个String类型的键值对
   mget:根据多个key获取多棵String的value
   incr:让整型的key自增1
   incrby:让整型的key自增指定数量，如：incrby num 2，让num的值自增2
   incrbyfloat:float类型自增指定数量
   setnx:添加一个String类型的键值对，前提是这个key不存在，否则不执行
   setex:添加String类型的键值对，并且指定有效期
  ```
* 如果用String类型存放Json对象的话，我们后期更改一个或者多个属性会很麻烦，只能全部重新更改。这个时候可以采用hash这种数据结构了.

# 2:hash
* 类似于hashmap
  * 我们一个属性可以设定一个key(注意这里的key为：redis中Value，我们存放的Hash的key)，然后我们可以更改一个或者多个从而不需要重新更改。
```
   Hset key field value:添加或者修改Hash类型的key的field值
   Hget key field:获取一个hash类型的key的field的值
   Hmset:批量添加多个Hash类型的key的field
   Hmget批量获得多个hash类型的key的field值
   HgetAll:获取一个hash类型中的key中所有的field和value
   Hkeys:获取一个hash类型的key中所有的field
   Hvals:获取一个hash类型的key中的所有的value
   HincrBy:让一个hash类型key字段值自增并指定步长
   Hsetnx:添加一个hash类型的key的filed值，前提是这个field不存在，否则不执行
  ```
* 之前做一个blog系统的时候，我是用hash做点赞，转发，数量的记录。


# 3:List
 * redis中list和java中的一个linkedList相似，双向链表。原理决定了适用场景：有序，元素可重复，快速插入，删除，但是查询效率一般。(朋友圈点赞，谁先点赞，谁后点赞)
```
   Lpush key element:像列表左插入一个或者多个元素
   Lpop key:移除并返回列表左侧的第一个元素
   Rpush key element: 像列表的右侧插入一个或者多个元素
   Rpop key:移除并返回列表中右侧第一个元素
   Lrange key star end:返回一段角标范围内的所有元素
   Blpop和Brpop:和Lpop和Rpop相似，只不过就是没有元素的时候等待指定时间，而不是直接返回null
```
# 4:Set
 * 和java中的hashset相似，可以看作一个value为null的hashmap，主要特点是：无序，元素不可重复，查找快，支持交集，并集，差集。
``` * Sadd key member：向set中添加一个或者多个元素
   Srem key member：移除set中的指定元素
   Scard key：返回set中元素个数
   Sismember key member：判断一个元素是否存在与set中
   Smembers：获取set中的所有元素
  ```

# 5:SortedSet
 * 简单来说就是java中的TreeSet，但是底层相差很大，SortedSet中，每个元素都带有一个score属性，可以基于score属性对元素排序，底层实现的是一个跳表+hash表。特点就是：可排序，元素不重复，查询速度快
```
   Zadd key score member:添加一个或者多个元素到SortedSet
   Zrem key member:删除SortedSet指定元素
   Zscore key member:获取SortedSet中的指定元素的score
   Zrank key member:获取SortedSet中指定元素的排名
   Zcard key:获取SortedSet元素个数
   Zcount key min max:统计Score值在给定范围内的所有元素的个数
   ZincrBy key increment member:让SortedSet中指定元素自增，步长为指定的increment值
   Zrange key min max:按照Score排序后，获取指定排名范围内的元素
   Zrangebyscore key min max:按照score排序后，获取指定score范围内的元素
```
# 6:Redis持久化：
* RDB：持久化即通过创建快照（压缩的二进制文件）的方式进行持久化，保存某个时间点的全量数据（内存）。RDB持久化是Redis默认的持久化方式。RDB持久化的触发包括手动触发与自动触发两种方式。
  * 优点：RDB文件紧凑，体积小，网络传输快，适合全量复制；恢复速度比AOF快很多，与AOF相比，RDB最重要的优点之一是对性能的影响相对较小。
  * RDB文件的致命缺点在于其数据快照的持久化方式决定了必然做不到实时持久化，而在数据越来越重要的今天，数据的大量丢失很多时候是无法接受的，因此AOF持久化成为主流。此外，RDB文件需要满足特定格式，兼容性差（如老版本的Redis不兼容新版本的RDB文件）。
* RDB持久化过程：
  * 在持久化的适合会fork一个子进程，来进行持久化。
  * 会先将数据写入到一个临时文件中，等到持久化过程都结束了。
  * 在用这个临时文件替换上次持久化产生的临时文件。
* AOF（Append-Only-File）持久化即记录所有变更数据库状态的指令，以append的形式追加保存到AOF文件中。在服务器下次启动时，就可以通过载入和执行AOF文件中保存的命令，来还原服务器关闭前的数据库状态。
 * 优点：支持秒级持久化、兼容性好。
 * 缺点是文件大、恢复速度慢、对性能影响大。
# 7:缓存穿透，缓存击穿，缓存雪崩
* 缓存穿透：指查询一个一定不存在的数据，如果从存储层查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到 DB 去查询，可能导致 DB 挂掉。
  * 解决办法：
    * 查询返回的数据为空，仍把这个空结果进行缓存，但过期时间会比较短。
    * 布隆过滤器：将所有可能存在的数据哈希到一个足够大的 bitmap 中，一个一定不存在的数据会被这个 bitmap 拦截掉，从而避免了对 DB 的查询。
* 缓存击穿：对于设置了过期时间的 key，缓存在某个时间点过期的时候，恰好这时间点对这个 Key 有大量的并发请求过来，这些请求发现缓存过期一般都会从后端 DB 加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把 DB 压垮。
  * 解决办法：
    * 使用互斥锁：当缓存失效时，不立即去 load db，先使用如 Redis 的 setnx 去设置一个互斥锁，当操作成功返回时再进行 load db 的操作并回设缓存，否则重试 get 缓存的方法。
    * 永远不过期：物理不过期，但逻辑过期（后台异步线程去刷新）。
* 缓存雪崩：设置缓存时采用了相同的过期时间，导致缓存在某一时刻同时失效，请求全部转发到 DB， DB 瞬时压力过重雪崩。与缓存击穿的区别：雪崩是很多 key，击穿是某一个key 缓存。
   * 解决办法：
      * 将缓存失效时间分散开，比如可以在原有的失效时间基础上增加一个随机值，比如 1-5 分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。

# 8:集群模式
* 主从复制
  * 从数据库启动时，就会同步向主数据库发送sync命令，主数据库收到sync命令会开始在后台保存快照RDB
* 哨兵模式
  * 作用是监控redis主，从数据库是否正常运行。
  * 主出现故障自动将从数据库转换为主数据库。
  * 哨兵的使用重点：
     * 哨兵至少需要三个实例。保障健壮性。
     * 哨兵 + redis 主从的部署架构，是不保证数据零丢失的，只能保证 redis 集群的高可用性。
     * 使用哨兵和redis最好在测试环境做好一切压测。
     * 配置哨兵监控一个系统时，只需要配置其监控主数据库即可，哨兵会自动发现所有复制该主数据库的从数据库。

# 9:Redis数据和Mysql数据库中不一致怎么办
* 采用延迟双删策略
    * 第二个删除redis，如果删除了缓存Redis，还没有来得及写库MySQL，另一个线程就来读取，发现缓存为空，则去数据库中读取数据写入缓存，此时缓存中为脏数据
    * 第一个删redis。如果先写了库，在删除缓存前，写库的线程宕机了，没有删除掉缓存，则也会出现数据不一致情况。
    * 先淘汰缓存。
    * 在写入数据库。
    * sleep800ms，在进行淘汰。
    * 这样做的好处就是我们可以在800ms内所造成的缓存脏数据，再次删除。
 ```
public void use(String key, Object data){
    redis.delKey(key);
    db.updateData(data);
    Thread.sleep(800);
    redis.delKey(key);
}
```

# 10:java整合Redis
[java整合redis](https://github.com/Carpe-Wang/Springboot-Redis) 
