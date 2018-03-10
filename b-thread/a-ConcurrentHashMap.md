## 数据结构

> 分段锁的设计，只有在同一个分段内才存在竞争关系，不同分段锁之间没有锁竞争。分段锁称为Segment，类似HashMap，拥有一个数组，数组中每个元素是链表。能容16个写线程进入（写线程才需要锁定，读线程几乎不受控制）。

## 源码分析

##### 类成员变量

```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable 
{	
	//最大容量
	private static final int MAXIMUM_CAPACITY = 1 << 30;
	//默认的初始容量是16
    private static final int DEFAULT_CAPACITY = 16;
	//
    static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
	//
    private static final int DEFAULT_CONCURRENCY_LEVEL = 16;
	//默认的填充因子
    private static final float LOAD_FACTOR = 0.75f;
	//当桶(bucket)上的结点数大于这个值时会转成红黑树
    static final int TREEIFY_THRESHOLD = 8;
	// 当桶(bucket)上的结点数小于这个值时树转链表
    static final int UNTREEIFY_THRESHOLD = 6;
	// 桶中结构转化为红黑树对应的table的最小大小
    static final int MIN_TREEIFY_CAPACITY = 64;
	//扩容时最小的迁移分组大小
    private static final int MIN_TRANSFER_STRIDE = 16;

    private static int RESIZE_STAMP_BITS = 16;
	//	
    private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;
	//
    private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;
	// 转发节点（ForwardingNode）hash
    static final int MOVED     = -1; 
    // 红黑树（TreeBin）hash
    static final int TREEBIN   = -2; 
    static final int RESERVED  = -3; 
    static final int HASH_BITS = 0x7fffffff; 
	// 处理器数量
    static final int NCPU = Runtime.getRuntime().availableProcessors();
    
    
    // 哈希数组（链地址法）
    transient volatile Node<K,V>[] table; 
    // 当前扩容时的table（临时）
    private transient volatile Node<K,V>[] nextTable; 
    // 节点基础计数（见addCount方法）
	private transient volatile long baseCount; 
	// 扩容阈值，类似于HashMap中的threshold
	private transient volatile int sizeCtl; 
	// 线程迁移bin的起始位置，CAS(transferIndex)成功者可迁移transferIndex前置stride个bin（见transfer）
	private transient volatile int transferIndex; 
	// counterCells锁（见fullAddCount方法）
	private transient volatile int cellsBusy;
    // baseCount增量（见fullAddCount方法）
	private transient volatile CounterCell[] counterCells; 
}
```

##### 构造函数

> 构造的时候临界值存入大于initialCapacity的最小的二次幂数值，在第一次put的时候初始化table，这会把初始值和临界值修改回来。

```java
public HashMap(int initialCapacity, float loadFactor) {
  // 初始容量不能小于0，否则报错
  if (initialCapacity < 0)
  throw new IllegalArgumentException("Illegal initial capacity: " +
  initialCapacity);
  // 初始容量不能大于最大值，否则为最大值
  if (initialCapacity > MAXIMUM_CAPACITY)
  initialCapacity = MAXIMUM_CAPACITY;
  // 填充因子不能小于或等于0，不能为非数字
  if (loadFactor <= 0 || Float.isNaN(loadFactor))
  throw new IllegalArgumentException("Illegal load factor: " +
  loadFactor);
  // 初始化填充因子                                        
  this.loadFactor = loadFactor;
  // 初始化threshold大小
  this.threshold = tableSizeFor(initialCapacity);    
}
```

## 重点函数分析

##### get

```java
public V get(Object key) {
  Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
  int h = spread(key.hashCode());
  // 当前bin不为空，e头结点
  if ((tab = table) != null && (n = tab.length) > 0 &&
      (e = tabAt(tab, (n - 1) & h)) != null) {
    if ((eh = e.hash) == h) {
      // 头结点为待查找节点
      if ((ek = e.key) == key || (ek != null && key.equals(ek)))
        return e.val;
    }
    else if (eh < 0)
      return (p = e.find(h, key)) != null ? p.val : null;
    //// 遍历Node链表
    while ((e = e.next) != null) {
      if (e.hash == h &&
          ((ek = e.key) == key || (ek != null && key.equals(ek))))
        return e.val;
    }
  }
  return null;
}
```
