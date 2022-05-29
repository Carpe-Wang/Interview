# 1:String(最大存储512M)：
* 根据字符串的各式不同，可以分为三种：
  * String，如："Hello,World", "Carpe-Wang"
  * int 如：10
  * float如：92.5
* 常间命令：
  * set:添加或者修改已经存在的一个String类型的键值对
  * get:根据key来获取相关的value
  * mset:批量添加多个String类型的键值对
  * mget:根据多个key获取多棵String的value
  * incr:让整型的key自增1
  * incrby:让整型的key自增指定数量，如：incrby num 2，让num的值自增2
  * incrbyfloat:float类型自增指定数量
  * setnx:添加一个String类型的键值对，前提是这个key不存在，否则不执行
  * setex:添加String类型的键值对，并且指定有效期
* 如果用String类型存放Json对象的话，我们后期更改一个或者多个属性会很麻烦，只能全部重新更改。这个时候可以采用hash这种数据结构了.

# 2:hash
* 类似于hashmap
  * 我们一个属性可以设定一个key(注意这里的key为：redis中Value，我们存放的Hash的key)，然后我们可以更改一个或者多个从而不需要重新更改。
* 常间命令：
  * Hset key field value:添加或者修改Hash类型的key的field值
  * Hget key field:获取一个hash类型的key的field的值
  * Hmset:批量添加多个Hash类型的key的field
  * Hmget批量获得多个hash类型的key的field值
  * HgetAll:获取一个hash类型中的key中所有的field和value
  * Hkeys:获取一个hash类型的key中所有的field
  * Hvals:获取一个hash类型的key中的所有的value
  * HincrBy:让一个hash类型key字段值自增并指定步长
  * Hsetnx:添加一个hash类型的key的filed值，前提是这个field不存在，否则不执行
* 之前做一个blog系统的时候，我是用hash做点赞，转发，数量的记录。

# 
