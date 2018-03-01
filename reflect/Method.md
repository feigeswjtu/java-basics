前一篇文章讲了Class中的成员变量（`java.lang.reflect.Field`）的常用使用方式以及其注意事项。我们接着讲Class中的成员方法(`java.lang.reflect.Method`)。

# 介绍

方法就是一段可执行的代码，可以是被继承而来的，也可以进行重载和重写，或者被编译器强制隐藏。但是相反的，反射代码是使方法选择被限制在一个特定的类中，而不考虑它的父类，虽然我们有办法查找到它的父类，但是这不是方法的反射能做到的，所以这里很容易引起问题。

# 获取Method的声明

方法的声明包括方法名称、描述符、参数、返回类型和异常表。类`java.lang.reflect.Method` 提供可以获取这些信息的方式。
1. 获取方法的名称
`String getName()`
2. 获取方法的描述符
`int getModifiers()` 返回值可以参见上一篇文章的介绍。
3. 返回方法的返回值类型
`Class<?> getReturnType()`
`Type getGenericReturnType`
4. 返回方法的参数（列表）
`Class<?>[]  getParameterTypes()`
`Type[] getGenericParameterTypes()`
5. 返回方法的异常信息
`Class<?>[] getExceptionTypes()`
`Type[] getGenericExceptionTypes()`

为什么获取方法的参数、返回值类型和异常表会有两个方法呢？因为带有Generic是返回声明的类型，即使这个声明的类型是泛型，也会返回泛型标识符，而不会返回真正的类型，上一章讲过的泛型擦除有介绍这点。

口说无凭，举个例子吧。
```java
public class MethodSpy {
    private static final String  fmt = "%24s: %s%n";

    // for the morbidly curious
    <E extends RuntimeException> void genericThrow() throws E {}

    public static void main(String... args) {
  try {
      Class<?> c = Class.forName(args[0]);
      Method[] allMethods = c.getDeclaredMethods();
      for (Method m : allMethods) {
    if (!m.getName().equals(args[1])) {
        continue;
    }
    out.format("%s%n", m.toGenericString());

    out.format(fmt, "ReturnType", m.getReturnType());
    out.format(fmt, "GenericReturnType", m.getGenericReturnType());

    Class<?>[] pType  = m.getParameterTypes();
    Type[] gpType = m.getGenericParameterTypes();
    for (int i = 0; i < pType.length; i++) {
        out.format(fmt,"ParameterType", pType[i]);
        out.format(fmt,"GenericParameterType", gpType[i]);
    }

    Class<?>[] xType  = m.getExceptionTypes();
    Type[] gxType = m.getGenericExceptionTypes();
    for (int i = 0; i < xType.length; i++) {
        out.format(fmt,"ExceptionType", xType[i]);
        out.format(fmt,"GenericExceptionType", gxType[i]);
    }
      }
  } catch (ClassNotFoundException x) {
      x.printStackTrace();
  }
    }
}
```

这个例子的大致意思是输入一个类，以及要获取方法信息的方法名。
输入结果:

1. java.lang.Class的getConstructor方法

```data
$ java MethodSpy java.lang.Class getConstructor
public java.lang.reflect.Constructor<T> java.lang.Class.getConstructor
  (java.lang.Class<?>[]) throws java.lang.NoSuchMethodException,
  java.lang.SecurityException
              ReturnType: class java.lang.reflect.Constructor
       GenericReturnType: java.lang.reflect.Constructor<T>
           ParameterType: class [Ljava.lang.Class;
    GenericParameterType: java.lang.Class<?>[]
           ExceptionType: class java.lang.NoSuchMethodException
    GenericExceptionType: class java.lang.NoSuchMethodException
           ExceptionType: class java.lang.SecurityException
    GenericExceptionType: class java.lang.SecurityException
```

2. java.lang.Class的cast方法

```data
$ java MethodSpy java.lang.Class cast
public T java.lang.Class.cast(java.lang.Object)
              ReturnType: class java.lang.Object
       GenericReturnType: T
           ParameterType: class java.lang.Object
    GenericParameterType: class java.lang.Object
```

cast方法的返回值是就是泛型，标识符是"T"，所以getGenericReturnType()方法会返回T，而getReturnType()则返回java.lang.Object，也就是泛型擦除之后的类型。

3. java.io.PrintStream的format方法

```data
$ java MethodSpy java.io.PrintStream format
public java.io.PrintStream java.io.PrintStream.format
  (java.util.Locale,java.lang.String,java.lang.Object[])
              ReturnType: class java.io.PrintStream
       GenericReturnType: class java.io.PrintStream
           ParameterType: class java.util.Locale
    GenericParameterType: class java.util.Locale
           ParameterType: class java.lang.String
    GenericParameterType: class java.lang.String
           ParameterType: class [Ljava.lang.Object;
    GenericParameterType: class [Ljava.lang.Object;
public java.io.PrintStream java.io.PrintStream.format
  (java.lang.String,java.lang.Object[])
              ReturnType: class java.io.PrintStream
       GenericReturnType: class java.io.PrintStream
           ParameterType: class java.lang.String
    GenericParameterType: class java.lang.String
           ParameterType: class [Ljava.lang.Object;
    GenericParameterType: class [Ljava.lang.Object;
```

# 获取参数的信息

我们单独拿出一章讲解获取参数的信息是因为参数比较特殊，为了安全和内存考虑，class的字节码文件里并不会存储参数的名称，比如一些参数名，如secret或password，可能会公开有关安全敏感方法的信息，比如很长的参数，存储其参数名会引起内存暴增。

当然我们执行时加上-parameters参数，可以强制存储其参数名称，默认情况下是不会存储的。

官方有一个打印参数的demo代码:

```java
public class MethodParameterSpy {

    private static final String  fmt = "%24s: %s%n";

    // for the morbidly curious
    <E extends RuntimeException> void genericThrow() throws E {}

    public static void printClassConstructors(Class c) {
        Constructor[] allConstructors = c.getConstructors();
        out.format(fmt, "Number of constructors", allConstructors.length);
        for (Constructor currentConstructor : allConstructors) {
            printConstructor(currentConstructor);
        }
        Constructor[] allDeclConst = c.getDeclaredConstructors();
        out.format(fmt, "Number of declared constructors",
            allDeclConst.length);
        for (Constructor currentDeclConst : allDeclConst) {
            printConstructor(currentDeclConst);
        }
    }

    public static void printClassMethods(Class c) {
       Method[] allMethods = c.getDeclaredMethods();
        out.format(fmt, "Number of methods", allMethods.length);
        for (Method m : allMethods) {
            printMethod(m);
        }
    }

    public static void printConstructor(Constructor c) {
        out.format("%s%n", c.toGenericString());
        Parameter[] params = c.getParameters();
        out.format(fmt, "Number of parameters", params.length);
        for (int i = 0; i < params.length; i++) {
            printParameter(params[i]);
        }
    }

    public static void printMethod(Method m) {
        out.format("%s%n", m.toGenericString());
        out.format(fmt, "Return type", m.getReturnType());
        out.format(fmt, "Generic return type", m.getGenericReturnType());

        Parameter[] params = m.getParameters();
        for (int i = 0; i < params.length; i++) {
            printParameter(params[i]);
        }
    }

    public static void printParameter(Parameter p) {
        out.format(fmt, "Parameter class", p.getType());
        out.format(fmt, "Parameter name", p.getName());
        out.format(fmt, "Modifiers", p.getModifiers());
        out.format(fmt, "Is implicit?", p.isImplicit());
        out.format(fmt, "Is name present?", p.isNamePresent());
        out.format(fmt, "Is synthetic?", p.isSynthetic());
    }

    public static void main(String... args) {

        try {
            printClassConstructors(Class.forName(args[0]));
            printClassMethods(Class.forName(args[0]));
        } catch (ClassNotFoundException x) {
            x.printStackTrace();
        }
    }
}
```

获取到参数是Parameter，和Field一样，同样有这几种方法:

* getType()：参数类型
* getName(): 参数名称，如果编译器加了参数-parameters，则会返回真正的参数名，如果没有加，则参数会是argN的形式，N是第几个参数。
* getModifiers(): 参数标识符，不详细介绍了。
* isImplicit()：如果是隐式声明，则返回true。比如内部类会隐式声明parent成员变量和构造器。

我们的代码:

```java
public class MethodParameterExamples {
    public class InnerClass { }
}
```

编译器真正生成的代码:

```java
public class MethodParameterExamples {
    public class InnerClass {
        final MethodParameterExamples parent;
        InnerClass(final MethodParameterExamples this$0) {
            parent = this$0;
        }
    }
}
```

* isNamePresent(): 名称是否可以同样跟-parameters有关。
* isSynthetic(): 是否是编译器生成的。

比如用上面的代码获取下面类的方法信息:

```java
public class ExampleMethods<T> {

    public boolean simpleMethod(String stringParam, int intParam) {
        System.out.println("String: " + stringParam + ", integer: " + intParam);
      return true;
    }

    public int varArgsMethod(String... manyStrings) {
        return manyStrings.length;
    }

    public boolean methodWithList(List<String> listParam) {
        return listParam.isEmpty();
    }

    public <T> void genericMethod(T[] a, Collection<T> c) {
        System.out.println("Length of array: " + a.length);
        System.out.println("Size of collection: " + c.size());
    }

}
```

编译器加-parameters的执行结果如下:

```data
Number of constructors: 1

Constructor #1
public ExampleMethods()

Number of declared constructors: 1

Declared constructor #1
public ExampleMethods()

Number of methods: 4

Method #1
public boolean ExampleMethods.simpleMethod(java.lang.String,int)
             Return type: boolean
     Generic return type: boolean
         Parameter class: class java.lang.String
          Parameter name: stringParam
               Modifiers: 0
            Is implicit?: false
        Is name present?: true
           Is synthetic?: false
         Parameter class: int
          Parameter name: intParam
               Modifiers: 0
            Is implicit?: false
        Is name present?: true
           Is synthetic?: false

Method #2
public int ExampleMethods.varArgsMethod(java.lang.String...)
             Return type: int
     Generic return type: int
         Parameter class: class [Ljava.lang.String;
          Parameter name: manyStrings
               Modifiers: 0
            Is implicit?: false
        Is name present?: true
           Is synthetic?: false

Method #3
public boolean ExampleMethods.methodWithList(java.util.List<java.lang.String>)
             Return type: boolean
     Generic return type: boolean
         Parameter class: interface java.util.List
          Parameter name: listParam
               Modifiers: 0
            Is implicit?: false
        Is name present?: true
           Is synthetic?: false

Method #4
public <T> void ExampleMethods.genericMethod(T[],java.util.Collection<T>)
             Return type: void
     Generic return type: void
         Parameter class: class [Ljava.lang.Object;
          Parameter name: a
               Modifiers: 0
            Is implicit?: false
        Is name present?: true
           Is synthetic?: false
         Parameter class: interface java.util.Collection
          Parameter name: c
               Modifiers: 0
            Is implicit?: false
        Is name present?: true
           Is synthetic?: false
```

不加参数的执行结果:

```data
  Number of constructors: 1
public reflect.ExampleMethods()
    Number of parameters: 0
Number of declared constructors: 1
public reflect.ExampleMethods()
    Number of parameters: 0
       Number of methods: 4
public boolean reflect.ExampleMethods.simpleMethod(java.lang.String,int)
             Return type: boolean
     Generic return type: boolean
         Parameter class: class java.lang.String
          Parameter name: arg0
               Modifiers: 0
            Is implicit?: false
        Is name present?: false
           Is synthetic?: false
         Parameter class: int
          Parameter name: arg1
               Modifiers: 0
            Is implicit?: false
        Is name present?: false
           Is synthetic?: false
public int reflect.ExampleMethods.varArgsMethod(java.lang.String...)
             Return type: int
     Generic return type: int
         Parameter class: class [Ljava.lang.String;
          Parameter name: arg0
               Modifiers: 0
            Is implicit?: false
        Is name present?: false
           Is synthetic?: false
public boolean reflect.ExampleMethods.methodWithList(java.util.List<java.lang.String>)
             Return type: boolean
     Generic return type: boolean
         Parameter class: interface java.util.List
          Parameter name: arg0
               Modifiers: 0
            Is implicit?: false
        Is name present?: false
           Is synthetic?: false
public <T> void reflect.ExampleMethods.genericMethod(T[],java.util.Collection<T>)
             Return type: void
     Generic return type: void
         Parameter class: class [Ljava.lang.Object;
          Parameter name: arg0
               Modifiers: 0
            Is implicit?: false
        Is name present?: false
           Is synthetic?: false
         Parameter class: interface java.util.Collection
          Parameter name: arg1
               Modifiers: 0
            Is implicit?: false
        Is name present?: false
           Is synthetic?: false
```

# 获取方法的标识符

`int getModifiers`获取标识符，这里举个例子就好，不做详细介绍。

```java
public class MethodModifierSpy {

    private static int count;
    private static synchronized void inc() { count++; }
    private static synchronized int cnt() { return count; }

    public static void main(String... args) {
  try {
      Class<?> c = Class.forName(args[0]);
      Method[] allMethods = c.getDeclaredMethods();
      for (Method m : allMethods) {
    if (!m.getName().equals(args[1])) {
        continue;
    }
    out.format("%s%n", m.toGenericString());
    out.format("  Modifiers:  %s%n",
         Modifier.toString(m.getModifiers()));
    out.format("  [ synthetic=%-5b var_args=%-5b bridge=%-5b ]%n",
         m.isSynthetic(), m.isVarArgs(), m.isBridge());
    inc();
      }
      out.format("%d matching overload%s found%n", cnt(),
           (cnt() == 1 ? "" : "s"));
  } catch (ClassNotFoundException x) {
      x.printStackTrace();
  }
    }
}
```

输出结果:

```data
$ java MethodModifierSpy java.lang.Object wait
public final void java.lang.Object.wait() throws java.lang.InterruptedException
  Modifiers:  public final
  [ synthetic=false var_args=false bridge=false ]
public final void java.lang.Object.wait(long,int)
  throws java.lang.InterruptedException
  Modifiers:  public final
  [ synthetic=false var_args=false bridge=false ]
public final native void java.lang.Object.wait(long)
  throws java.lang.InterruptedException
  Modifiers:  public final native
  [ synthetic=false var_args=false bridge=false ]
3 matching overloads found

$ java MethodModifierSpy java.lang.StrictMath toRadians
public static double java.lang.StrictMath.toRadians(double)
  Modifiers:  public static strictfp
  [ synthetic=false var_args=false bridge=false ]
1 matching overload found

$ java MethodModifierSpy MethodModifierSpy inc
private synchronized void MethodModifierSpy.inc()
  Modifiers: private synchronized
  [ synthetic=false var_args=false bridge=false ]
1 matching overload found

$ java MethodModifierSpy java.lang.Class getConstructor
public java.lang.reflect.Constructor<T> java.lang.Class.getConstructor
  (java.lang.Class<T>[]) throws java.lang.NoSuchMethodException,
  java.lang.SecurityException
  Modifiers: public transient
  [ synthetic=false var_args=true bridge=false ]
1 matching overload found

$ java MethodModifierSpy java.lang.String compareTo
public int java.lang.String.compareTo(java.lang.String)
  Modifiers: public
  [ synthetic=false var_args=false bridge=false ]
public int java.lang.String.compareTo(java.lang.Object)
  Modifiers: public volatile
  [ synthetic=true  var_args=false bridge=true  ]
2 matching overloads found
```

# 执行方法

使用反射执行方法（Invoking Methods）是很简单的事情，调用:
`public Object invoke(Object obj,
                     Object... args)
              throws IllegalAccessException,
                     IllegalArgumentException,
                     InvocationTargetException`
即可，obj是拥有方法的Class的实例，args是方法的参数。
注意方法调用可能抛出 IllegalAccessException、IllegalArgumentException、InvocationTargetException异常。

# 反射之Method的一些注意事项

1. 查找方法时c.getMethod(mName, cArg) 注意会抛出异常。
2. 私有方法调用时会抛出IllegalAccessException，但是这个可以通过AccessibleObject.setAccessible()设置成功后可以调用成功。
3. 方法调用时抛出IllegalArgumentException，抛出这个异常的原因是参数不合法。
4. 方法调用时抛出InvocationTargetException，这个异常比较特殊，方法调用成功了，但是方法内部抛出了异常，所有invoke()会抛出这个异常，可以通过Throwable cause = IllegalArgumentException.getCause()和cause.getMessage获取到异常信息。
