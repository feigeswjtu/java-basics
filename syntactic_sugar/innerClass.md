内部类就是定义在一个类内部的类。

有些时候我们希望一个类只在另一个类中有用，不想让这类在其他地方被使用，这个时候就要用到内部类了，内部类之所以是语法糖，是因为其只是一个编译时的概念，一旦编译完成，编译器就会为内部类生成一个单独的class文件，名为outer$innter.class，比如Mybatis生成器生成的Example文件就使用了静态内部类。
```java
public class Out {
    class Inner{
    }
}
```
我们看下编译之后的class文件会发现生成了两个class文件，一个是Out.class，一个是Out$Inner.class，内容如下:
```java
public class Out {
}
```
```java
class Out$Inner {

   // $FF: synthetic field
   final Out this$0;


   Out$Inner(Out var1) {
      this.this$0 = var1;
   }
}
```

通过注释可以看出是编译器生成的成员，感兴趣的可以看下我的关于[反射的系列文章](../reflect.md)。
内部类有四种：成员内部类、局部内部类、匿名内部类、静态内部类，每一种都有各自的用法，这里就不介绍了。

