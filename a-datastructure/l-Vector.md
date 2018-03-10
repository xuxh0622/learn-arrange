## 数据结构

> 数组

## 源码分析

##### 成员变量

```java
public class Vector<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
	//Object[]类型的数组
    protected Object[] elementData;
	//动态数组实际大小
    protected int elementCount;
	//容量增长相关增长系数
    protected int capacityIncrement;
    
    private static final long serialVersionUID = -2767605614048989439L;
}
```

##### 构造函数

```java
public Vector(int initialCapacity, int capacityIncrement) {
  super();
  if (initialCapacity < 0)
    throw new IllegalArgumentException("Illegal Capacity: "+
                                       initialCapacity);
  this.elementData = new Object[initialCapacity];
  this.capacityIncrement = capacityIncrement;
}
```

## 重点方法

##### add

```java
public synchronized boolean add(E e) {
  modCount++;
  ensureCapacityHelper(elementCount + 1);
  elementData[elementCount++] = e;
  return true;
}
```

