## 数据结构

> TreeMap实现，基于红黑树

## 源码分析

##### 成员变量

```java
public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable
{
    /**
     * The backing map.
     */
    private transient NavigableMap<E,Object> m;

    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();
}
```

##### 构造函数

```java
public TreeSet() {
	this(new TreeMap<E,Object>());
}
```

##### add

```java
public boolean add(E e) {
	return m.put(e, PRESENT)==null;
}
```

##### remove

```java
public boolean remove(Object o) {
	return m.remove(o)==PRESENT;
}
```

