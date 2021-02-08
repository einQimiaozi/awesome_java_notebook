## 和hashmap的区别

hashmap和hashtable基本上一样，可以按照hashmap学习，只不过有些小的区别

1.hashtable和hashmap的构造类似，区别在于默认容量为11，因为质数可以最大程度的减少hash碰撞，并且Hashtable不要求底层数组的容量一定要为2的整数次幂

2.hashtable线程安全，具体做法就是给每个方法加上sync......

3.hashtable保留了contains方法，hashmap没保留，实际上hashtable的contasinkey方法调用了contains方法

4.Hashtable中key和value都不允许为null，而HashMap中key和value都允许为null（key只能有一个为null，而value则可以有多个为null）。但是如果在Hashtable中有类似put(null,null)的操作，编译同样可以通过，因为key和value都是Object类型，但运行时会抛出NullPointerException异常，这是JDK的规范规定的。

5.Hashtable扩容时，将容量变为原来的2倍加1



