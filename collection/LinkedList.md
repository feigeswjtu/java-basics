# LinkedList和ArrayList的区别

[上一篇](./ArrayList.md)我们剖析了ArrayList的源码，LinkedList和ArrayList有什么区别呢？
顾名思义，LinkedList是基于链表存储元素的，而ArrayList是基于数组存储的，所以LinkedList不支持高效的随机访问，每次访问都得从头遍历直到找到这个元素。
但是LinkedList也有很多优点的，尾部的添加、删除不需要遍历，时间复杂度为O(1)，因此LinkedList整体读写效率不如ArrayList。但是链表结构对内存要求低，不需要大块连续内存来满足扩容，能够自动动态地消耗内存，容量变小时会自动释放曾经占用的内存，ArrayList则需要手动trim。

![LinkedList](../assets/images/collection.png)

看下LinkedList类声明。
```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    //...
}
```

LinkedList除了实现了List接口外还实现了Deque接口，那么LinkedList可以作为队列使用，事实上却是如此，后面我们再详细介绍。

# LinkedList的成员
```java
    //列表大小
    transient int size = 0;

    /**
     * 第一个节点的引用
     */
    transient Node<E> first;

    /**
     * 最后一个节点的引用
     */
    transient Node<E> last;

```

LinkedList既有一个头节点也有一个尾节点，如果要从尾节点向前遍历的话，Node这个类肯定是双向结点，也就是说LinkedList是以一个双向链表来存储元素的。

```java
    private static class Node<E> {
        E item;
        Node<E> next;//指向下一个元素的引用
        Node<E> prev;//指向前一个元素的引用

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

# 核心方法

## linkFirst

linkLast方法是在链表的头部插入一个元素，这个元素作为头部，用于add和push等方法，源码如下:
```java
    private void linkFirst(E e) {
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;
        size++;
        modCount++;
    }
```

## linkLast
linkLast方法是在链表的尾部链接一个元素，这个元素为尾部，用于addFirst方法，源码如下:
```java
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```

## linkBefore
在已知的结点前插入一个元素
```java
    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }
```

## unlinkFirst
删除头结点，用户删除结点或者出队操作。
```java
    /**
     * Unlinks non-null first node f.
     */
    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC
        first = next;
        if (next == null)
            last = null;
        else
            next.prev = null;
        size--;
        modCount++;
        return element;
    }
```
## 删除尾节点
删除尾结点，用户删除结点或者出队操作。
```java
    /**
     * Unlinks non-null last node l.
     */
    private E unlinkLast(Node<E> l) {
        // assert l == last && l != null;
        final E element = l.item;
        final Node<E> prev = l.prev;
        l.item = null;
        l.prev = null; // help GC
        last = prev;
        if (prev == null)
            first = null;
        else
            prev.next = null;
        size--;
        modCount++;
        return element;
    }
```

# 结尾
LinkedList本身是很简单的，只要理解双向链表的原理，实现它并不是什么难事。
