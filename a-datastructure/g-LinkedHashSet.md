## 数据结构

> 继承HashSet，数组 + 单链表 + 红黑树 + 双链表的综合所得

## 源码分析

##### 成员变量

```java
public class LinkedHashSet<E>
    extends HashSet<E>
    implements Set<E>, Cloneable, java.io.Serializable {
}
```

##### 构造函数

```java
public LinkedHashSet() {
  super(16, .75f, true);
}

HashSet(int initialCapacity, float loadFactor, boolean dummy) {
  map = new LinkedHashMap<>(initialCapacity, loadFactor);
}
```

