## 前言

> 对树进行排序，maps.entrySet()获取的值排序之后的。

## 数据结构

![](https://github.com/xuxh0622/learn-arrange/blob/master/source/aea.png)

## 源码分析

##### 属性

> 继承了抽象类AbstractMap，AbstractMap实现了Map接口，实现了部分方法。不能进行实例化，实现了NavigableMap,Cloneable,Serializable接口，其中NavigableMap是继承自SortedMap的接口，定义了一系列规范。

```java
public class TreeMap<K,V>
    extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable
{
    // 比较器，用于控制Map中的元素顺序
    private final Comparator<? super K> comparator;
    // 根节点
    private transient Entry<K,V> root;
    // 树中结点个数
    private transient int size = 0;
    // 对树进行结构性修改的次数
    private transient int modCount = 0;
}
```

## 核心函数

##### put函数

> 插入一个元素时，若用户自定义比较器，则会按照用户自定义的逻辑确定元素的插入位置，否则，将会使用K自身实现的比较器确定插入位置

```java
public V put(K key, V value) {
        // 记录根节点
        Entry<K,V> t = root;
        // 根节点为空
        if (t == null) {
            // 比较key
            compare(key, key); // type (and possibly null) check
            // 新生根节点
            root = new Entry<>(key, value, null);
            // 大小加1
            size = 1;
            // 修改次数加1
            modCount++;
            return null;
        }
        int cmp;
        Entry<K,V> parent;
        // 获取比较器
        Comparator<? super K> cpr = comparator;
        // 比较器不为空
        if (cpr != null) {
            // 找到元素合适的插入位置
            do {
                // parent赋值
                parent = t;
                // 比较key与元素的key值，在Comparator类的compare方法中可以实现我们自己的比较逻辑
                cmp = cpr.compare(key, t.key);
                // 小于结点key值，向左子树查找
                if (cmp < 0)
                    t = t.left;
                // 大于结点key值，向右子树查找
                else if (cmp > 0)
                    t = t.right;
                // 表示相等，直接更新结点的值
                else
                    return t.setValue(value);
            } while (t != null);
        }
        // 比较器为空
        else {
            // key为空，抛出异常
            if (key == null)
                throw new NullPointerException();
            @SuppressWarnings("unchecked")
                // 取得K实现的比较器
                Comparable<? super K> k = (Comparable<? super K>) key;
            // 寻找元素插入位置
            do {
                parent = t;
                cmp = k.compareTo(t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        // 新生一个结点
        Entry<K,V> e = new Entry<>(key, value, parent);
        // 根据比较结果决定存为左结点或右结点
        if (cmp < 0)
            parent.left = e;
        else
            parent.right = e;
        // 插入后进行修正
        fixAfterInsertion(e);
        // 大小加1
        size++;
        // 进行了结构性修改
        modCount++;
        return null;
    }
```

##### getEntry函数

> 当我们调用get函数时，实际上是委托getEntry函数获取元素，对于用户自定义实现的Comparator比较器而言，是使用getEntryUsingComparator函数来完成获取逻辑。

```java
final Entry<K,V> getEntry(Object key) {
        // 判断比较器是否为空
        if (comparator != null)
            // 根据自定义的比较器来返回结果
            return getEntryUsingComparator(key);
        // 比较器为空
        // key为空，抛出异常
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
            // 取得K自身实现了比较接口
            Comparable<? super K> k = (Comparable<? super K>) key;
        Entry<K,V> p = root;
        // 根据Comparable接口的compareTo函数来查找元素
        while (p != null) {
            int cmp = k.compareTo(p.key);
            if (cmp < 0)
                p = p.left;
            else if (cmp > 0)
                p = p.right;
            else
                return p;
        }
        return null;
    }
```

```java
final Entry<K,V> getEntryUsingComparator(Object key) {
        @SuppressWarnings("unchecked")
            // 向下转型
            K k = (K) key;
        // 取得比较器
        Comparator<? super K> cpr = comparator;
        // 比较器不为空
        if (cpr != null) {
            Entry<K,V> p = root;
            // 开始遍历树节点找到对应的结点
            while (p != null) {
                int cmp = cpr.compare(k, p.key);
                // 小于结点key值，向左子树查找
                if (cmp < 0)
                    p = p.left;
                // 大于结点key值，向右子树查找
                else if (cmp > 0)
                    p = p.right;
                // 相等，找到，直接返回
                else
                    return p;
            }
        }
        return null;
    }
```

##### deleteEntry函数

> deleteEntry函数会在remove函数中被调用，它完成了移除元素的主要工作，删除该结点后会对红黑树进行修正，此部分内容以后会详细讲解，同时，在此函数中需要调用successor函数，即找到该结点的后继结点。具体函数代码如下　

```java
private void deleteEntry(Entry<K,V> p) {
        // 结构性修改
        modCount++;
        // 大小减1
        size--;
        // p的左右子结点均不为空
        if (p.left != null && p.right != null) {
            // 找到p结点的后继
            Entry<K,V> s = successor(p);
            // 将p的值用其后继结点的key-value替换，并且用s指向其后继
            p.key = s.key;
            p.value = s.value;
            p = s;
        } 

        // 开始进行修正，具体的修正过程我们会在之后的数据结构专区进行讲解
        // 现在可以看成是为了保持红黑树的特性，提高性能
        Entry<K,V> replacement = (p.left != null ? p.left : p.right);

        if (replacement != null) {
            // Link replacement to parent
            replacement.parent = p.parent;
            if (p.parent == null)
                root = replacement;
            else if (p == p.parent.left)
                p.parent.left  = replacement;
            else
                p.parent.right = replacement;

            // Null out links so they are OK to use by fixAfterDeletion.
            p.left = p.right = p.parent = null;

            // Fix replacement
            if (p.color == BLACK)
                fixAfterDeletion(replacement);
        } else if (p.parent == null) { // return if we are the only node.
            root = null;
        } else { //  No children. Use self as phantom replacement and unlink.
            if (p.color == BLACK)
                fixAfterDeletion(p);

            if (p.parent != null) {
                if (p == p.parent.left)
                    p.parent.left = null;
                else if (p == p.parent.right)
                    p.parent.right = null;
                p.parent = null;
            }
        }
    }
```

```java
// 找到后继
    static <K,V> TreeMap.Entry<K,V> successor(Entry<K,V> t) {
        // t为null，直接返回null
        if (t == null)
            return null;
        // 右孩子不为空
        else if (t.right != null) {
            // 找到右孩子的最底层的左孩子，返回
            Entry<K,V> p = t.right;
            while (p.left != null)
                p = p.left;
            return p;
        } else { // 右孩子为空
            // 保存t的父节点
            Entry<K,V> p = t.parent;
            // 保存t结点
            Entry<K,V> ch = t;
            // 进行回溯，找到后继，直到p == null || ch != p.right
            while (p != null && ch == p.right) {
                ch = p;
                p = p.parent;
            }
            return p;
        }
    }
```

##### getLowerEntry

```java
final Entry<K,V> getLowerEntry(K key) {
        // 保存根节点
        Entry<K,V> p = root;
        // 根节点不为空
        while (p != null) {
            // 比较该key与节点的key
            int cmp = compare(key, p.key);
            if (cmp > 0) { // 如果该key大于结点的key
                // 如果结点的右子树不为空，与该结点右结点进行比较
                if (p.right != null)
                    p = p.right;
                else // 右子树为空，则直接返回结点；因为此时已经没有比该结点key更大的结点了(右子树为空)
                    return p;
            } else { // 如果该key小于等于结点的key
                // 结点的左子树不为空，与该结点的左结点进行比较
                if (p.left != null) { 
                    p = p.left;
                } else { // 结点的左子树不为空，则开始进行回溯
                    Entry<K,V> parent = p.parent;
                    Entry<K,V> ch = p;
                    while (parent != null && ch == parent.left) {
                        ch = parent;
                        parent = parent.parent;
                    }
                    return parent;
                }
            }
        }
        return null;
    }
```

## 流程图

![](https://github.com/xuxh0622/learn-arrange/blob/master/source/aeb.png)