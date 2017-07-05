## 数据结构

> 数组实现的List类

## 源码分析

##### 成员变量

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;

    //初始化空间大小
    private static final int DEFAULT_CAPACITY = 10;

    //空对象
    private static final Object[] EMPTY_ELEMENTDATA = {};

    //初始化对象
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    //数据存储对象
    transient Object[] elementData; // non-private to simplify nested class access

    //数组长度
    private int size;
}
```

##### 构造函数

```java
public ArrayList() {
  this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

## 重点方法

##### get

```java
 public E get(int index) {
   rangeCheck(index);
   return elementData(index);
 }
```

##### set

```java
public E set(int index, E element) {
  rangeCheck(index);
  E oldValue = elementData(index);
  elementData[index] = element;
  return oldValue;
}
```

##### add

```java
// 将element添加到ArrayList的指定位置   
public void add(int index, E element) {
  rangeCheckForAdd(index);
  ensureCapacityInternal(size + 1); 

  //从指定源数组中复制一个数组，复制从指定的位置开始，到目标数组的指定位置结束。
  //arraycopy(被复制的数组, 从第几个元素开始复制, 要复制到的数组, 从第几个元素开始粘贴, 一共需要复制的元素个数)
  //即在数组elementData从index位置开始，复制到index+1位置,共复制size-index个元素
  System.arraycopy(elementData, index, elementData, index + 1,size - index);
  elementData[index] = element;
  size++;
}
```

