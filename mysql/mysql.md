## Mysql基础架构

![mysql](https://github.com/einQimiaozi/awesome_java_notebook/blob/main/mysql/Resources/mysql.jpg)

存储引擎默认是innoDB

1.连接器：负责客户端和server建立链接，维持，管理连接等
  - 应尽量减少连接动作，使用长链接
  - 长链接造成的OOM和内存占用过大可以通过定期断开重连和执行musql_reset_connection来重置链接资源

2.查询缓存
  - 因为数据表会经常更新，所以查询缓存失效频繁，不建议使用
  - 除非你的业务是一张静态表
  - mysql8.0中查询缓存功能被删除了

3.分析器：一般就是用来检查语法的

4.优化器：一般用来决定选择用哪个索引，决定多表关联下各个表的连接顺序，优化器不影响查询结果，只影响查询效率

5.执行器
  - 1.判断当前用户对要查询的表是否有查询权限
  - 2.有权限就根据表的引擎，使用这个引擎所提供的接口
  - 3.对值的查找和范围判断之类的都在这部分完成，返回结果也通过执行器

## redo log

1.redo log就是WAL(Write Ahead Logging)技术的实现方法，用于解决crash-safe(字面意思)，是innoDB特有的日志

2.sql语句执行的时候会把执行记录写入redo log，本质就是写入内存，等适当的时候一次性写入磁盘，当出现crash-safe的时候可以根据redo log记录的操作进行数据库回滚恢复

3.redo log的结构：
  - 1.大小固定，一组4个文件，每个文件1gb
  - 2.使用write pos指针记录当前位置，write pos边写边向后移动，checkpoint指针记录擦除位置，通俗的说chechpoint-write pos的空间就是当前空着的空间，可以写入
  - 3.redo log采用循环记录的方式，例如4个文件分别是abcd，d文件写完之后会回到a文件开头继续写，当write pos追上checkpoint位置的时候就是写满了，要对文件进行擦除，同时被擦除的部分要写入磁盘

4.物理日志，记录了“某个数据页上修改了什么”

## binlog

1.binlog是所有执行引擎都可以使用的日志

2.并不是循环记录的，而是不断追加写入的，不会覆盖以前的日志

3.逻辑日志，记录了“给id=2这一行的c字段+1”这种东西

4.只能用于归档，没有crash-safe能力

binlog和redo log两种日志一般一起使用，进行两阶段提交，两阶段提交的意思就是：先写入redo log，进入prepare阶段，然后写入binlog，然后让redo log进行commit，如果不进行两阶段提交而是先写一个再写一个的话会导致其中一个日志回复的数据和原库不同

innode_flush_log_at_trx_commit参数控制redo log的commit频率，建议设置为1,这样每次事务的redo log都会写入磁盘

sync_binlog同上



