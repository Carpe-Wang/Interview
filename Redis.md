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
