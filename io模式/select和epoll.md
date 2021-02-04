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
