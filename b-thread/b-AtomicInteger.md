## 算法介绍

> Conpare and Swap，内存值V，旧预期值A，修改新值B，当V与A相同，修改V为B，否则什么都不做。

## 源码分析

##### 类成员变量

```java
public class AtomicInteger extends Number implements java.io.Serializable
{	
	//该类是JDK提供的可以对内存直接操作的工具类
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    //该值保存着AtomicInteger基础数据的内存地址，方便unsafe直接对内存的操作
    private static final long valueOffset;
	//保存着AtomicInteger基础数据，使用volatile修饰，可以保证该值对内存可见，也是原子类实现的理论保障。
    private volatile int value;
}
```

## 重点函数分析

##### incrementAndGet

```java
public final int incrementAndGet() {
	return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}

public final int getAndAddInt(Object var1, long var2, int var4) {
  int var5;
  do {
    var5 = this.getIntVolatile(var1, var2);
  } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

  return var5;
}
```

##### Unsafe里面getIntVolatile函数

```java
//getIntVolatile方法native实现，通过volatile方法获取当前内存中该对象的value值。
jint sun::misc::Unsafe::getIntVolatile (jobject obj, jlong offset)    
{    
  //计算value的内存地址。
  volatile jint *addr = (jint *) ((char *) obj + offset);
  //将值赋值给中间变量result。
  jint result = *addr; 
  //插入读屏障，保证该屏障之前的读操作后后续的操作可见。
  read_barrier (); 
  //返回当前内存值
  return result; 
}  
inline static void read_barrier(){
  __asm__ __volatile__("" : : : "memory");
}
```

##### Unsafe里面compareAndSwap函数

```java
static inline bool compareAndSwap (volatile jlong *addr, jlong old, jlong new_val)    {    
  jboolean result = false;    
  //使用自旋锁来处理并发问题。
  spinlock lock; 
  //比较内存中的值与调用方法时调用方所期待的值。
  if ((result = (*addr == old)))
    //如果3中的比较符合预期，则重置内存中的值。
    *addr = new_val;
  //如果成功置换则返回true，否则返回false；
  return result;
} 
```

