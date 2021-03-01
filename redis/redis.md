## sql和nosql的区别

1.sql是多个二维表格，不同的表格之间的关系组成了一个数据库，使用关系模型构建

2.nosql是分布式的，不保证遵循acid，一般以键值对存储，结构不固定

## redis和Memcached的区别

1.memcached只支持kv数据类型，redis除了kv还支持list，set，zset，hash等数据结构

2.redis支持数据持久化，memcached不支持

3.redis有灾难恢复机制

4.redis内存用完之后会使用磁盘，memcached直接报错

5.memcached原生不支持集群，redis原生支持

6.memcached采用aio网络模型，redis则用的是nio

7.redis支持发布订阅，lua脚本之类的功能

8.memcached过期数据采用惰性删除策略，redis采用惰性删除和定期删除策略

9.redis使用单核，memcached使用多核，所以redis存储大数据其实不如memcached性能好

## redis数据类型

1.string：kv结构，使用c语言编写，但是不是c语言的char数组，二十自己构建的动态字符串SDS，可以保存文本和二进制数据，获取字符串长度的时间复杂度为O(1)，不会造成缓冲区溢出，常用于计数

2.list：双向链表，支持反向查找和反向遍历

3.hash：类似java中的hashmap，kv都是字符串，比较适合存储对象(k:对象 v:对象信息)

4.set：无序去重集合，适合存储需要快速去重的数据，集群的时候非常好用，比如你的数据要去重，但是不管你怎么做一台机器上只能去重自己的数据，这个时候就需要使用set了

5.sorted set：使用权重参数score排序的set，底层使用跳跃表实现

6.跳跃表：跳跃表就是个多指针有序链表，具体来说就是把链表中取几个节点做成第二个链表，第二个链表中再取几个节点做成第三个链表，节点数量上：表3<表2<表1，搜索某个节点可以从表3开始，如果小于当前节点则顺序向前搜索，如果大于当前节点则转移到表2向前搜索，以此类推，可以有效减少查找次数，一般适用于需要根据某个值进行排序的集合，比如直播中的礼物榜单信息(sroce=礼物，数据=送礼者信息)

![jumptable](https://github.com/einQimiaozi/awesome_java_notebook/blob/main/redis/Resources/jumptable.jpg)

大概意思就上面这样，比如要查15,顺序是1->24 退回 1->9->24 退回9->15 ，一共查了5次，其实顺序查找只需要4次，但是数据如果多的话就会快很多，并且很多时候查找不需要走到最下面的链表，复杂度是log(n)，一般用这种结构可以代替avl树和红黑树，因为原理简单一些，并且插入删除不需要旋转树，速度更快

## redis查询流程

![redis](https://github.com/Snailclimb/JavaGuide/blob/master/docs/database/Redis/images/redis-all/%E7%BC%93%E5%AD%98%E7%9A%84%E5%A4%84%E7%90%86%E6%B5%81%E7%A8%8B.png)


