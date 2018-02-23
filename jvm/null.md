# 背景
无意间看到一个题目，将null强制转换成一个类（class），然后调用类的静态方法（static function），代码如下:

```java
public class Test {
    public static void printStatic(){
        System.out.println("I am test static method");
    }
    static public void main(String []agrs){
        ((Test)null).printStatic();
    }
}
```
答案有三个答案:
1. 编译错误
2. 什么也没有输出
3. 输入I am test static method

第一感觉是编译错误，null怎么可能转换成其他对象呢。但是答案却是3。神奇的Java，分析一下为什么能打出method。

# null能强制转换为所有类的对象

为什么null能强制转换为任意类型的对象？答案我也不确定，可能有两种原因（欢迎指正）。
1. 任何类变量的默认值都是null，所以能把null转换为任意类型的对象。
2. Java的约定（任何解释不通的现象都可以先说成约定）。
这道题目这一个知识点不是重点，重点是null强制转换的对象在内存里也是null，为什么能调用类的实例方法呢。
我们使用javap -verbose CLASSNAME（读者请自行搜索它的用法）来看生成的字节码，方便我们分析问题。

```java
Classfile /home/chenyafei/workspace/java/test/Test.class
  Last modified Aug 20, 2017; size 495 bytes
  MD5 checksum 704b330ea58e0db02a92c1b130b950f2
  Compiled from "Test.java"
public class Test
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #7.#17         // java/lang/Object."<init>":()V
   #2 = Fieldref           #18.#19        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #20            // I am test static method
   #4 = Methodref          #21.#22        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Methodref          #6.#23         // Test.printStatic:()V
   #6 = Class              #24            // Test
   #7 = Class              #25            // java/lang/Object
   #8 = Utf8               <init>
   #9 = Utf8               ()V
  #10 = Utf8               Code
  #11 = Utf8               LineNumberTable
  #12 = Utf8               printStatic
  #13 = Utf8               main
  #14 = Utf8               ([Ljava/lang/String;)V
  #15 = Utf8               SourceFile
  #16 = Utf8               Test.java
  #17 = NameAndType        #8:#9          // "<init>":()V
  #18 = Class              #26            // java/lang/System
  #19 = NameAndType        #27:#28        // out:Ljava/io/PrintStream;
  #20 = Utf8               I am test static method
  #21 = Class              #29            // java/io/PrintStream
  #22 = NameAndType        #30:#31        // println:(Ljava/lang/String;)V
  #23 = NameAndType        #12:#9         // printStatic:()V
  #24 = Utf8               Test
  #25 = Utf8               java/lang/Object
  #26 = Utf8               java/lang/System
  #27 = Utf8               out
  #28 = Utf8               Ljava/io/PrintStream;
  #29 = Utf8               java/io/PrintStream
  #30 = Utf8               println
  #31 = Utf8               (Ljava/lang/String;)V
{
  public Test();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 4: 0

  public static void printStatic();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=0, args_size=0
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String I am test static method
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 6: 0
        line 7: 8

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=0, locals=1, args_size=1
         0: invokestatic  #5                  // Method printStatic:()V
         3: return
      LineNumberTable:
        line 9: 0
        line 10: 3
}
SourceFile: "Test.java"
```

main方法里的Code下是直接调用了方法（#5 指向了类方法），并没有通过实例对象调用，可以看出来，**实例化的对象调用类方法是跟实例化的对象没有直接关系的，只是通过实例找到这个对象的类，进而找到类方法。**

那有人问了，main方法也是Test的类方法，本类的类方法肯定是可以调用本类的类方法的，是不是这个原因呢。
那我们把main方法从这个类中拆出来，再试试呢。

Test:
```java
public class Test {
    public static void printStatic(){
        System.out.println("I am test static method");
    }

}
```

运行类:
TestPublic
```java
public class TestPublic{
static public void main(String []agrs){
Test.printStatic();
}
}
```
执行 javac Test.java TestPublic.java，编译生成字节码。
执行 javap -verbose TestPublic:

```java
Classfile /home/chenyafei/workspace/java/test/TestPublic.class
  Last modified Aug 20, 2017; size 306 bytes
  MD5 checksum 9d19fa001bed7835381a82fbf60b95cb
  Compiled from "TestPublic.java"
public class TestPublic
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #4.#13         // java/lang/Object."<init>":()V
   #2 = Methodref          #14.#15        // Test.printStatic:()V
   #3 = Class              #16            // TestPublic
   #4 = Class              #17            // java/lang/Object
   #5 = Utf8               <init>
   #6 = Utf8               ()V
   #7 = Utf8               Code
   #8 = Utf8               LineNumberTable
   #9 = Utf8               main
  #10 = Utf8               ([Ljava/lang/String;)V
  #11 = Utf8               SourceFile
  #12 = Utf8               TestPublic.java
  #13 = NameAndType        #5:#6          // "<init>":()V
  #14 = Class              #18            // Test
  #15 = NameAndType        #19:#6         // printStatic:()V
  #16 = Utf8               TestPublic
  #17 = Utf8               java/lang/Object
  #18 = Utf8               Test
  #19 = Utf8               printStatic
{
  public TestPublic();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=0, locals=1, args_size=1
         0: invokestatic  #2                  // Method Test.printStatic:()V
         3: return
      LineNumberTable:
        line 3: 0
        line 4: 3
}
SourceFile: "TestPublic.java"
```

结果也是一样，直接调用了类方法，并没有跟实例对象绑定在一起，证实了，我们的结论。

