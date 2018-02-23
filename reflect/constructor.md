我们知道每个类都至少有一个构造器，因为一个类如果没有显示定义一个构造器，编译器自动会自动生成一个默认无参的构造器，构造器作为一个类的入口方法，在使用类的成员变量和方法之前，类的构造器必须被调用，生成一个实例，另外构造器不能被继承，如果子类的构造器没有显示的调用父类的构造器，执行器会默认的调用父类的构造器。

说了这么多，我们该进入正题了，本文讲解一个反射中怎么获取一个构造器的声明、参数等信息，怎么调用一个构造器。

# 获取构造器的方法

和[获取类的方法](https://feige.gitbooks.io/java-basics/content/reflect/method.html)一样，在反射的包里，获取类的构造器也是有两个方法:
1. Class.getDeclaredConstructors(): 获取非public的构造器。
2. Class.getConstructors(): 获取public的构造器。
这两个方法的返回值都是java.lang.reflect.Constructor<?>[]，可以看出获取类构造器的声明、参数、异常信息则是通过java.lang.reflect.Constructor实现的。

写个例子来简单的介绍一下java.lang.reflect.Constructor的实现:
```java
public class ConstructorSift {
    public static void main(String... args) {
  try {
      Class<?> cArg = Class.forName(args[1]);

      Class<?> c = Class.forName(args[0]);
      Constructor[] allConstructors = c.getDeclaredConstructors();
      for (Constructor ctor : allConstructors) {
    Class<?>[] pType  = ctor.getParameterTypes();
    for (int i = 0; i < pType.length; i++) {
        if (pType[i].equals(cArg)) {
      out.format("%s%n", ctor.toGenericString());

      Type[] gpType = ctor.getGenericParameterTypes();
      for (int j = 0; j < gpType.length; j++) {
          char ch = (pType[j].equals(cArg) ? '*' : ' ');
          out.format("%7c%s[%d]: %s%n", ch,
               "GenericParameterType", j, gpType[j]);
      }
      break;
        }
    }
      }

        // production code should handle this exception more gracefully
  } catch (ClassNotFoundException x) {
      x.printStackTrace();
  }
    }
}
```
这个例子的功能是查找第一个输入参数的类具有第二个输入参数的构造方法的信息。
执行几个例子: 
## 查找普通类作为参数的类构造器
查找有Locale参数的java.util.Formatter的构造方法的信息:
```data
$ java ConstructorSift java.util.Formatter java.util.Locale
public
java.util.Formatter(java.io.OutputStream,java.lang.String,java.util.Locale)
throws java.io.UnsupportedEncodingException
       GenericParameterType[0]: class java.io.OutputStream
       GenericParameterType[1]: class java.lang.String
      *GenericParameterType[2]: class java.util.Locale
public java.util.Formatter(java.lang.String,java.lang.String,java.util.Locale)
throws java.io.FileNotFoundException,java.io.UnsupportedEncodingException
       GenericParameterType[0]: class java.lang.String
       GenericParameterType[1]: class java.lang.String
      *GenericParameterType[2]: class java.util.Locale
public java.util.Formatter(java.lang.Appendable,java.util.Locale)
       GenericParameterType[0]: interface java.lang.Appendable
      *GenericParameterType[1]: class java.util.Locale
public java.util.Formatter(java.util.Locale)
      *GenericParameterType[0]: class java.util.Locale
public java.util.Formatter(java.io.File,java.lang.String,java.util.Locale)
throws java.io.FileNotFoundException,java.io.UnsupportedEncodingException
       GenericParameterType[0]: class java.io.File
       GenericParameterType[1]: class java.lang.String
      *GenericParameterType[2]: class java.util.Locale
```
## 查找数组类作为参数的构造器
查找String的char[]构造方法。
```java
$ java ConstructorSift java.lang.String "[C"
java.lang.String(int,int,char[])
       GenericParameterType[0]: int
       GenericParameterType[1]: int
      *GenericParameterType[2]: class [C
public java.lang.String(char[],int,int)
      *GenericParameterType[0]: class [C
       GenericParameterType[1]: int
       GenericParameterType[2]: int
public java.lang.String(char[])
      *GenericParameterType[0]: class [C
```
三、查找变长参数的构造器
 ProcessBuilder 有一个构造器: `public ProcessBuilder(String... command)`
```data
java ConstructorSift java.lang.ProcessBuilder "[Ljava.lang.String;"
public java.lang.ProcessBuilder(java.lang.String[])
      *GenericParameterType[0]: class [Ljava.lang.String;
```
变成参数是一个语法糖，编译成字节码之后参数就是一个数组而已，关于语法糖的一些知识看可以一下我的另外一篇[文章](http://blog.csdn.net/feigeswjtu/article/details/79000588)。
四、查找泛型参数的构造器
```data
java ConstructorSift java.util.HashMap java.util.Map
public java.util.HashMap(java.util.Map<? extends K, ? extends V>)
      *GenericParameterType[0]: java.util.Map<? extends K, ? extends V>
```

# 获取构造器的标识符
构造器和类的其他方法不一样，构造器只有以下几种标识符:
1. 访问权限描述符: public, protected, and private。
2. 注解。
```java
public class ConstructorAccess {
    public static void main(String... args) {
  try {
      Class<?> c = Class.forName(args[0]);
      Constructor[] allConstructors = c.getDeclaredConstructors();
      for (Constructor ctor : allConstructors) {
    int searchMod = modifierFromString(args[1]);
    int mods = accessModifiers(ctor.getModifiers());
    if (searchMod == mods) {
        out.format("%s%n", ctor.toGenericString());
        out.format("  [ synthetic=%-5b var_args=%-5b ]%n",
             ctor.isSynthetic(), ctor.isVarArgs());
    }
      }

        // production code should handle this exception more gracefully
  } catch (ClassNotFoundException x) {
      x.printStackTrace();
  }
    }

    private static int accessModifiers(int m) {
  return m & (Modifier.PUBLIC | Modifier.PRIVATE | Modifier.PROTECTED);
    }

    private static int modifierFromString(String s) {
  if ("public".equals(s))               return Modifier.PUBLIC;
  else if ("protected".equals(s))       return Modifier.PROTECTED;
  else if ("private".equals(s))         return Modifier.PRIVATE;
  else if ("package-private".equals(s)) return 0;
  else return -1;
    }
}
```
这个例子是获取第一个参数名的类具有第二参数类型的构造器。
比如获取File类有private访问权限的构造器:
```java
$ java ConstructorAccess java.io.File private
private java.io.File(java.lang.String,int)
  [ synthetic=false var_args=false ]
private java.io.File(java.lang.String,java.io.File)
  [ synthetic=false var_args=false ]
```
和获取类的注解以下，获取构造器的注解也是通过Constructor.getAnnotations获取。

# 反射调用类构造器
我们知道Class上有一个反射实例化一个类的方法: Class.newInstance()，现在又有了另外一种方式： java.lang.reflect.Constructor.newInstance()，两者的调用是不同的:
1. Class.newInstance() 只能调用无参构造器，有参构造器只能通过java.lang.reflect.Constructor.newInstance()来调用。
2. Class.newInstance() 会抛出很多种异常，java.lang.reflect.Constructor.newInstance()只会抛出InvocationTargetException。
3. Class.newInstance() 只能调用当前调用者可见的构造器，java.lang.reflect.Constructor.newInstance()可以调用private等当前调用者不可见的构造器。
```java
class EmailAliases {
    private Set<String> aliases;
    private EmailAliases(HashMap<String, String> h) {
  aliases = h.keySet();
    }

    public void printKeys() {
  out.format("Mail keys:%n");
  for (String k : aliases)
      out.format("  %s%n", k);
    }
}

public class RestoreAliases {

    private static Map<String, String> defaultAliases = new HashMap<String, String>();
    static {
  defaultAliases.put("Duke", "duke@i-love-java");
  defaultAliases.put("Fang", "fang@evil-jealous-twin");
    }

    public static void main(String... args) {
  try {
      Constructor ctor = EmailAliases.class.getDeclaredConstructor(HashMap.class);
      ctor.setAccessible(true);
      EmailAliases email = (EmailAliases)ctor.newInstance(defaultAliases);
      email.printKeys();

        // production code should handle these exceptions more gracefully
  } catch (InstantiationException x) {
      x.printStackTrace();
  } catch (IllegalAccessException x) {
      x.printStackTrace();
  } catch (InvocationTargetException x) {
      x.printStackTrace();
  } catch (NoSuchMethodException x) {
      x.printStackTrace();
  }
    }
}
```
本例通过反射调用一个Hash参数的类构造器。
```java
$ java RestoreAliases
Mail keys:
  Duke
  Fang
```
# 构造器经常遇到的异常
## 无默认构造器异常
上面我们讲了Class.newInstance()只能调用无参构造器，如果没有这个构造器就会抛出InstantiationException。
```java
public class ConstructorTrouble {
    private ConstructorTrouble(int i) {}

    public static void main(String... args){
  try {
      Class<?> c = Class.forName("ConstructorTrouble");
      Object o = c.newInstance();  // InstantiationException

        // production code should handle these exceptions more gracefully
  } catch (ClassNotFoundException x) {
      x.printStackTrace();
  } catch (InstantiationException x) {
      x.printStackTrace();
  } catch (IllegalAccessException x) {
      x.printStackTrace();
  }
    }
}
```
测试结果:
```java
$ java ConstructorTrouble
java.lang.InstantiationException: ConstructorTrouble
        at java.lang.Class.newInstance0(Class.java:340)
        at java.lang.Class.newInstance(Class.java:308)
        at ConstructorTrouble.main(ConstructorTrouble.java:7)

```
## 构造器抛出异常
如果调用构造器时，构造器本身抛出异常，则我们的反射调用也会抛出java.lang.RuntimeException。
```java
public class ConstructorTroubleToo {
    public ConstructorTroubleToo() {
   throw new RuntimeException("exception in constructor");
    }

    public static void main(String... args) {
  try {
      Class<?> c = Class.forName("ConstructorTroubleToo");
      // Method propagetes any exception thrown by the constructor
      // (including checked exceptions).
      if (args.length > 0 && args[0].equals("class")) {
    Object o = c.newInstance();
      } else {
    Object o = c.getConstructor().newInstance();
      }

        // production code should handle these exceptions more gracefully
  } catch (ClassNotFoundException x) {
      x.printStackTrace();
  } catch (InstantiationException x) {
      x.printStackTrace();
  } catch (IllegalAccessException x) {
      x.printStackTrace();
  } catch (NoSuchMethodException x) {
      x.printStackTrace();
  } catch (InvocationTargetException x) {
      x.printStackTrace();
      err.format("%n%nCaught exception: %s%n", x.getCause());
  }
    }
}
```
测试结果:
```java
$ java ConstructorTroubleToo class
Exception in thread "main" java.lang.RuntimeException: exception in constructor
        at ConstructorTroubleToo.<init>(ConstructorTroubleToo.java:6)
        at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
        at sun.reflect.NativeConstructorAccessorImpl.newInstance
          (NativeConstructorAccessorImpl.java:39)
        at sun.reflect.DelegatingConstructorAccessorImpl.newInstance
          (DelegatingConstructorAccessorImpl.java:27)
        at java.lang.reflect.Constructor.newInstance(Constructor.java:513)
        at java.lang.Class.newInstance0(Class.java:355)
        at java.lang.Class.newInstance(Class.java:308)
        at ConstructorTroubleToo.main(ConstructorTroubleToo.java:15)
```

## 查找或者调用错误的构造器
查找或者调用错误的构造器会抛出NoSuchMethodException和IllegalArgumentException，这里就不举例子的，读者可以自己实现这个例子。

## 调用不可访问的构造器
构造器存在，但是对当前访问者不可见时会抛出: IllegalAccessException异常。
```java
class Deny {
    private Deny() {
  System.out.format("Deny constructor%n");
    }
}

public class ConstructorTroubleAccess {
    public static void main(String... args) {
  try {
      Constructor c = Deny.class.getDeclaredConstructor();
//        c.setAccessible(true);   // solution
      c.newInstance();

        // production code should handle these exceptions more gracefully
  } catch (InvocationTargetException x) {
      x.printStackTrace();
  } catch (NoSuchMethodException x) {
      x.printStackTrace();
  } catch (InstantiationException x) {
      x.printStackTrace();
  } catch (IllegalAccessException x) {
      x.printStackTrace();
  }
    }
}
```
测试结果:
```java
java.lang.IllegalAccessException: Class ConstructorTroubleAccess can not access
  a member of class Deny with modifiers "private"
        at sun.reflect.Reflection.ensureMemberAccess(Reflection.java:65)
        at java.lang.reflect.Constructor.newInstance(Constructor.java:505)
        at ConstructorTroubleAccess.main(ConstructorTroubleAccess.java:15)

```
