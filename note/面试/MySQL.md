

**LIKE和索引**

![1559210919331](D:\笔记\img\1559210919331.png)



**事务**

原子性，一致性，隔离性，持久性



**隔离级别**



**脏读**

读到另一个事务的未提交更新数据读到另一个事务的未提交更新数据

**不可重复读**

对同一记录的两次读取不一致，读取到了另一事务的更新

**幻读**

幻读是读取到了另一事务的插入，对同一表的两次查询不一致



A　SERIALIZABLE（串行化）（三种问题都能避免）

l  不会出现任何并发问题，因为它是对同一数据的访问是串行的，非并发访问的；

l  性能最差；

 

B　REPEATABLE READ（可重复读）（MySQL）

l  防止脏读和不可重复读（能处理幻读）

l  性能比SERIALIZABLE好

 

C　READ COMMITTED（读已提交数据）（Oracle）

l  防止脏读，没有处理不可重复读，也没有处理幻读；

l  性能比REPEATABLE READ好

 

D　READ UNCOMMITTED（读未提交数据）啥也不处理

l  可能出现任何事务并发问题

l  性能最好



**事务传播属性**

Propagation.***REQUIRED***

如果当前运行的方法，已经存在事务，不会运行新的事务，就会加入当前的事务

Propagation.***REQUIRED_NEW***

如果当前运行的方法，已经存在事务，事务会挂起，会始终开启一个新的事务，执行完后，刚才挂起的事务才继续运行。



**主从复制**

从机 根据主机 的日志自己操作一遍



**索引**

**B+Tree原理**

<https://blog.csdn.net/weixin_30531261/article/details/79312676>	

先加载顶节点，然而二分查找法查询到一个更小的区间的指针，加载指针指向的节点，然后继续缩小范围。一般三层树就能表示几百万的数据

**BTree 和 B+Tree**

后者是前者的优化，后者的数据只放在叶子节点上，非叶子节点上放键值和指针

B+树有一个最大的好处，方便扫库，B树必须用中序遍历的方法按序扫库，而B+树直接从叶子结点挨个扫一遍就完了。 

**什么情况下建立索引**

频繁查询字段，外键字段（如果不能创建索引，可加大**joinbuffer**参数），排序字段，统计字段，分组字段

数据少，经常增删改，性别列只有两个值，不适合创建索引



**Innodb 和 myisam**

|          | MyISAM                                                   | InnoDB                                                       |
| -------- | -------------------------------------------------------- | ------------------------------------------------------------ |
| 主外键   | 不支持                                                   | 支持                                                         |
| 事务     | 不支持                                                   | 支持                                                         |
| 行表锁   | 表锁，即使操作一条记录也会锁住整个表，不适合高并发的操作 | 行锁, 操作时只锁某一行，不对其它行有影响，适合高并发的操作   |
| 缓存     | 只缓存索引，不缓存真实数据                               | 不仅缓存索引还要缓存真实数据，对内存要求较高，而且内存大小对性能有决定性的影响 |
| 表空间   | 小                                                       | 大                                                           |
| 关注点   | 性能                                                     | 事务                                                         |
| 默认安装 | Y                                                        | Y                                                            |



**mysql的锁**

<https://www.toutiao.com/i6669505644367708680/?timestamp=1553409423&app=news_article&group_id=6669505644367708680>

- **表级锁**：开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。

加读锁，lock table t read；加写锁：lock table t write;

读锁会阻塞写，但是不会堵塞读。而写锁则会把读和写都堵塞

*表锁分析***：**show status like 'table%'

Table_locks_immediate：产生表锁定的次数

Table_locks_waited：因为表锁而等待的次数，大表示锁争用激烈

- **行级锁**：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。

修改某行数据时，会锁住其他会话的写操作

无索引，行锁会升级为表锁

*行锁分析*: show status like 'innodb_row_lock%'

Innodb_row_lock_current_waits：当前正在等待锁定的数量；

Innodb_row_lock_time：从系统启动到现在锁定总时间长度；

Innodb_row_lock_time_avg：每次等待所花平均时间；

Innodb_row_lock_time_max：从系统启动到现在等待最常的一次所花的时间；

Innodb_row_lock_waits：系统启动后到现在总共等待的次数；

- **页面锁**：开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般



**innodb中的锁**

**S 锁**：共享锁，持有S 锁可以读，别的事务也能获取 S 锁，但是不能获取X 锁

**X 锁**：排他锁，持有X 锁可以删改，别的事务不能获取S 锁和X 锁

**IS锁**：意向共享锁，表级，当前事务试图在当前行上设置共享锁，select ... lock in share mode

**IX锁**：意向排他锁，表级，当前事务试图在当前行上设置排他锁，select ... for update

![img](http://note.youdao.com/yws/res/1392/1AA969CB75EB4833AB80B5A4D25CB4EB)



- **间隙锁**，使用范围条件时，范围内的数据都会被加锁，即使键不存在也会锁住

如果索引不是唯一索引，where t.code = 10，也会有间隙锁，会锁住，上一个code到10之间的数据

- **Record锁**是对索引记录的锁定,分S模式和X模式
- **next-key 锁，**将下一个索引本身和它之前的间隙，加上x锁或者s锁，解决了幻读问题

例子：code列上创建了非唯一索引，code有，1,5,10,三个数，

如果条件为code = 5，那么5这行会加record锁，1 - 5会有间隙锁，5 - 10 会有 next-key锁，所以如果是 IX 锁，1 -10都会被锁住了

如果条件为code > 8，那么5 - 10之间有间隙锁，10以上有next-key锁，所以如果是 IX 锁，5以后都会被锁住了

- **自增锁**，是表级的

AUTO_INCREMENT的列，插入的时候其他的事务就等待，锁只持续到sql 的末尾，而不是整个事务

- **插入意向锁，**高并发插入

同时写入同一索引间隙，不会阻塞



**死锁：**S锁 和S 锁不冲突，但是和X锁冲突，当两个事物同时获取了S 锁，又打算获取X锁修改数据的时候，就会相互等待

例：一个事物打算删除一条记录，另外两个事物打算插入这条记录，就会可能会发生死锁

原因，第一个事物先获取X 锁，另外两个事务需要先读才会发现键冲突，读就要获取S 锁，然后第一个事物提交了，由于S 锁互相兼容，两个事物同时获得S 锁，然后他们打算获取X 锁，进行修改操作，由于S 锁会阻塞 X 锁，所以两个事务就相互等待



**RR级别更新丢失**

因为select操作是读取快照，而update操作读取的是最新的数据，所以，当前事务更新数据的时候，其他事务已经修改过这条数据，这样当前事务的更新就 0 rows affects



**explain**

![img](http://note.youdao.com/yws/res/1878/6524A7BF358448659A928564D90CFA4D)

**id：**表的处理顺序，越大越先执行，小表驱动大表

- 当B表的数据集一定小于A表时，使用in好过exists

select * from A where id in (select id from B);

 

- 当A表的数据集小于B表是，使用exists好过in

select * from A where exists (select 1 from B where b.id = A.id);



**type：**

system （只有一条记录）> const （唯一索引 = 一个常量） > eq_ref （唯一索引一一对应，即外键关联）> ref （非唯一索引 = 一个常量）> range （in，between）> index（全索引） > ALL（全表），一般来说，得保证查询至少达到range级别，最好能达到ref



**exrta**

不要出现Using filesort，MySQL没办法用索引排序，只能用文件排序

和Using temporary，order by 或者 group by时，MySQL创建了临时表



**总结：**

1. 最佳左前缀，查询从索引列的最左边开始，不要跳过某一列，索引中的列都用上最好（全值匹配）
2. 使用覆盖索引，查询的列，索引上都有，避免访问数据
3. 索引列上不要用函数，WHERE left(NAME,4) = 'July';
4. 范围之后都失效
5. 左外链接的时候，索引建在右表上
6. 不等号，is null，is not null，like "%ada%"（%在左边失效），or连接条件，字符串不加引号，都会导致索引失效



**order by，group by**

使用最佳左前缀法则，索引不要断开，范围之后都失效



**慢日志**

slow_query_log=1;

slow_query_log_file=/var/lib/mysql/atguigu-slow.log

long_query_time=3;

log_output=FILE



**mysqldumpslow**

得到返回记录集最多的10个SQL

mysqldumpslow -s r -t 10 /var/lib/mysql/atguigu-slow.log

 

得到访问次数最多的10个SQL

mysqldumpslow -s c -t 10 /var/lib/mysql/atguigu-slow.log

 

得到按照时间排序的前10条里面含有左连接的查询语句

mysqldumpslow -s t -t 10 -g "left join" /var/lib/mysql/atguigu-slow.log