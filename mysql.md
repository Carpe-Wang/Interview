# 1:三大范式：
* 第一范式（1NF）:列不可再分（比如，地址拆分为：省市区）
* 第二范式（2NF）属性完全依赖于主键
* 第三范式（3NF）属性不依赖于其它非主属性 属性直接依赖于主键，不存在递归关系。
# 2:事务
* 四大性质
	* A原子性
	* C一致性
	* I 隔离性
	* D持久性
`start transaction
commit（提交事务）和rollback（事务回滚）`
# 3:事务并发读产生的问题：
* 脏读：读到另一个事务的未提交更新数据，即读取到了脏数据。(A转账给B，但是A未提交事务，B查询成功，但是A回滚数据)。
* 不可重复读：对同一记录的两次读取不一致，因为另一个事务对该记录做了修改。（针对修改操作）。
* 幻读：对同一张表的两次查询不一致，因为另一事务插入了一条记录(是针对插入或删除操作)。
# 4:事务隔离级别：
* 读未提交(read-uncommitted)
>安全级别最低，会导致脏读，不可重复读，幻读等问题产生。
* 读已提交(read-committed)
>可以防止脏读，但是对于不可重复度，幻读无法处理
* 可重复读(repeatable-read)
>可以防止脏读，不可重复度，但是幻读无法处理
* 序列化(serializable)
>最高隔离级别不会产生问题

<font color="#dd0000">隔离级别的设定：set tx_isolation=‘隔离级别名称’;
注意！！！mysql的默认隔离级别为可重复度</font><br /> 
# 5:索引
>简单来说，一本1000页的书，使用目录可以快速找到希望找到的内容，这里的目录就是索引
* 索引建立规则：
	* 适合建立索引：
>在最频繁使用的、用以缩小查询范围的字段上建立索引；
在频繁使用的、需要排序的字段上建立索引。

* 不适合建立索引：
> 对于查询中很少设计的列或者重复值比较多的列，不建议建立索引；
对于一些特殊处理的数据类型，比如：text...！

`MySQL中常采用的索引结构为：B+和hash索引`
# 6:B+ 树：
* 基于 B 树和`叶子节点顺序访问指针(双向指针)进行实现`，它具有 B 树的平衡性，并且通过顺序访问指针来提高区间查询的性能。
* 和`红黑树`相比，`红黑树`的插入、删除操作均会破坏平衡树的平衡性，因此在插入删除操作之后，需要对树进行一个分裂、合并、旋转等操作来维护平衡性。
	* B+ 树一个节点可以存储`多个元素`，相对于红黑树的树高更低，磁盘 IO 次数更少。
	* 为了`减少磁盘 I/O 操作`，磁盘往往不是严格按需读取，而是每次都会预读。预读过程中，磁盘进行顺序读取，顺序读取不需要进行磁盘寻道。每次会读取页的整数倍。
操作系统一般将`内存和磁盘`分割成固定大小的块，每一块称为`一页`，`内存与磁盘`以页为单位交换数据。数据库系统将索引的一个节点的大小设置为页的大小，使得一次 I/O 就能完全载入一个节点。

* B + 树与 B 树的比较
	* `B+ 树的内部节点并没有指向关键字具体信息的指针`。因此其内部节点相对 B 树更小。如果把所有同一内部结点的关键字存放在同一盘块中，那么盘块所能容纳的关键字数量也越多。一次性读入内存中的需要查找的关键字也就越多。相对来说`IO读写`次数也就降低了。
	* B+ 树的`查询效率更加稳定`,由于`非叶子结点并不是最终指向文件内容的结点`，而只是`叶子结点中关键字的索引`。所以任何关键字的查找必须走一条从根结点到叶子结点的路。所有关键字查询的路径长度相同，导致每一个数据的查询效率相当。


# 7:hash索引：
`哈希索引能以 O(1) 时间进行查找`，但是失去了`有序性`。无法用于`排序与分组`、只支持精确查找，无法用于部分查找和范围查找。
# 8:索引分类：
* 数据结构角度出发：
	* 树索引
	* hash索引（innodb）：
		* innoDB存储引擎内部监控存储引擎热数据，然后内部创建一个hash索引，称之为自适应hash索引。控制参数：innodb_adaptive.
		* `哈希索引能以 O(1) 时间进行查找`，但是失去了`有序性`。无法用于`排序与分组`、只支持精确查找，无法用于部分查找和范围查找。
	* 从物理存储角度
		* 聚集索引：将表的主键，用来构造一个B+树，并且整张表的数据存放在该表的B+树的叶子节点中。没有主键，默认生成
		* 非聚集索引,如表中的关键非主键关键字。
	* 从逻辑角度
		* 普通索引
		* 唯一索引
		* 主键索引
		* 联合索引:将表上的多个列组合起来，进行索引
		* 全文索引：like
# 9:mysql架构：
Server层：包括连接器、查询缓存、分析器、优化器、执行器等，涵盖了 MySQL 的大多数核心服务功能，以及所有的内置函数（如：日期、时间、数学和加密函数等），所有跨存储引擎的功能都在这一层实现，比如：存储过程、触发器、视图等等。
存储引擎：据的存储和提取。其架构是插件式的，支持 InnoDB、MyISAM 等多个存储引擎。从 MySQL5.5.5 版本开始默认的是InnoDB，但是在建表时可以通过 engine = MyISAM 来指定存储引擎。不同存储引擎的表数据存取方式不同，支持的功能也不同。
# 10:MVCC：
MVCC多版本并发控制用来解决读写冲突的无锁并发控制的，为事务分配单向增长的时间戳，为每个修改保存一个版本，版本与事务时间戳关联，读操作只读该事务开始前的数据库的快照。

* 读-读：不需要并发控制，也不会存在任何问题
* 读-写：有线程安全问题，可能会造成事务隔离性问题，可能遇到脏读，幻读，不可重复读；
* 写-写：有线程安全问题，可能会存在更新丢失问题。
* 解决办法：
	* 在并发读写数据库时：可以做到在读操作时不用阻塞写操作，写操作也不用阻塞读操作，提高了数据库并发读写的性能；
	* 同时还可以解决脏读，幻读，不可重复读等事务隔离问题，但不能解决更新丢失问题。
* 原理：
	* 首先需要明白的是mysql是行式存储，`每一行有三个隐藏的列`。`DB_Row_ID（如果没有主键会使用这个作为主键）,DB_TRX_ID（自增，用于判断事务画的先后）,DB_ROLL_PTR（回滚指针，之前先一个事务版本）`这里需要注意的就是`DB_TRX_ID`对于那些已经提交的事务就可以删除掉所以需要明白的是如果事务编号为100的事务正在运行，事务为99的不能删除，因为执行失败需要回滚。
	* 	在`RR`隔离级别下，当一个事务->`begin`会生成一个当前id，比如是9，
		* 所以查不到比9大的事务对数据的更改
		* 如果当前id=9，肯定可以看到。
		* 事务id小于9的。
			* 如果提交的话，就可以看到。
			* 如果还未提交的话，就看不到。
			* 这也就引出了一个问题叫做Read View。 会有一个字端，保存活跃事务id的集合。 
# 10:行锁和表锁：
表级锁：开销小，加锁快，不会出现死锁。锁定粒度大，发生锁冲突的概率最高，并发量最低。
行级锁：开销大，加锁慢，会出现死锁。锁粒度小，发生锁冲突的概率小，并发度最高。
# 11:innodb和myisam：
区别：
事务：MyISAM不支持事务，InnoDB支持事务；
全文索引：MyISAM 支持全文索引，InnoDB 5.6 之前不支持全文索引；
关于count：MyISAM会直接存储总行数，InnoDB 则不会，需要按行扫描。意思就是对于 select count() from table; 如果数据量大，MyISAM 会瞬间返回，而 InnoDB 则会一行行扫描；
外键：myISAM不支持外键，Innodb支持外键。
隔离级别不同：Innodb是行锁，myISAM是表锁。
Innodb的存储引擎锁分类：
record lock：单个行记录上锁。
Gap lock：间隙锁锁定一个范围，不包括记录本身；
next-key lock：record+gap 锁定一个范围，包含记录本身。

# 12:查询性能优化方式：
只返回必要列，尽量不要使用select *
只返回必要的行：使用 LIMIT 语句来限制返回的数据。
使用缓存中的数据（使用缓存可以避免在数据库中进行查询）（redis）
使用索引
Union和UnionAll的区别
UNION 用于把来自多个 SELECT 语句的结果组合到一个结果集合中，MySQL 会把结果集中 重复的记录删掉，而使用 UNION ALL，MySQL 会把所有的记录返回，且效率高于 UNION 。

# 13:回表
在说回表之前，需要先弄清楚聚簇索引和非聚簇索引（物理存储方式）。
>`主键索引，其实就是聚簇索引（Clustered Index）`;主键索引之外，其他的都称之为非主键索引，`非主键索引也被称为二级索引（Secondary Index）`，或者叫作辅助索引，就比如对姓名这个列加索引。
* 如果我们直接`select * from user where id=100`需要搜索主键索引的 B+Tree 就可以找到数据。
* 通过非主键索引来查询数据，例如 `select * from user where username='java'`，那么此时需要先搜索 username 这一列索引的 B+Tree，搜索完成后得到主键的值，然后再去搜索主键索引的 B+Tree，就可以获取到一行完整的数据。
>对于第二种查询方式而言，一共搜索了两棵 B+Tree，第一次搜索 B+Tree 拿到主键值后再去搜索主键索引的 B+Tree，这个过程就是所谓的回表。


# 14:go链接Mysql进行简单操作
[详细参考](https://github.com/Carpe-Wang/Go_Mysql-Redis)
```go
package main

import (
	"database/sql"
	"fmt"
	"strings"
)

type User struct {
	Username string
	Password string
}

const (
	username = "root"
	password = "wkp159262"
	ip       = "127.0.0.1"
	port     = "3306"
	dbname   = "loginserver"
) //设置连接数据库的密码信息

var db *sql.DB //声明数据库连接池
func main() {
	connectMysql()
	//insertUser(User{
	//	Username: "CarpeWang",
	//	Password: "demo01",
	//})
}
func connectMysql() {
	path := strings.Join([]string{username, ":", password, "@tcp(", ip, ":", port, " )", dbname, "?charset=utf-8"}, "")
	db, _ := sql.Open("mysql", path) //driverName, dataSourceName string
	db.SetConnMaxLifetime(100)
	db.SetMaxIdleConns(10)
	//验证是否连接成功
	if err := db.Ping(); err != nil {
		fmt.Println("err:", err)
		return
	}
	fmt.Println("Connection successful")
}
func insertUser(user User) bool { //updata，delete，select操作都一直，区别为Prepare的区别
	tx, err := db.Begin() //开启事物
	if err != nil {
		fmt.Println("err:", err)
		return false
	}
	//准备sql语句
	statement, err1 := tx.Prepare("insert into loginserve (`name`,`password`) value (?,?)")
	if err1 != nil {
		fmt.Println(err1)
		return false
	}
	res, err2 := statement.Exec(user.Username, user.Password)
	if err2 != nil {
		fmt.Println("exec err")
		fmt.Println("err2:", err2)
		return false
	}
	tx.Commit() //提交事务
	fmt.Println(res.LastInsertId())
	return true
}
```
