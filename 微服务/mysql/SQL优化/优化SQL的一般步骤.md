**1.通过show status 了解各种SQL的执行频率**
show [session|global] status来显示session级或者gobal级的统计结果。不写默认session。

```
show status like 'Com_%';
```
Com_xxx表示xxx语句执行次数，比较关心以下参数：
- Com_select:执行select次数；
- Com_insert
- Com_update
- Com_delete
以上参数对所有存储引擎表操作累计，以下只针对InnoDB存储引擎：
- Innodb_rows_read;
- Innodb_rows_inserted;
- Innodb_rows_updated;
- Innodb_rows_deleted;
对于事务型应用，通过Com_commit和Com_rollback可以了解事务提交和回滚情况。
以下参数了解数据库基本情况：
- Connections:试图连接MySQL服务器次数
- Uptime：服务器工作时间
- Slow_queries：慢查询次数


**2.定位执行效率低的SQL语句**
- 通过慢查询日志定位执行效率低的SQL语句，用--log-slow-queries[=file_name]选项启动时，MySQL写一个包含所有执行时间超过long_query_time秒的SQL语句的日志文件；
- 慢查询日志在查询结束后才记录，所以在应用反映执行效率出现问题时日志并不能定位问题，可以使用show processlist查看当前MySQL在进行的线程，包括线程的状态、是否锁表等。可以实时查看SQL执行情况，同时对表操作进行优化。

**3.通过explain分析低效SQL执行计划**
通过explain或者desc获取MySQL如何执行select语句的信息，包括select语句执行过程中表如何连接和连接顺序。
![](https://upload-images.jianshu.io/upload_images/9449419-6837c86ba65af4f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- select_type:表示select类型。常见simple（简单表，不使用连接或者子程序）、primary（主查询，即外层查询）、union（union中的第二个或者后面的查询语句）、subquery（子程序的第一个select）等；
- table：输出结果集表
- type：表示MySQL中表中找到所需方式或者访问类型。
![常见访问类型](https://upload-images.jianshu.io/upload_images/9449419-61527e43f7db185d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
从左至右，性能由差到好：
（1）type=ALL,全表扫描，MySQL遍历来找到匹配行；
![](https://upload-images.jianshu.io/upload_images/9449419-80748f6f5c1959bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
（2）type=index，索引全扫描，MySQL遍历整个索引来查询匹配行；
（3）type=range，索引范围扫描，常见<、<=、>、>=、between等操作符。
（4）type=ref，使用非唯一索引扫描或者唯一索引的前缀扫描，返回匹配某个单独值的记录行。
（5）type=eq_ref,区别于ref在于使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，就是多表连接中使用primary key或者unique index作为关联条件。
（6）type=const/system，单表中最多有一个匹配行，查询起来非常迅速，所以整个匹配行中的其他列值可以被优化器在当前查询中当作常量处理。
（7）type=NULL，MySQL不用访问表或者索引，直接能够得到结果。
- possible_keys：表示查询时可能使用的索引
- key：表示实际使用索引
- key_len:使用索引字段长度
- rows：扫描行数量
- Extra：执行情况的说明和描述，包含不适合其他列中显示但是对执行计划非常重要的额外信息。

explain extended命令，通过explain extended加上show warnings，可以看到在SQL真正被执行前的优化器做的改写。在遇到复杂SQL时，可以利用MySQL extended的结果获取一个更加清晰的SQL。

通过explain partitions查看SQL访问的分区。

**4.通过show profile分析SQL**
通过参数have_profiling,能够看到当前MySQL是否支持profile。默认profiling是关闭的，可以使用set语句在session级别开启profiling；
```
select @@profiling;

set profiling=1;
```
![show profiles](https://upload-images.jianshu.io/upload_images/9449419-ca4ccd8ad015661b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![查询query id为2的执行过程中的线程状态](https://upload-images.jianshu.io/upload_images/9449419-8e438e72c6933c72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
Sending data状态标识MySQL线程开始访问数据行并把结果返回给客户端，需要做大量磁盘读取操作。可以查询INFORMATION_SCHEMA.PROFILING表并按时间排序。

获取状态后，可以进一步选择all、cpu、block、context switch、page faults等明细类型查看MySQL在使用什么资源消耗时间高。
```
show profile cpu for query 2;
```
**5.通过trace分析优化器如何选择执行计划**
通过trace文件能够进一步了解优化器行为。
使用方式：首先打开trace，设置格式为json，设置trace最大内存；
```
set OPTIMIZER_TRACE="enabled=on",END_MARKERS_IN_JSON=on;

set OPTIMIZER_TRACE_MAX_MEN_SIZE=1000000;
```
接下来执行想做trace的SQL语句。
最后检查INFORMATION_SCHEMA.OPTIMIZER_TRACE就可以知道MySQL如何执行的。

**6.确定问题并采取相应优化措施**
