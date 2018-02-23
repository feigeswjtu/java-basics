Java基础的程序员都知道，==和equals的区别就是前者比较引用值后者比较对象的内容。

那么以下的代码输出的结果是true还是false呢？
```java
class TestValue{
    int i;
}
public class Main {
    public static void main(String[] args) {
        TestValue t1 = new TestValue();
        TestValue t2 = new TestValue();
        t1.i = 10;
        t2.i = 10;
        System.out.println(t1.equals(t2));
    }

}
```
答案是false。
我们看看TestValue父类是什么？
```java
System.out.println(t1.getClass().getSuperclass());
```

输出的结果是: `class java.lang.Object`

通过查看源码可以看到Object的equals默认是比较引用的，如果不重写这个方法，是无法得到我们要的值。

```java
    public boolean equals(Object obj) {
        return (this == obj);
    }
```
下面我们重写equals方法：
```java
class TestValue{
    int i;
    public boolean equals(TestValue t){
        if(this.i == t.i){
            return true;
        }else{
            return false;
        }
    }
}
public class Main {
    public static void main(String[] args) {
        TestValue t1 = new TestValue();
        TestValue t2 = new TestValue();
        t1.i = 10;
        t2.i = 10;
        System.out.println(t1.equals(t2));
    }

}  
```

输出的结果为true。



