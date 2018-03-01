常量折叠是Java在编译期做的一个优化，简单的来说，在编译期就把一些表达式计算好，不需要在运行时进行计算。
比如: `int a = 1 + 2`，经过常量折叠后就变成了`int a = 3`。
我们举个例子:
```java
public class Main {
    public static void main(String[] args) {
        String s1 = "a" + "bc";
        String s2 = "ab" + "c";
        System.out.println(s1 == s2);
    }
}
```
执行结果为true。
我们使用javac编译之后，在通过反编译工具（这个有个[网站](http://javare.cn/)可以用）看下编译器优化后的代码:
```java
public class Main {
   public static void main(String[] var0) {
      String var1 = "abc";
      String var2 = "abc";
      System.out.println(var1 == var2);
   }
}
```
var1和var2的值都是常量池中的"abc"，是同一个引用，所以会相等。
修改下我们的例子:
```java
public class Main {
    public static void main(String[] args) {
        String a = "a";
        String b = "b";
        String s1= a + b;
        String s2= a + b;
        System.out.println(s1 == s2);
    }
}
```
执行结果为: false。
我们反编译生成的class文件:
```java
public class Main {

   public static void main(String[] var0) {
      String var1 = "a";
      String var2 = "b";
      String var3 = var1 + var2;
      String var4 = var1 + var2;
      System.out.println(var3 == var4);
   }
}
```
我们知道，对于字符串进行 a + b的代码，运行中是这样处理的:
`String s2 = (new StringBuffer()).append(a).append(b).toString(); `
所以最终得到的s1和s2是不相等(==)的。
第一个例子就是“常量折叠”，并不是所有的常量都会进行折叠，必须是编译期常量之间进行运算才会进行常量折叠，编译器常量就是编译时就能确定其值的常量，这个定义很严格，需要满足以下条件:
1. 字面量是编译期常量（数字字面量，字符串字面量等）。
2. 编译期常量进行简单运算的结果也是编译期常量，如1+2，"a"+"b"。
3. 被编译器常量赋值的 final 的基本类型和字符串变量也是编译期常量。

最后我们举一个final标识的常量折叠的例子:
```java
public class Main {
    static final String a = "a";
    static final String b = "b";
    public static void main(String[] args) {
        String s1= a + b;
        String s2= a + b;
        System.out.println(s1 == s2);
    }
}
```
输出结果为: `true`
反编译的结果如下:
```java

public class Main {

   static final String a = "a";
   static final String b = "b";
   public static void main(String[] var0) {
      String var1 = "ab";
      String var2 = "ab";
      System.out.println(var1 == var2);
   }
}
```
