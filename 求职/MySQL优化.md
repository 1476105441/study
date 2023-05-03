场地管理怎么使用索引覆盖、索引条件下推？



# 索引覆盖

1、不使用索引覆盖，0.21s

~~~sql
select * from place where maximum = 11;
~~~

2、使用索引覆盖，0.043s

~~~sql
select maximum,name from place where maximum = 11;
~~~





# 索引条件下推

​	场地表中有70万条数据，name是场地名称，maximum是场地容量，现在要查找场地名称中带有“篮球”且最大容量是30的场地。

~~~sql
select * from `place` where `maximum` = 30 and name like '%篮球%';
~~~

1、不使用索引，全表扫描：耗时1.08s

2、使用索引，没有使用索引条件下推：耗时0.25s

~~~sql
create index maximum_idx on place(maximum);
~~~

3、使用索引条件下推：耗时0.02s

~~~SQL
create index maximum_name_idx on place (maximum,name);
~~~





# 数据库优化步骤

## 定位和分析

1、通过开启慢查询日志，查找性能瓶颈的SQL，设置慢查询日志的**long_query_time**属性，表示执行时间大于这个时间的SQL语句是慢SQL，默认值为10。

2、使用explain查看SQL的执行计划，或者使用show profile查看SQL中每一个步骤的时间成本，这样可以了解SQL查询慢是因为执行时间长还是等待时间长。

## 处理

1、如果是等待时间长，就可以调优服务器的参数，比如适当增加数据库缓冲池等。

2、如果是SQL执行时间长，就需要考虑是索引设计的问题，还是查询关联的数据表过多，或者是因为数据库表字段设计的不合理。

## 优化结构

如果上述处理都没有生效，需要考虑是否是数据库的SQL查询是否达到了瓶颈，如果没有达到性能瓶颈，就需要重新检查，重复以上步骤。

如果达到了性能瓶颈，则需要考虑增加服务器，采用读写分离的架构，或者考虑对数据库进行分库分表（垂直分库、垂直分表、水平分表等）。



## 实用指令

### 1、show status like '参数名'

可以查看系统记录的性能参数信息，例如：连接次数、慢查询次数、last_query_cost（上一次查询读取的页的数量）。

### 2、slow_query_log参数

是否开启慢查询日志，默认情况下，MySQL没有开启慢查询日志，需要手动开启。

开启指令：set global slow_query_log = 'on';

设置慢查询时间：set global long_query_time = 1;（需要注意的是设置全局的参数对当前的会话是不起作用的）

上述方法是临时性的开启慢查询日志，当MySQL重启后，配置就失效了，如果想要永久性的开启慢查询日志，则需要在MySQL配置文件（Linux系统中为my.cnf文件）中配置 slow_query_log = on

### 3、慢查询日志分析工具：mysqldumpslow

携带参数：

![image-20230212175134132](D:\影音图片\面试准备.assets\image-20230212175134132.png)

![image-20230212175147823](D:\影音图片\面试准备.assets\image-20230212175147823.png)

查询按查询耗时降序的前五条SQL：mysqldumpslow -s t -t 5 慢查询日志路径

查询返回记录集最多的10个SQL：mysqldumpslow -s r -t 10 日志路径

### 4、重新生成慢查询日志文件

使用此命令重新生成日志文件：mysqladmin -uroot -p flush-logs slow

### 5、查看SQL执行成本：show profile

查看show profile功能是否开启：show variables like 'profiling';

开启show profile功能：set profiling = 'on';

粗略查看当前会话执行过的SQL：show profiles;

详细查看最近执行的一条SQL的执行成本：show profile;

![image-20230212181050967](D:\影音图片\面试准备.assets\image-20230212181050967.png)

### 6、分析查询语句

explain select * from table;

重要的几个属性：

* type（查询的类型）
  * system：只有确定数量，并且数量很少的情况下是system，例如使用MyISAM存储引擎，创建一个只有一个数据的表，使用explain就会显示system。
  * const：使用主键查询。
  * eq_ref：使用主键进行表的连接匹配条件，驱动表的查询类型会是eq_ref。
  * ref：使用非主键索引查询。
  * ref_or_null：使用非主键索引并且or上为null的情况。
  * range：使用索引的范围查询。
  * index：使用非主键索引的全表扫描，不需要回表。

* key_len（使用的索引长度，使用联合索引时有用）
  
* extra（附加信息）
  * using where：使用where过滤，普通的where查询没有使用索引。
  * using index：使用索引查询。
  * using index condition：索引条件下推。
  * using filesort：使用文件排序，也就是没有使用上索引的排序，在内存中或者磁盘中的排序。
  * using temporary：创建临时表。




# 优化步骤2

1、建立索引（优化物理查询）

2、SQL优化——是否有太多的join或者子查询了（优化逻辑查询）

3、调整服务器参数——调整my.cnf

* innodb_buffer_pool_size：是最重要的参数之一，表示innodb的表和索引的最大缓冲池大小。
* key_buffer_size：表示索引缓冲区的大小，这个缓冲区是所有的线程共享的。
* table_cache：表示同时打开的表的个数。
* sort_buffer_size：表示每个需要进行排序的线程分配的缓冲区的大小，此缓冲区是每个连接一份。
* join_buffer_size：表示连接查询操作所能使用的缓冲区大小，也是每个连接一份缓冲区。
* innodb_flush_log_at_trx_commit：redo文件刷盘策略。
* innodb_log_buffer_size：innodb引擎的redo日志使用的缓冲区。

4、分库分表，读写分离
