## 阻塞io模型
  ![zuse](https://github.com/einQimiaozi/awesome_java_notebook/blob/main/io%E6%A8%A1%E5%BC%8F/resources/zuseio.jpg)
  - 进程或线程等待某个条件，如果条件不满足，则一直等下去。条件满足，则进行下一步操作
## 非阻塞io模型
  ![nozuse](https://github.com/einQimiaozi/awesome_java_notebook/blob/main/io%E6%A8%A1%E5%BC%8F/resources/nozuseio.jpg)
  - 应用进程与内核交互，目的未达到时，不再一味的等着，而是直接返回。然后通过轮询的方式，不停的去问内核数据准备好没
## 多路复用io模型
  ![iofuyong](https://github.com/einQimiaozi/awesome_java_notebook/blob/main/io%E6%A8%A1%E5%BC%8F/resources/iofuyong.jpg)
  - select 有三个文件描述符集，分别是可读文件描述符集（readfds）、可写文件描述符集（writefds）和异常文件描述符集（exceptfds）
  - 应用程序可将某个 socket （文件描述符）设置到感兴趣的文件描述符集中，并调用 select 等待所感兴趣的事件发生
  - 应用进程会将多个 socket 设置到感兴趣的文件描述符集中，并调用 select 等待所关注的事件（比如可读、可写）处于就绪状态。当某些 socket 处于就绪状态后，select 返回处于就绪状态的 sockct 数量。注意这里返回的是 socket 的数量，并不是具体的 socket。应用程序需要自己去确定哪些 socket 处于就绪状态了，确定之后即可进行后续操作
  - 注意，多路复用的优点并不是处理单个连接能够更快，而是能够处理更多的连接
## 信号驱动io模型
  ![xinhaoqudong](https://github.com/einQimiaozi/awesome_java_notebook/blob/main/io%E6%A8%A1%E5%BC%8F/resources/xinhaoqudongio.jpg)
  - 应用进程告诉内核，如果某个 socket 的某个事件发生时，请向我发一个信号。在收到信号后，信号对应的处理函数会进行后续处理
  - 该模型和多路复用的区别在于，信号到来之前，用户进程不需要等待，可以去做别的事情
## 异步io模型
  ![yibu](https://github.com/einQimiaozi/awesome_java_notebook/blob/main/io%E6%A8%A1%E5%BC%8F/resources/yibuio.jpg)
  - 用户进程把文件描述符、数据缓存空间，以及信号告诉内核，当文件描述符处于可读状态时，内核会亲自将数据从内核空间拷贝到应用进程指定的缓存空间呢。拷贝完在告诉进程 I/O 操作结束，你可以直接使用数据了
  - 该模型的优点在于，除了发送信号以外，全程的数据传输由内核控制，用户进程可以不需要等待
  - 和之前四个模型的区别在于，前四个模型在信号到来的时候都需要阻塞以等待内核空间将数据拷贝到用户空间，异步io在这一步也不需要阻塞，有时间的时候去取数据就行了
