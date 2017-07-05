## 数据结构

> 双向链表

## 源码分析

##### 成员变量

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    transient int size = 0;

    //头指针
    transient Node<E> first;

    //下一个节点
    transient Node<E> last;
}
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
      this.item = element;
      this.next = next;
      this.prev = prev;
    }
}
```

##### 构造函数

```java
public LinkedList() {
}
```

