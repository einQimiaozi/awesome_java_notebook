## 前置知识：

java方法和本地方法的区别：java方法是由由java语言编写，编译成字节码，存储在class文件中的。java方法是与平台无关的。本地方法是由其他语言（如C、C++ 或其他汇编语言）编写，编译成和处理器相关的代码。本地方法保存在动态连接库中，格式是各个平台专用的，运行中的java程序调用本地方法时，虚拟机装载包含这个本地方法的动态库，并调用这个方法。

## jvm,jdk,jre

区别：jvm是java虚拟机，jre是java类库api中的javase子集+jvm，jdk是java语言+jvm+java类库

所以jdk包含jre，jre包含jvm
