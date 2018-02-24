在JDK的源码里有一个RandomAccess接口，这个接口没有任何方法需要实现，那么它是干什么用的呢？

```java
public interface RandomAccess {
}
```

[官方文档](https://docs.oracle.com/javase/8/docs/api/java/util/RandomAccess.html)解释如下:

接口RandomAccess被List实现用来指示它们支持快速的（通常是恒定的）随机访问。此接口的主要目的是允许通用算法改变其行为，以便在应用于随机或顺序访问列表时提供良好的性能。
用于处理随机访问列表（如ArrayList）的最佳算法可应用于顺序访问列表（如LinkedList）时产生二次行为。鼓励通用列表算法检查给定列表是否是此接口的一个实例，然后应用算法，如果将其应用于顺序访问列表将提供较差的性能，并在必要时更改其行为以保证可接受的性能。

认识到随机访问和顺序访问之间的区别通常是模糊的。例如，一些List实现在渐近线性访问时间的情况下获得巨大的访问时间，但在实践中访问时间不变。这样的List实现通常应该实现这个接口。作为一个经验法则，如果对于典型的类实例，List实现应该实现这个接口。

根本原因是:
```java
 for（int i = 0，n = list.size（）; i <n; i ++）
   list.get（ⅰ）;
``` 
运行速度比这个循环更快：
```java
for（Iterator i = list.iterator（）; i.hasNext（）;）
    i.next（）;
```

我自己的理解是这样的。

容器一般情况下有两种实现方式，一种是数组，一种是链表，对于数组来说，随机访问（下标访问）是最快的，但是对于链表的实现形式来说，随机访问（下标访问）是很慢的，因为需要从头结点逐步访问到对应的下标结点。

所以为了提高访问效率，通过这个接口来区分是否当前类允许随机访问。

我们看下最常用的两个类ArrayList和LinkedList的类声明。
ArrayList:
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{...}
```
LinkedList:
```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
    {...}
```

ArrayList实现的其中一个接口就是RandomAccess，但是LinkedList没有，所以ArrayList的随机访问效率要高，实际上也是这样的，因为ArrayList是以数组存储元素的。

Collection框架中的工具类Collections的binarySearch()方法就是借助这个特性提高性能的。
```java
public class Collections {
  ...
    public static <T>
    int binarySearch(List<? extends Comparable<? super T>> list, T key) {
        if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
            return Collections.indexedBinarySearch(list, key);
        else
            return Collections.iteratorBinarySearch(list, key);
    }
    ...
}
```
接着我们看下indexedBinarySearch和iteratorBinarySearch方法的源码:
indexedBinarySearch
```java
    private static <T>
    int indexedBinarySearch(List<? extends Comparable<? super T>> list, T key) {
        int low = 0;
        int high = list.size()-1;

        while (low <= high) {
            int mid = (low + high) >>> 1;
            //直接根据下标获取元素值
            Comparable<? super T> midVal = list.get(mid);
            int cmp = midVal.compareTo(key);

            if (cmp < 0)
                low = mid + 1;
            else if (cmp > 0)
                high = mid - 1;
            else
                return mid; // key found
        }
        return -(low + 1);  // key not found
    }
```
iteratorBinarySearch
```java
    private static <T>
    int iteratorBinarySearch(List<? extends Comparable<? super T>> list, T key)
    {
        int low = 0;
        int high = list.size()-1;
        ListIterator<? extends Comparable<? super T>> i = list.listIterator();

        while (low <= high) {
            int mid = (low + high) >>> 1;
            //根据迭代器查找元素
            Comparable<? super T> midVal = get(i, mid);
            int cmp = midVal.compareTo(key);

            if (cmp < 0)
                low = mid + 1;
            else if (cmp > 0)
                high = mid - 1;
            else
                return mid; // key found
        }
        return -(low + 1);  // key not found
    }
```
indexedBinarySearch方法是以下标的形式访问元素，而iteratorBinarySearch是以迭代器的形式访问，符合我们之前的结论。
