继承自 AbstractSequentialList(一套基于顺序list访问的接口)，子类仅需实现部分代码即可拥有完整的一套访问某种序列表（比如链表）的接口, 而AbstractSequentialList 提供的方法基本上都是通过 ListIterator 实现的，比如：

```java
public E get(int index) {
    try {
        return listIterator(index).next();
    } catch (NoSuchElementException exc) {
        throw new IndexOutOfBoundsException("Index: "+index);
    }
}

public void add(int index, E element) {
    try {
        listIterator(index).add(element);
    } catch (NoSuchElementException exc) {
        throw new IndexOutOfBoundsException("Index: "+index);
    }
}

// 留给子类实现
public abstract ListIterator<E> listIterator(int index);
```
  
