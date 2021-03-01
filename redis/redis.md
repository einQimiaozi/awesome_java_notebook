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

## redis过期删除策略

1.为什么要设置过期事件：因为内存有限

2.如何判断数据过期：使用redis的过期字典(hashtable结构)，该字典的key是数据库中的某个key，value是该key的过期事件

3.惰性删除：取key的时候对数据进行过期检查，cpu消耗小，容易产生大量过期数据

4.定期删除：每隔一段时间按抽取一批key执行过期检查，并且redis底层会通过限制删除执行时长和频率来减少cpu消耗

5.内存淘汰：redis支持8种内存淘汰，防止使用惰性删除和定期删除依然会漏删造成大量过期数据引发oom的问题，其中volatile是针对在过期字典中的数据，allkeys是针对整个键空间的
  - 1.volatile：最近最少使用的淘汰
  - 2.volatile-ttl:快过期的淘汰
  - 3.allkeys-lru:最少使用的淘汰(这个是最常用的方法)
  - 4.allkeys-random：任意淘汰数据，注意这个任意淘汰是不看过期时间的，也就是说哪怕你这个数据没有在过期字典里，也有可能被淘汰
  - 5.no-eviction:当内存不足时新写入直接报错(这个方法基本没人用)
  - 6.vloatile-lfu:淘汰最不经常使用的
  - 7.allkeys-lfu:淘汰最不经常使用的

## redis的单线程事件处理模型(文件事件处理器)

1.通过io多路复用程序监听大量连接(多个socket)，并根据socket目前执行的任务来为socket关联不同的事件处理器

2.当被监听的套接字准备好执行accpet read write close 等操作时会产生对应的文件事件，文件事件处理器调用对应事件处理器来处理

3.文件事件处理器的结构

![shujianchuliqi](https://github.com/einQimiaozi/awesome_java_notebook/blob/main/redis/Resources/shijianchuliqi.jpg)
  
4.为什么不使用多线程
  - 实际上redis6.0之后引入多线程了
  - 使用单线程的原因有三个：1.单线程编程容易维护 2.redis的性能瓶颈不在cpu而在内存和网络 3.单线程不存在死锁，上下文切换的问题
  - 6.0后引入多线程的理由：印度的多线程实际上应用在网络数据的读写上，执行命令仍然使用单线程，主要是为了解决多线程网络的io瓶颈问题
  - 引入的多线程其实就是一堆io线程，并且io线程执行的时候，执行命令的主线程是阻塞的！！！

5.多线程下的情况
  - 1.使用一个等待列表记录socket
  - 2.主线程获取可读socket，如果等待列表不满就加入等待列表，否则就将等待列表中的socket和多个io线程绑定，之后主线程阻塞
  - 3.io线程读取socket并解析其请求
  - 4.所有socket读取完毕后主线程在顺序执行请求命令并将数据写入缓冲区
  - 5.socket的回写也是类似的执行，在回写和读都执行完毕后清空等待队列并返回主线程

![duoxiancheng](https://github.com/einQimiaozi/awesome_java_notebook/blob/main/redis/Resources/duoxiancheng.jpg)




