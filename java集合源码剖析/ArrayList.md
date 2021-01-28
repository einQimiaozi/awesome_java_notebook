## 构造

1.ArrayList 有两个构造方法，一个是无参，另一个需传入初始容量值。

2.无参数构造方法会构造一个大小为10的空object数组，10为默认长度

## 基本信息

1.使用int类型的变量size记录容器大小

2.在ArrayList中除了默认长度以外，还有一个必要长度信息，即minCapacity，在很多操作前会对这两个数值进行检查

3.Arraylist的最大容量为Integer.MAX_VALUE-8，其中的8为虚拟机中用于保留头信息的文字，防止内存溢出

## modCount

这个变量用于记录ArrayList结构变化的次数，因为容器在迭代的同时不禁止删除增加等操作，并且java原生集合是非线程安全的，所以这些操作有可能带来迭代错误

线程之间通过检查modCount是否匹配来确定是否存在错误，如果不匹配就出发fail-fast机制，抛出concurrentModifiedException异常

## add操作

1.检查数组容量，如果为空则初始化为10

2.modCount++

3.如果长度超过当前容量，则按照1.5倍扩容，如果使用了扩容函数(grow方法，该方法扩容的下界为minCapacity,上界为Integer.MAX_VALUE-8,扩容时的拷贝操作使用Arrays.copyOf方法，该方法内部使用System.arraycopy实现数组拷贝)，并且传入扩容参数大于1.5，那么按照传入参数扩容

如果不需要扩容并在末尾添加，则时间复杂度为O(1)，如果不再末尾添加(需要移动数组)，则时间复杂度为O(n)

## 删除操作

1.modCount++

2.检查索引是否合法

3.删除索引对应元素并使用system.arraycopy对后面的数组进行移动，时间复杂度为O(n)

注意，ArrayList本身没有自动缩容机制，如果一个容量很大的ArrayList在大量删除后会造成空间浪费，此时可以手动使用trimToSize()方法缩容

该方法内部也进行了modCount++，并且使用Arrays.copyOf方法进行数组拷贝缩容(是拷贝的，不是移动数组)

## 查找操作

由于ArrayList底层使用数组实现，所以查找直接返回索引就行了

## 遍历

ArrayList实现了RandomAccess接口，关于这个接口在java基础部分有说明，通过该接口标注ArrayList具有随机访问能力，所以虽然支持foreach遍历，但是使用下标+get遍历的效率会更高

## traisient修饰ELEMENT_DATA的理由

ArrayList的底层数组是被transient修饰的(非持久化，序列化后不可访问)

因为ArrayList的动态扩容机制会带来底层数组的冗余，而ArrayList的序列化是将数据写入ObjectOutputStream，反序列化是将数据从ObjectInputStream中读取size和element本身来恢复ELEMENT_DATA

所以本质上，ArrayList的序列化过程，根本不需要保存ELEMENT_DATA!!!!使用transient就可以在序列化的时候只保留size和element，不保留ELEMENT_DATA，节省空间

## 其他总结

ArrayList线程不安全

实现RandomAccess接口，具有随机访问能力

底层使用数组

扩容依靠拷贝而不是移动数组，性能消耗大，没有自动缩容能力


