# MySQL


## 基本的SQL语句，CRUD？设置索引？


- create
```sql
CREATE TABLE `media_birth_selected_0` (
	`id` bigint(20) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '自增id',
	`album_id` bigint(20) NOT NULL COMMENT '宝宝相册ID',
	`media_id` bigint(20) NOT NULL COMMENT '照片ID',
	`score` double NOT NULL COMMENT '照片美学评分',
	`special_day_type` int(8) NOT NULL COMMENT '添加的日期类型:枚举->数值',
	`date_time` int(11) NOT NULL COMMENT '照片拍摄时间',
	`status` tinyint(4) NOT NULL COMMENT '状态：当前使用/可用/已被更换/源照片被删除',
	`rank` int(8) NOT NULL COMMENT '该条记录在[纪念日|节日|其他]中的级别',

	PRIMARY KEY (`id`),
	UNIQUE KEY `uk_mbs_id` (`album_id`, `media_id`),
	KEY `idx_mbs_tag` (`album_id`, `tag`),
	KEY `idx_mbs_rank` (`album_id`, `rank`, `date_time`, `score`),
	KEY `idx_mbs_status` (`album_id`, `status`, `date_time`, `score`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

- insert
```sql
insert into t (col1, col2) values (1, 2);
```

- update
```sql
update t set col1 = col1 + 1 where primary_key_id = 1234;
```

- delete
```sql
delete from t where primary_key_id = 1234;
```


## 三范式？


- 一范式：列是原子不可分的
- 二范式：有主键，保证完全依赖。每列都和主键相关，而不能只和主键的一部分相关
- 三范式：无传递依赖。每个列都是和主键直接相关的，而非间接相关



## MyISAM和InnoDB特点、区别、使用场景？
| 功能 | MyISAM | InnoDB |
| --- | --- | --- |
| 事务 | 不支持 | 支持 |
| 锁机制 | 表级锁 | 行级锁、间隙锁 |
| 外键 | 不支持 | 支持 |
| 适用场景 | 读多写少，不需要**事务**支持 | 需要**事务**支持 |



## 创建索引的SQL语句？实现方法？


`create index on table(column);`


对某列建立有序的数据结构，提高查询的更新的效率，但是相应的会因为维护索引导致插入删除性能有所降低。


MySQL创建索引其实就是创建了一个有索引结构的新表，然后把旧表数据复制过去，效率比较低。表数据量大的时候谨慎使用这个方法。


InnoDB使用B+tree实现索引，其他存储引擎还有Hash方法实现的索引（内存引擎）、全文索引等。


## 索引类型

- 聚簇索引
- 唯一索引
- 普通索引 index
- 前缀索引
- 全文索引
- 哈希索引

不是具体的索引类型，但用于描述一种索引的应用场景：

- 覆盖索引
- 二级索引

## 什么是聚簇索引？


索引键值的逻辑顺序与索引所服务的表中相应行的物理顺序相同的索引，被该索引称为聚集索引，反之为非聚集索引。InnoDB的主键是聚簇索引的。


## 什么是覆盖索引？

当索引中的内容覆盖了一个查询要查找的所有列，称这个查询使用了覆盖索引。这种情况下，数据库不需要通过查找到的主键值进行二次查找，而是可以直接从索引返回要查询的字段。

例如一张用户表`user`包含`name`, `age`, `icon`字段和`index(age, name)`索引，此时如果执行如下查询便可直接返回数据，不需要从次级索引查出主键后再回到聚簇索引上查询数据，但如果待查询字段加上`icon`就不可以了。

`select name, age from user where age = 22;`;


## 主键和唯一索引的区别？


主键是一个记录的唯一标识。一张表只能有一个主键。


唯一索引指的是其限定的列的取值不能出现冲突，但允许多个NULL。


主键和唯一索引区别主要有如下几点：


- 主键不允许NULL；唯一索引允许NULL，而且可以有多个NULL（在数据库中NULL和NULL不等，所以这不算破坏了唯一性）
- 一个表只能有一个主键而可以有多个唯一索引
- 从设计角度讲，主键用于唯一标志行，而唯一索引只是一种限定条件

## 哪些键适合建索引？

- 用于**查找**的建
- 用于**连接**的键
- 用于**排序**、**分组**的键

## 什么时候索引会失效？


- 以`%`开头的模糊匹配
- `OR`涉及到的列没有各自的索引
- 匹配的并非索引列本身，而是使用表达式、函数一类的计算值，例如`where col*2 <= 100`
- 要查询的数据量占据了全表的很大部分（20%以上）时，使用索引会带来大量的随机IO，此时MySQL会选择全表扫描的顺序IO
- 多个查询条件使用列未满足最左前缀索引的要求
- 范围匹配后的索引全部失效



## B树是什么？


B树是一棵多路平衡查找树，如果每个节点可以有最多m个子树，则其每个节点可以存储关键码数量的范围在`[m/2, m]`区间内，少于或超出会进行节点重组和拆分。这种设计保证了B树是平衡、数据有序的，并且树的高度相比二叉平衡树可以减少很多。


## B+树改进？这个索引结构为什么有效？

B+树其改进主要是如下几点：

- 所有的数据只存在于叶节点，非叶节点只存储关键码用于索引
- 叶节点是前后相连的，对应到全数据扫描的操作只需要遍历所有叶节点即可，而不用像B树一样深搜整棵树

B+树的一个叶节点一般被实现为一个页，页是在磁盘上一块连续的存储空间，这充分利用了磁盘I/O的顺序读取快而随机读取慢的特点。


## 什么是Hash索引？为什么InnoDB没有显式的Hash索引？


Hash索引是一个适用于随机访问的索引，不适合磁盘这类随机访问很慢但顺序访问较快的存储器，而是适合内存、固态硬盘这类存储器。


## 事务的四个基本特性？ ACID


- 原子性（Atomicity）：事务要么全部执行，要么全部不执行
- 一致性（Consistency）：事务执行前后的数据库完整性不会被破坏，例如银行存款总数不会改变
- 隔离性（Isolation）：多个事务之间是隔离的。一个事务不会看到另一个事务执行时的中间状态的数据。
- 持久性（Durability）：在事务完成后，事务对数据库所做的变更会永久写入数据库中，即使发生致命错误也不会丢失，且无法回滚



## 事务的隔离级别？

- Read Uncommitted：读未提交，可以读到其他事物的中间状态数据。产生“脏读”
- Read Committed：读提交，可在一个事务内多次读读到不同结果。避免“脏读”，产生“不可重复读”
- Repeatable read：可重复读。避免“脏读”、“不可重复读”，产生“幻读”
- Serializable：串行化，避免“脏读”、“不可重复读”、“幻读”

对几种读问题的解释：
- 脏读：会在一个事务内读到另外一个事务回滚之前修改的数据。如果出现脏读，那事务的隔离性已经没有得到保证
- 不可重复读：会在一个事务内的多次读取同一数据时读出不同的结果
- 幻读：事务A读取一些行时，事务B在该范围内插入了新行，事务A再次读取，结果与第一次读取不同，好像是幻觉一样，谓之“幻读”。

注意区分幻读与不可重复读，不可重复读对应**非结构性修改**，例如更新（update）；幻读对应**结构性修改**，例如插入（insert）与删除（delete）

MVCC解决了读场景下的幻读问题，不能解决写场景下的幻读问题，例如更新范围内所有的记录。写场景下解决幻读依赖于间隙锁。

## MVCC是什么？解决了什么问题？


## InnoDB插入数据时发生了什么？


## InnoDB如何保证事务持久性的？

// todo: redo log

![](.数据库%20FAQ.assets/2022-09-14-17-35-05.png)

1. 数据按页加载到内存
2. 引擎修改页上的数据
3. 定时刷新脏页，写回到硬盘

redo log存在一个两阶段提交的机制以保证binlog和redo log的同步。
redo log事务提交分为prepare和commit阶段，其顺序如下：

1. redo log prepare
2. binlog 写入
3. redo log commit

可以参看下图：

![](.数据库%20FAQ.assets/2022-09-14-17-38-22.png)

ref: [MySQL三大日志(binlog、redo log和undo log)详解 | JavaGuide](https://javaguide.cn/database/mysql/mysql-logs.html#%E5%89%8D%E8%A8%80)

## 如何处理事务回滚？ undo-log

// todo: undo log

每一行都有三个隐藏字段：

- DB_TRX_ID（6字节）：表示最后一次插入或更新该行的事务 id。此外，delete 操作在内部被视为更新，只不过会在记录头 Record header 中的 deleted_flag 字段将其标记为已删除
- DB_ROLL_PTR（7字节） 回滚指针，指向该行的 undo log 。如果该行未被更新，则为空
- DB_ROW_ID（6字节）：如果没有设置主键且该表没有唯一非空索引时，InnoDB 会使用该 id 来生成聚簇索引

undo log 主要有两个作用：

当事务回滚时用于将数据恢复到修改前的样子
另一个作用是 MVCC ，当读取记录时，若该记录被其他事务占用或当前版本对该事务不可见，则可以通过 undo log 读取之前的版本数据，以此实现非锁定读

在 InnoDB 存储引擎中 undo log 分为两种： insert undo log 和 update undo log。

不同事务或者相同事务的对同一记录行的修改，会使该记录行的 undo log 成为一条链表，链首就是最新的记录，链尾就是最早的旧记录。

![](.数据库%20FAQ.assets/2022-09-14-17-19-36.png)


## MySQL复制过程？


## drop、delete与truncate的区别


- drop会移除整张表，包括索引和权限。不能回滚，且不会触发任何DML触发器
- delete可以删除表中的任意多数据
   - 用户需要执行commite或rollback
   - delete会触发表上的触发器
- truncate清除表中所有数据但不移除表
truncate要比同样目的的delete快一些，但**不能回滚**，不会触发表上触发器。



## 什么是视图？


视图是虚拟的表，具有物理表的功能，可以插入、删除、修改等，但不能设置索引。


## 什么是游标？


查询返回行集，通过改变游标的位置查看行集中的具体行信息。


## 什么是触发器？


在满足某种条件时执行特定的语句序列，多用于保持数据库的完整性。

## `UNION ALL` / `UNION` 区别？

在明显不会有重复值时使用 UNION ALL 而不是 UNION

- UNION 会把两个结果集的所有数据放到临时表中后再进行去重操作
- UNION ALL 不会再对结果集进行去重操作

## 如何修改大表的表结构？

考虑使用 `pt-online-schema-change`，或类似思路

核心在于：

- 避免大表修改产生的主从延迟
- 避免在对表字段进行修改时进行锁表

对大表数据结构的修改一定要谨慎，会造成严重的锁表操作，尤其是生产环境，是不能容忍的。

`pt-online-schema-change` 会首先建立一个与原表结构相同的新表，并且在新表上进行表结构的修改，然后再把原表中的数据复制到新表中，并在原表中增加一些触发器。把原表中新增的数据也复制到新表中，在行所有数据复制完成之后，把新表命名成原表，并把原来的表删除掉。把原来一个 DDL 操作，分解成多个小的批次进行。

## 乐观锁、悲观锁？

// todo

# Redis

## 为什么Redis快？

- 内存型
- 单线程无锁 + IO多路复用的Reactor模式
- 高效的数据结构

![](.数据库%20FAQ.assets/2022-09-14-17-44-22.png)

## 常用数据结构，及其使用场景？

8 种：

- String
- Hash
- List
- Set
- ZSet
- Bitmap
- Geo
- HyperLogLog

## 实现层面的数据结构

- 简单动态字符串 SDS
- 链表
- 哈希表
- 跳表
- 整数集合
- 压缩列表：连续内存存储的entry，用于小型列表和哈希表的存储

## Redis删除过期key机制？

如果假设你设置了一批 key 只能存活 1 分钟，那么 1 分钟后，Redis 是怎么对这批 key 进行删除的呢？

常用的过期数据的删除策略就两个（重要！自己造缓存轮子的时候需要格外考虑的东西）：

惰性删除 ：只会在取出 key 的时候才对数据进行过期检查。这样对 CPU 最友好，但是可能会造成太多过期 key 没有被删除。
定期删除 ： 每隔一段时间抽取一批 key 执行删除过期 key 操作。并且，Redis 底层会通过限制删除操作执行的时长和频率来减少删除操作对 CPU 时间的影响。

Redis采取上述两种方案组合的方法来实现最终的过期key清除任务

## Redis数据淘汰策略

## Redis持久化方法？

- RDB: 保存一份当前kv的数据快照，导出过程会导致redis无响应
- AOF: 保存历史执行的指令，而且可以每秒执行，不影响redis性能
- 混合开启 RDB+AOF: 在某个时间点之前，使用RDB记录键值，之后使用AOF，思想和key compact类似，可以压缩持久化文件的大小

## Redis事务？

1. Redis事务

默认的Redis事务只是一种打包运行多条命令的机制，节省网络开销，并且中途不会插入其他命令，但不满足原子性和持久性。
Redis 事务在运行错误的情况下，除了执行过程中出现错误的命令外，其他命令都能正常执行。
并且，Redis 是不支持回滚（roll back）操作的。因此，Redis 事务其实是不满足原子性的（而且不满足持久性）。

一个Redis事务执行范例如下，由`MULTI`开始，`EXEC`执行或`DISCARD`放弃。
调用`EXEC`前如果发生指令错误（例如命令参数错误）则会直接报错，全部命令都不会执行。
调用`EXEC`后，执行过程中发生错误（例如`incr`了一个字符串），则当前指令失败，前序和后续指令正常执行。

```r
> MULTI
OK
> SET USER "user"
QUEUED
> GET USER
QUEUED
> EXEC # 或者调用DISCARD放弃执行该事务
1) OK
2) "user"
```

但是从Redis自己的角度看来，一条命令执行后抛出错误，仍然属于「执行了这条命令」，所以整体看来，
所有的命令仍然满足「要么全执行，要么全不执行」的原子性语义，但个人认为这个说法很牵强。

2. lua脚本

一种更好的方法是将操作封装为lua脚本，可以保证这中间不会插入其他命令，但是仍然不满足原子性和持久性。
该方法与Redis事务区别在于如果中间命令执行失败，则不会继续执行后续命令，但已执行的命令无法撤回。

3. Functions

Redis 7.0发布增加了Functions特性，可以理解为优化了的lua脚本实现。

// todo

## Redis集群？

三种模式：
1. 主备，需要人工切换
2. sentinel：更自动化的主备方案，备库会检测主库状态，并且在异常时通过订阅发布的方式通知客户端切换
3. 分片：通过哈希槽的方式将数据分配到不同的redis实例上从而达成更高的集群性能

ref: [[Redis] 你了解 Redis 的三种集群模式吗？ - SegmentFault 思否](https://segmentfault.com/a/1190000022808576)

## Redis缓存一致性

Cache Through Pattern:

读写都直接在缓存上进行，当写请求到来时由缓存更新DB（同步操作）

![](.数据库%20FAQ.assets/2022-09-14-23-46-21.png)

Cache Aside Pattern:

标准方案：更新DB -> 删除缓存，简单，但中间间隔期间可能会有脏读和缓存击穿的的问题。
如果更新DB成功但删除缓存失败，需要进行重试处理。这个重试可以使业务代码重试，也可以是发送到MQ重试，取决于业务重要程度。

比较极端的方案，适用于极端的一致性要求的场景。
业务更新DB，并且有独立的同步服务（例如canal）读DB的binlog并更新至缓存中。

![](.数据库%20FAQ.assets/2022-09-14-23-44-21.png)

不存在更完美的同步、高性能、强一致性的解决方案，这本质上是一种DB和缓存系统的分布式事务问题，想做到强一致性，必然会牺牲性能。

ref: 
- [缓存和数据库一致性问题，看这篇就够了](https://mp.weixin.qq.com/s?__biz=MzIyOTYxNDI5OA==&mid=2247487312&idx=1&sn=fa19566f5729d6598155b5c676eee62d&chksm=e8beb8e5dfc931f3e35655da9da0b61c79f2843101c130cf38996446975014f958a6481aacf1&scene=178&cur_album_id=1699766580538032128#rd)
- [缓存更新的套路 | 酷 壳 - CoolShell](https://coolshell.cn/articles/17416.html)

## 缓存雪崩是什么？怎么解决

定义：在一个时间点有大量缓存失效导致流量打到了DB上。

解决方案：

- 不同的缓存设置随机过期时间
- 热点key设置为永不过期，依赖于主动更新

## Redis查找一个key的过程，以get为例

1.`lookupKeyReadOrReply()`，查找不到就回复reply给客户端。
	1. 判断key过期与否 & 删除过期key
	2. 统计访问次数
2. 查找 `dictEntry *dictFind(dict *d, const void *key)`
	1. 如果在渐进rehash，执行一个key的rehash
	2. 计算hash
	3. table中找出对应entry，如果存在key冲突，需要遍历对应的value链表找到指定key

ref: [Redis源代码阅读：key的查找过程 | 毛英东的个人博客](https://www.maoyingdong.com/redis/redis_source_code_get_string_type_key/)

## Redis分布式锁

- `setnx`
- `redlock`

各自的问题？