我们知道，Java为每一种基础类型都提供了对应的包装类型，比如int型的包装类型为Integer，java5之后，我们可以这样做，`Integer a = 1`, 可以把基础类型直接赋值给一个包装类型，这个就是语法糖，我们举个例子。

编译之前的源码:
```java
    public static void main(String[] args) {
        Integer i = 10;
        i = i + 5;
        System.out.println(i);
    }
```
经过反编译之后的代码:
```java
   public static void main(String[] var0) {
      Integer var1 = Integer.valueOf(10);
      var1 = Integer.valueOf(var1.intValue() + 5);
      System.out.println(var1);
   }
```

可以看出，包装类型只要涉及到计算的操作，都要经过拆箱`Integer.intValue()`和装箱操作`Integer.valueOf()`，所以，如果数字经常变化时，最好使用非包装类型，减少拆箱和装箱操作，提高效率。

另外要说下自动装箱和拆箱是有一定的坑的，使用的时候要注意，举个例子:

```java
    public static void main(String[] args) {
        Integer a = 1;
        Integer b = 2;
        Integer c = 3;
        Integer d = 3;
        Integer e = 321;
        Integer f = 321;
        Long g = 3L;
        System.out.println(c == d);
        System.out.println(e == f);
        System.out.println(c == a + b);
        System.out.println(c.equals(a + b));
        System.out.println(g == a + b);
        System.out.println(g.equals(a + b));
    }
```
读者可以先猜测一下答案，然后再上机进行验证一些下。
结果是不是有些出乎意料。
我们反编译一下这段代码，看看最终拆箱装箱之后的结果:
```java
   public static void main(String[] var0) {
      Integer var1 = Integer.valueOf(1);//1
      Integer var2 = Integer.valueOf(2);//2
      Integer var3 = Integer.valueOf(3);//3
      Integer var4 = Integer.valueOf(3);//4
      Integer var5 = Integer.valueOf(321);//5
      Integer var6 = Integer.valueOf(321);//6
      Long var7 = Long.valueOf(3L);
      System.out.println(var3 == var4);//a
      System.out.println(var5 == var6);//b
      System.out.println(var3.intValue() == var1.intValue() + var2.intValue());//c
      System.out.println(var3.equals(Integer.valueOf(var1.intValue() + var2.intValue())));//d
      System.out.println(var7.longValue() == (long)(var1.intValue() + var2.intValue()));//e
      System.out.println(var7.equals(Integer.valueOf(var1.intValue() + var2.intValue())));//f
   }
```
我们先把结果贴出来，一个一个的分析:
```data
true
false
true
true
true
false
```
结果b为false很容易理解，==是对引用的比较，var5 == var6不一样很正常的，为什么a会是true呢?答案就是在装箱的方法valueOf()中，我们看下源码:
```java
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```
原来和String一样，Integer也有一个cache，如果装箱结果命中了这个cache，直接从缓存池中拿一个对象，而默认情况下，这个缓存池的大小是-128到127，当然这个缓存池的大小可以进行调整，具体请看源码，这里就不具体介绍。
通过c的结果我们可以知道在数字的==前后如果有数字的计算，会进行拆箱操作，使用原始值进行比较，所以c为true。
d就不用说了，类型一样，值一样，肯定为true。
e和c类似，Long和Integer无法进行==，编译器就会做拆箱操作，进行数学层面的比较。
f是不同类型的equals的比较，肯定为true，这个也是equals的设计原则，想了解深的可以看下我的另外[一篇文章](../hashCodeAndEquals.md)。

好了，装箱和拆箱操作就先讲到这，原则是尽量避免使用自动装箱和拆箱。
