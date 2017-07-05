## 数据结构

> 内部通过HashMap实现，（数组+链表+红黑树）综合所得。

## 源码分析

##### 成员变量

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    static final long serialVersionUID = -5024744406713321676L;

    private transient HashMap<E,Object> map;

    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();
}
```

##### 构造函数

```java
public HashSet() {
        map = new HashMap<>();
    }
```

##### add

> 把值放入key中，保证不重复

```java
//private static final Object PRESENT = new Object();
public boolean add(E e) {
  return map.put(e, PRESENT)==null;
}
```

##### remove

> 如果传入的o不存在，map的remove方法返回为null，则对应的结果是HashSet的remove操作应该放回false

```java
 public boolean remove(Object o) {
 	return map.remove(o)==PRESENT;
 }
```

