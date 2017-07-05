## HashMap与LinkedHashMap迭代器

## 迭代器继承图

![](https://github.com/xuxh0622/learn-arrange/blob/master/source/ada.png)

![](https://github.com/xuxh0622/learn-arrange/blob/master/source/adb.png)

## HashMap迭代器

### HashIterator

> HashIterator是一个抽象类，封装了迭代器内部工作的一些操作，使用迭代器模式实现功能。提供hasNext和next方法供外面统一调用，隐藏内部实现的差异。

##### 属性

```java
abstract class HashIterator {
    // 下一个结点
    Node<K,V> next;        // next entry to return
    // 当前结点
    Node<K,V> current;     // current entry
    // 期望的修改次数
    int expectedModCount;  // for fast-fail
    // 当前桶索引
    int index;             // current slot
}
```

##### 构造函数

> next将表示第一个非空桶中的第一个结点，index将表示下一个桶。

```java
HashIterator() {
        // 成员变量赋值
        expectedModCount = modCount;
        Node<K,V>[] t = table;
        current = next = null;
        index = 0;
        // table不为空并且大小大于0
        if (t != null && size > 0) { // advance to first entry
            // 找到table数组中第一个存在的结点，即找到第一个具有元素的桶
            do {} while (index < t.length && (next = t[index++]) == null);
        }
    }
```

### 核心函数分析

##### hasNext函数

```java
// 是否存在下一个结点
public final boolean hasNext() {
    return next != null; 
}
```

##### nextNode函数

> nextNode函数屏蔽掉了桶的不同所带来的差异，就好像所有元素在同一个桶中，依次进行遍历。

```java
final Node<K,V> nextNode() {
    // 记录next结点
    Node<K,V> e = next;
    // 若在遍历时对HashMap进行结构性的修改则会抛出异常
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
    // 下一个结点为空，抛出异常
    if (e == null)
        throw new NoSuchElementException();
    // 如果再下一个结点为空，并且table表不为空；表示桶中所有结点已经遍历完，需寻找下一个不为空的桶
    if ((next = (current = e).next) == null && (t = table) != null) {
        // 找到下一个不为空的桶
        do {} while (index < t.length && (next = t[index++]) == null);
    }
    return e;
}
```

##### remove函数

```java
public final void remove() {
    Node<K,V> p = current;
    // 当前结点为空，抛出异常
    if (p == null)
        throw new IllegalStateException();
    // 若在遍历时对HashMap进行结构性的修改则会抛出异常
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
    // 当前结点为空
    current = null;
    K key = p.key;
    // 移除结点
    removeNode(hash(key), key, null, false, false);
    // 赋最新值
    expectedModCount = modCount;
}
```

### KeyIterator

> KeyIterator类是键迭代器，继承自HashIterator，实现了Iterator接口，可以对HashMap中的键进行遍历。

```java
final class KeyIterator extends HashIterator
    implements Iterator<K> {
    public final K next() { return nextNode().key; }
}
```

## LinkedHashIterator

> 与上面区别就是下一个元素通过链表获取。