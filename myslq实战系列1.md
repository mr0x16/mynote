---
typora-root-url: D:\mynote\assets
---

# mysql实战系列1

## 基础架构

> server层：连接器、查询缓存、分析器、优化器、执行器
>
> 存储引擎层：负责数据的存储和提取，架构是插件式的支持InnoDB、MyISAM、Memory等多个存储引擎

![1542604390736](/1542604390736.png)

1. 连接器：负责和客户端建立连接、获取权限、维持和管理连接

   > 利用show processlist查看连接状态
   >
   > 客户端空闲时间太长，连接器会自动断开，由时间参数wait_timeout控制，默认值是8小时
   >
   > 在使用中尽量多的使用长连接，但是全部使用长连接后，会导致mysql占用内存上涨很快，因为mysql在执行过程中临时使用的内存是管理在连接对象里面的。这些资源只有在连接断开是才能被释放，此时可以考虑以下两种解决方案：
   >
   > * 定期断开长连接。使用一段时间，或者程序里面判断执行过一个占用内存的大查询后，断开连接，之后要查询再重连。
   > * 如果用的是mysql5.7或者更新的版本，可以在每次执行一个比较大的操作后，通过执行mysql_reset_connection来重新初始化连接资源。这个过程不需要重连和重新做权限验证，但是会将连接恢复到刚刚创建完成是的状态

2. 查询缓存：mysql拿到查询后，会先到查询缓存看看，之前是不是执行过这条语句。

   > 之前执行过的查询语句会以：key-value的形式存储在缓存中，key是查询语句，value是查询结果
   >
   > 对于非静态表，查询缓存的失效概率很大，使用起来往往**弊大于利**
   >
   > 可以将参数query_cache_type设置成DEMAND，这样默认不适用查询缓存，对于需要使用查询缓存的，可以使用`SQL_CACHE`关键词，例如`select SQL_CACHE * from T where ID=10`
   >
   > 需要注意的是，**mysql8.0**版本直接将查询缓存整块功能删掉了。。。

3. 解析器：告诉mysql，sql语句该做什么。先对sql语句做“词法分析“识别关键词、表名、列名等，之后对语句进行”语法分析“，判断输入的sql语句是否满足sql语法

4. 优化器：告诉mysql，sql语句该怎么做

   > 例如在表里面有多个索引的时候，决定使用哪个索引；或者在一个语句有多表关联（join）的时候，决定各个表的连接顺序。

5. 执行器：执行sql语句

   > * 判断用户对表有无执行权限
   > * 打开表，根据引擎定义，使用引擎提供的接口开始执行
   >
   > 如果要执行`select * from T where ID=10`
   >
   > 则需要一下三步：
   >
   > 1. 调用InnoDB引擎接口，取这个表的第一行，判断ID值是不是10，如果不是则跳过，如果是则将这行存在结果集中
   > 2. 调用引擎接口取“下一行”，重复相同的判断逻辑，直到取到这个表的最后一行
   > 3. 执行器将上述遍历过程中所有满足条件的行租场记录集违结果集返回给客户端

## 日志系统

以`update T set c=c+1 where ID=2`为例

1. redolog

   > 先将需要更新的记录记录在redolog中，并更新内存，在系统空闲时写入到磁盘中。有redolog的缘故，InnoDB可以保证crash-safe
   >
   > innodb_flush_log_at_trx_commit这个参数设置成1的时候，表示每次事务的redo log都直接持久化到磁盘，这个参数建议设置成1，宝成mysql异常重启后数据不丢失

2. binlog（归档日志）：mysql中server层的日志

   > sync_binlog这个参数建议设置成1，表示每次事务的binlog都持久化到磁盘。建议设置成1，这样保证mysql异常重启后binlog不丢失。
   >
   > 有两种模式，statment格式只记录了sql语句，row格式会记录行内容，更新之前和更新之后的

两种日志有一下三点不同：

1. redolog是InnoDB引擎特有的；binlog是mysql的server层实现的，所有引擎都可以使用。
2. redolog是物理日志，记录的是“在某个数据页上做了什么修改”；binlog是逻辑日志，记录的是这个语句的原始逻辑，比如“给ID=2这一行的c字段加1”

3. redolog是循环写的，空间固定会用完并覆盖；binlog是可以追加写入的。“追加写”是指binlog文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。

以此为基础，上面语句的执行流程为：

> * 执行器先找引擎取ID=2这一行。ID是逐渐，引擎直接用树搜索找到这一行。如果ID=2这一行所在的这一行本来就在内存中，就直接返回给执行器；否则，需要先从磁盘读入内存，然后再返回。
> * 执行器拿到引擎给的行数据，把这个值加上1，得到新的一行数据，再调用引擎接口写入这行新数据。
> * 引擎将这行新数据更新到内存中，同时将这个更新操作记录到redo log里面，此时redo log处于prepare状态。然后告知执行器执行完成，随时可以提交事务。
> * 执行器生成这个操作的binlog，并把binlog写入磁盘。
> * 执行器调用引擎的提交事务接口，引擎把刚刚写入的redo log改成提交(commit)状态，更新完成

如下图所示：

![1542677496083](/1542677496083.png)

**两阶段提交**

上说过程中将redo log的写入分成两个步骤：prepare和commit，这就是”两阶段提交“。

> 目的是让redo log和binlog之间的逻辑保持一致

