在类中用{}包起来，但是没有用static标识的就是普通代码块，比如：
```java
public class Test{
    {
        System.out.println("I am ordinary code block!");
    }
}
```
用static标识的代码块就是静态代码块，比如:
```java
public class Test{
    static {
        System.out.println("I am static code block!");
    }
}
```
再讲解它俩的区别时，我们先看个例子:
```java
public class Test{
    {
        System.out.println("I am ordinary code block!");
    }
    static {
        System.out.println("I am static code block!");
    }
    public static void main(String[] args) {
    }
}
```
这个例子会打印出什么东西呢？
答案是:
```data
I am static block!
```
我们修改一下main方法:
```java
    public static void main(String[] args) {
        Test s = new Test();
    }
```
再看执行结果:
```data
I am static code block!
I am ordinary code block!
```
普通代码块执行了，并且在static(静态)代码块执行之后执行，虽然static代码块在普通代码块之后，可以看出，static是在类加载时执行，并且只执行一次，而普通代码块是在类初始化时执行，当然也只是执行一次。
那么是不是static代码块都会执行呢？
并不是，这个就看类加载机制是否加载到这个类，比如我们虽然定义了一个类，我们的代码执行期间从来没有用过这个类的信息，即使我们用了类的静态变量，也不会触发static(静态代码块)的执行。
举个例子:
Base类:
```java
public class Base {
   static {
      System.out.println("super class init!");
   }
   public static final int VALUE =100;
}
```
main方法所在的类:
```java
public class Main {
    public static void main(String[] args) {
        System.out.println(Base.VALUE);
    }
}
```
打印结果:
```data
100
```
有人会问，我们的main方法不是有Base.value这段代码么，这样Base类就被加载了么？
实际上不是这样的，JVM为了执行效率高，会把静态变量直接插入代码执行代码中，而不会真正通过静态变量所在的类进行查找，这样，减少查找的过程，大大提高了代码的执行效率，感兴趣的可以看下JVM的内存模型。
总结一下:
1. 静态(static)代码块是在类加载阶段执行。
2. 普通代码块只有在类第一次被初始化时才执行。
3. 所以静态(static)代码块肯定是在普通代码块执行之前执行。
4. 某些条件静态(static)代码不一定会执行，只有当静态(static)代码块所在的类被加载时，才执行。
