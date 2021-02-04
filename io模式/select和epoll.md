## 工作队列和等待队列

![work](https://github.com/einQimiaozi/awesome_java_notebook/blob/main/io%E6%A8%A1%E5%BC%8F/resources/workqueue.jpg)

工作队列存在于内核空间，队列内的进程按顺序执行(并发的情况下就一个进程执行一小段时间)

![wait](https://github.com/einQimiaozi/awesome_java_notebook/blob/main/io%E6%A8%A1%E5%BC%8F/resources/waitqueue.jpg)

等待队列由socket对象控制，如果进程想要监听某个socket，就把自己加入到socket的等待队列中即可，注意进程被加入到等待队列=等待队列中拥有进程的引用，本质上进程还在工作队列中，只不过是阻塞挂起状态

## 基本的io流程

![1](https://github.com/einQimiaozi/awesome_java_notebook/blob/main/io%E6%A8%A1%E5%BC%8F/resources/1.jpg)

假设进程a b为socket处理进程，那么将进程a b先加入到socket的等待队列中，阻塞，此时工作队列执行进程c

待socket接收到数据data，想cpu发出中端信号，cpu中断在执行的进程，从接收缓冲区里接收数据交给进程a b处理

![2](https://github.com/einQimiaozi/awesome_java_notebook/blob/main/io%E6%A8%A1%E5%BC%8F/resources/2.jpg)

进程a b被唤醒回到工作队列处理进程，同时等待队列中a b的引用被删除

## select

当需要一个服务端同时监听多个客户端(socket)的时候，就需要对上面的流程进行改进

select使用了socket队列，将socket放入一个socket队列中，进程通过遍历将自己加入需要监听的socket中，在很多情况下，一个进程可能要监听全部socket

当其中某个socket接收到中断的时候就通过循环遍历整个socket队列找到该socket下对应的进程，并全部删除(就是说当一个进程监听了3个socket，socket1唤醒进程之后我们还要找到socket2和socket3,将进程从他们的等待队列里删除)

整个流程如下

![select](https://github.com/einQimiaozi/awesome_java_notebook/blob/main/io%E6%A8%A1%E5%BC%8F/resources/select.jpg)

缺点：如上图所示，加入等待队列和从等待队列里唤醒都需要遍历整个socket队列，select对socket队列长度的最大值规定为1024,即便如此时间复杂度依然很高

## epoll

epoll用于解决select效率低下的问题，采用了将维护等待队列和阻塞进程分开处理的方法

![epoll](https://github.com/einQimiaozi/awesome_java_notebook/blob/main/io%E6%A8%A1%E5%BC%8F/resources/epoll.jpg)

epoll对socket和进程的管理使用中介eventpoll完成，不让socket和进程之间相互直接操作

eventpoll中比较重要的部分就是epoll_ctl，epoll_wait和rdlist

epoll_ctl：用户维护等待队列

epoll_wait:用于唤醒和阻塞进程

rdlist：维护一个指向变化过的socket的引用列表

整个流程如下：

首先在初始化阶段将进程添加到希望监听的socket列表里，这部分和select一样，只不过通过eventpoll对象操作，同时维护一个rdlist

如果rdlist为空(没有socket接收被触发)，则epoll_wait就让进程处于阻塞状态

当socket收到数据后，首先将变化的socket加入rdlist中(通过Epitem对象间接引用)，后续从socket的等待列表中进行进程的删除和阻塞唤醒操作都根据rdlist中指向的socket来完成，这样避免了select中想要找到哪些socket有当前需要操作的进程就必须遍历的问题，因为只有变化过的socket才有可能需要对其等待列表中的进程进行才做

epoll中全部的操作都是依靠中断当前进程后操作eventpoll完成的

rdlist负责所有socket添加和删除的索引定位操作，内部使用双向链表实现

而存放socket引用的结构则是由红黑树完成

是否阻塞通过判断rdlist是否为空决定，不为空的话返回rdlist索引定位到的socket

注意，rdlist和等待队列一个负责socket一个负责进程
