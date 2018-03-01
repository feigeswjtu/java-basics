Java里面的类型是一个引用或者一个基本类型，类、枚举、或者数组都是继承于java.lang.Object，它们和接口一样都是引用类型，对于这些类型，JVM提供了方式可以在运行中获取对象对应类型，也就是它属于哪个Class。java.lang.class也提供了创建Class和Class对应的对象的方式。

# 获取Class
目前有很多种方式方式可以获取到一个类(Class)。

所有的反射操作的切入点都是java.lang.Class，这也印证了Java是面向对象编程语言，在包java.lang.reflect中，除了 java.lang.reflect.ReflectPermission之外都没有public的构造方法，为了得到这些类，需要在java.lang.Class上调用对应的方法。

## Object.getClass()

引用对象可以调用getClass()方法获取到它的类。

```java
Class c = "foo".getClass();
System.out.println(c);//class java.lang.String

Class c = System.out.getClass();
System.out.println(c);//class java.io.PrintStream

byte[] bytes = new byte[1024];
Class c = bytes.getClass();
System.out.println(c);//class [B

Set<String> s = new HashSet<String>();
Class c = s.getClass();
System.out.println(c);//class java.util.HashSet
```

这里要注意是的通过接口的引用调用其getClass()方法是返回引用对象的真正类，而不是接口的名称。

数组对象的Class会以“[”开头。

## .class

.class的方式是使用Class本身调用.class获取。

比如:
```java
boolean b;
Class c = b.getClass();   // compile-time error
Class c = boolean.class;  // boolean

Class c = java.io.PrintStream.class;//class java.io.PrintStream
Class c = int[][][].class;//class [[[I
```

## Class.forName()

forName()是声明异常类方法，如果找不到这个类会抛出java.lang.ClassNotFoundException异常。
比如:
```java
Class c = Class.forName("com.duke.MyLocaleServiceProvider");
Exception in thread "main" java.lang.ClassNotFoundException: com.duke.MyLocaleServiceProvider
  at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
      at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:335)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
          at java.lang.Class.forName0(Native Method)
            at java.lang.Class.forName(Class.java:264)
              at reflect.Main.main(Main.java:8)
```
如果能找到类:
```java
Class cDoubleArray = Class.forName("[D");
Class cStringArray = Class.forName("[[Ljava.lang.String;");
```

## 基础类型包装类的TYPE属性

基础类型的包装类都有一个TYPE属性，和包装类的Class是一样的。

```java
Class c = Double.TYPE;
Class c = Void.TYPE;
```
## 其他方式

### Class.getSuperclass()

获得当前类的父类。

### Class.getClasses()

返回当前类的内部定义的类、接口、枚举类型。

### Class.getDeclaredClasses()

返回当前类中显式声明的接口、枚举类型。

Class.getDeclaringClass()方法可以引申出另外几个方法:
java.lang.reflect.Field.getDeclaringClass()
java.lang.reflect.Method.getDeclaringClass()
java.lang.reflect.Constructor.getDeclaringClass()
作用和Class.getDeclaredClasses()一样，只是作用于的类型不一样，分别是Field、Method、Constructor。

## Class.getEnclosingClass()

返回当前类的封闭类，也就是包含这个类的类，比如:

```java
Class c = Thread.State.class.getEnclosingClass();
System.out.println(c);//class java.lang.Thread
```

# 获取类的描述符

我们知道了如何获得一个类的类名，有没有什么办法可以获取到类的访问权限、是否是接口、是否是static类，是否final标识、是否strictfg标识等，获取类的注解等。

答案是肯定的。

访问权限、接口标识、static标识，final标识、strictfp标识等这些都是标识Class的标识符。

可以通过Class.getModifiers()获取，然后自己做遍历判断。

Class.getAnnotations()是获取Class的注解的方式。

更多的Class方法可以参考[官方文档](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html)。

贴一个官方的例子:

```java
import java.lang.annotation.Annotation;
import java.lang.reflect.Modifier;
import java.lang.reflect.Type;
import java.lang.reflect.TypeVariable;
import java.util.Arrays;
import java.util.ArrayList;
import java.util.List;
import static java.lang.System.out;

public class ClassDeclarationSpy {
    public static void main(String... args) {
  try {
      Class<?> c = Class.forName(args[0]);
      out.format("Class:%n  %s%n%n", c.getCanonicalName());
      out.format("Modifiers:%n  %s%n%n",
           Modifier.toString(c.getModifiers()));

      out.format("Type Parameters:%n");
      TypeVariable[] tv = c.getTypeParameters();
      if (tv.length != 0) {
    out.format("  ");
    for (TypeVariable t : tv)
        out.format("%s ", t.getName());
    out.format("%n%n");
      } else {
    out.format("  -- No Type Parameters --%n%n");
      }

      out.format("Implemented Interfaces:%n");
      Type[] intfs = c.getGenericInterfaces();
      if (intfs.length != 0) {
    for (Type intf : intfs)
        out.format("  %s%n", intf.toString());
    out.format("%n");
      } else {
    out.format("  -- No Implemented Interfaces --%n%n");
      }

      out.format("Inheritance Path:%n");
      List<Class> l = new ArrayList<Class>();
      printAncestor(c, l);
      if (l.size() != 0) {
    for (Class<?> cl : l)
        out.format("  %s%n", cl.getCanonicalName());
    out.format("%n");
      } else {
    out.format("  -- No Super Classes --%n%n");
      }

      out.format("Annotations:%n");
      Annotation[] ann = c.getAnnotations();
      if (ann.length != 0) {
    for (Annotation a : ann)
        out.format("  %s%n", a.toString());
    out.format("%n");
      } else {
    out.format("  -- No Annotations --%n%n");
      }

        // production code should handle this exception more gracefully
  } catch (ClassNotFoundException x) {
      x.printStackTrace();
  }
    }

    private static void printAncestor(Class<?> c, List<Class> l) {
  Class<?> ancestor = c.getSuperclass();
   if (ancestor != null) {
      l.add(ancestor);
      printAncestor(ancestor, l);
   }
    }
}
```

执行结果:
```data
$ java ClassDeclarationSpy java.util.concurrent.ConcurrentNavigableMap
Class:
  java.util.concurrent.ConcurrentNavigableMap

Modifiers:
  public abstract interface

Type Parameters:
  K V

Implemented Interfaces:
  java.util.concurrent.ConcurrentMap<K, V>
  java.util.NavigableMap<K, V>

Inheritance Path:
  -- No Super Classes --

Annotations:
  -- No Annotations --
```

```data
$ java ClassDeclarationSpy "[Ljava.lang.String;"
Class:
  java.lang.String[]

Modifiers:
  public abstract final

Type Parameters:
  -- No Type Parameters --

Implemented Interfaces:
  interface java.lang.Cloneable
  interface java.io.Serializable

Inheritance Path:
  java.lang.Object

Annotations:
  -- No Annotations --
```
```java
$ java ClassDeclarationSpy java.io.InterruptedIOException
Class:
  java.io.InterruptedIOException

Modifiers:
  public

Type Parameters:
  -- No Type Parameters --

Implemented Interfaces:
  -- No Implemented Interfaces --

Inheritance Path:
  java.io.IOException
  java.lang.Exception
  java.lang.Throwable
  java.lang.Object

Annotations:
  -- No Annotations --
```

```data
$ java ClassDeclarationSpy java.security.Identity
Class:
  java.security.Identity

Modifiers:
  public abstract

Type Parameters:
  -- No Type Parameters --

Implemented Interfaces:
  interface java.security.Principal
  interface java.io.Serializable

Inheritance Path:
  java.lang.Object

Annotations:
  @java.lang.Deprecated()
```

# 获取类的成员

知道了怎么获取到类的标识符，比如类的访问权限、接口标识、static标识，final标识、strictfp标识等，下面我们继续跟着官方文档讲解一下怎么获取类的成员，类成员包括以下三种：成员变量、类方法（实例方法和类方法）、构造器（构造方法）。
## 获取方式
为了更好的描述，我们做个约定个通配符XXXX，如果是成员变量就代表Field，如果是类方法就代表Method，如果是构造器就代表Constructor。
那么怎么获取到这三类成员呢？
获取单个的成员的方式用: getXXXX()和getDeclaredXXXX();
列举多个成员的方式用: getXXXXs()和getDeclaredXXXXs();
## getXXXX和getDeclared的区别
那么他们有什么区别呢？
简单的来说普通的方式(不带Declared)获取类的公共（public）的成员，包括父类，带有Declared的方式获取类的所有申明的成员，即包括public、private和protected声明的成员，不包括父类的申明字段。 
那么就有人疑问那怎么获取到父类的成员呢？当然是获取到父类的Class之后，通父类的Class调用这两类方法获取。
普通的方式是比较常用的方式，反射本身就破坏封装的一种方式，为了减少这种破坏，我们应该操作public成员即可。
具体的区别如下:

### 获取成员变量
| Class的API方法 | 是否可以列举 | 是否能列举继承类的成员 |是否能列举私有成员 |
|:-------------:|:-------------:|:-----:|:----:|
| getDeclaredField() | 否 | 否 | 是 |
|getField()| 否 | 是 | 否 |
|getDeclaredFields()| 是 | 否 | 是 |
|getFields()| 是 | 否 | 否 |
### 获取成员方法
| Class的API方法 | 是否可以列举 | 是否能列举继承类的成员 |是否能列举私有成员 |
|:-------------:|:-------------:|:-----:|:----:|
| getDeclaredMethod() | 否 | 否 | 是 |
|getMethod()| 否 | 是 | 否 |
|getDeclaredMethods()| 是 | 否 | 是 |
|getMethods()| 是 | 否 | 否 |
### 获取构造器

| Class的API方法 | 是否可以列举 | 是否能列举继承类的成员 |是否能列举私有成员 |
|:-------------:|:-------------:|:-----:|:----:|
| getDeclaredConstructor() | 否 | 否 | 是 |
|getConstructor()| 否 | 是 | 否 |
|getDeclaredConstructors()| 是 | 否 | 是 |
|getConstructors()| 是 | 否 | 否 |

### 类成员的Class
getXXXX()和getDeclaredXXXX()获取到的类也就是类成员的Class，对应的Class如下.
成员变量: java.lang.reflect.Field
成员方法: java.lang.reflect.Method
构造器方法: java.lang.reflect.Constructor
后面会分为三章分别解释一下对应的用法。

类如果没有成员，只是一个类名，就像一个空架子，就无法使用，后面两章我们会介绍一个反射机制中类成员变量(Field)、类方法(method)和类的构造器(Constructor)。


