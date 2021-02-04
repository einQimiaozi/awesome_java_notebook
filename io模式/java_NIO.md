javaNIO的核心就是 Buffer、Channel 和 Selector

## Buffer

Buffer是javaNIO中常用的一个缓冲区类，被多个子类继承，其中包括ByteBuffer、CharBuffer、IntBuffer、FloatBuffer等等

Buffer 本质就是一个数组

重要属性
  - capacity容量：Buffer 所能容纳数据元素的最大数量，也就是底层数组的容量值。在创建时被指定，不可更改。
  - position位置：下一个被读或被写的位置，读写的时候需要经常调整
  - limit上界：可供读写的最大位置，用于限制 position，position < limit
  - mark标记：位置标记，用于记录某一次的读写位置，可以通过 reset 重新回到这个位置
  
注意，clear方法的本质就是调整position位置，不会冲洗缓冲区内容

## 为什么要用缓冲区

1.用户缓冲区是为了减少系统调用的次数，降低用户态和内核态之间的切换消耗

2.进程和内核都有自己的缓冲区
  
  
