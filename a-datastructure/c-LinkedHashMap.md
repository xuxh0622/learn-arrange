## 数据结构

![](https://github.com/xuxh0622/learn-arrange/blob/master/source/aca.png))

> 数组 + 单链表 + 红黑树 + 双链表综合所得，双链表存储插入顺序。

## 源码分析

##### 类属性

> 由于继承HashMap，所以HashMap中的非private方法和字段，都可以在LinkedHashMap直接中访问。

```java
public class LinkedHashMap<K,V>  extends HashMap<K,V> implements Map<K,V> {
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
    // 版本序列号
    private static final long serialVersionUID = 3801124242820219131L;

    // 链表头结点
    transient LinkedHashMap.Entry<K,V> head;

    // 链表尾结点
    transient LinkedHashMap.Entry<K,V> tail;

    // 访问顺序
    final boolean accessOrder;
}
```

##### 构造函数

> accessOrder默认为false，access为true表示之后访问顺序按照元素的访问顺序进行，即不按照之前的插入顺序了，access为false表示按照插入顺序访问。

```java
public LinkedHashMap(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor);
        accessOrder = false;
}
```

## 重要函数分析

##### newNode

> HashMap里面put方法初始化一个节点，此时子类将该节点添加链表

```java
// 当桶中结点类型为HashMap.Node类型时，调用此函数
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    // 生成Node结点
    LinkedHashMap.Entry<K,V> p =
        new LinkedHashMap.Entry<K,V>(hash, key, value, e);
    // 将该结点插入双链表末尾
    linkNodeLast(p);
    return p;
}
```

##### newTreeNode

```java
// 当桶中结点类型为HashMap.TreeNode时，调用此函数
TreeNode<K,V> newTreeNode(int hash, K key, V value, Node<K,V> next) {
    // 生成TreeNode结点
    TreeNode<K,V> p = new TreeNode<K,V>(hash, key, value, next);
    // 将该结点插入双链表末尾
    linkNodeLast(p);
    return p;
}
```

##### afterNodeAccess

```java
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    // 若访问顺序为true，且访问的对象不是尾结点
    if (accessOrder && (last = tail) != e) {
        // 向下转型，记录p的前后结点
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        // p的后结点为空
        p.after = null;
        // 如果p的前结点为空
        if (b == null)
            // a为头结点
            head = a;
        else // p的前结点不为空
            // b的后结点为a
            b.after = a;
        // p的后结点不为空
        if (a != null)
            // a的前结点为b
            a.before = b;
        else // p的后结点为空
            // 后结点为最后一个结点
            last = b;
        // 若最后一个结点为空
        if (last == null)
            // 头结点为p
            head = p;
        else { // p链入最后一个结点后面
            p.before = last;
            last.after = p;
        }
        // 尾结点为p
        tail = p;
        // 增加结构性修改数量
        ++modCount;
    }
}
```

![](https://github.com/xuxh0622/learn-arrange/blob/master/source/acb.png))

##### transferLinks

> 替换对象，只需修改链表

```java
// 用dst替换src
private void transferLinks(LinkedHashMap.Entry<K,V> src,
                               LinkedHashMap.Entry<K,V> dst) {
    LinkedHashMap.Entry<K,V> b = dst.before = src.before;
    LinkedHashMap.Entry<K,V> a = dst.after = src.after;
    if (b == null)
        head = dst;
    else
        b.after = dst;
    if (a == null)
        tail = dst;
    else
        a.before = dst;
}
```

![](https://github.com/xuxh0622/learn-arrange/blob/master/source/acc.png))

##### containsValue

> containsValue函数根据双链表结构来查找是否包含value，是按照插入顺序进行查找的

```java
public boolean containsValue(Object value) {
    // 使用双链表结构进行遍历查找
    for (LinkedHashMap.Entry<K,V> e = head; e != null; e = e.after) {
        V v = e.value;
        if (v == value || (value != null && value.equals(v)))
            return true;
    }
    return false;
}
```





