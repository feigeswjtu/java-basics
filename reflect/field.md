前面一章讲了怎么通过Class获取到成员（成员变量、成员方法、构造器），本篇文章开始详细讲解成员变量（java.lang.reflect.Field）的详细用法。
# 获取field的类型
有两种方式可以获取到field的属性，Field.getType()和Field.getGenericType()，其中getGenericType可以获取到泛型的标识符，如果这个field是泛型，则返回泛型的标识，如果不是泛型，这会转而调用getType获取到真正的类型，也就是Object。
这里可以提一下，Java里的泛型是假泛型，从字节码到可以执行文件的时候，已经把泛型擦除了，变成真正的类型，但是getType()调用时，并没有真正的类型代入，所以会返回所有的类的父类Object。
我们举个例子:
```java
public class FieldSpy<T> {
    public boolean[][] b = {{ false, false }, { true, true } };
    public String name  = "Alice";
    public List<Integer> list;
    public T val;

    public static void main(String[] args) {
        try {
            Class<?> c = Class.forName(args[0]);
            Field f = c.getField(args[1]);
            System.out.format("Type: %s%n", f.getType());
            System.out.format("GenericType: %s%n", f.getGenericType());
        } catch (ClassNotFoundException x) {
            x.printStackTrace();
        } catch (NoSuchFieldException x) {
            x.printStackTrace();
        }
    }
}
```
执行命令以及执行结果:
```data
$ java FieldSpy FieldSpy b
Type: class [[Z
GenericType: class [[Z
$ java FieldSpy FieldSpy name
Type: class java.lang.String
GenericType: class java.lang.String
$ java FieldSpy FieldSpy list
Type: interface java.util.List
GenericType: java.util.List<java.lang.Integer>
$ java FieldSpy FieldSpy val
Type: class java.lang.Object
GenericType: T
```
# 检索和解析Field的修饰符
Field的修饰符可以通过public int getModifiers()方法获取，这个方法返回的是int型，代表意义可以参见[修饰符](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Modifier.html)，如果要判断一个field是否具有某个修饰符，可以通过位运算符&判断，比如判断一个field的修饰符是否有public属性:
```java
Field f = OneClass.getField("field");
int modify = f.getModifiers();
return modify&Modifier.PUBLIC == Modifier.PUBLIC
```
可以看一个官方的例子:
```java
enum Spy { BLACK , WHITE }

public class FieldModifierSpy {
    volatile int share;
    int instance;
    class Inner {}

    public static void main(String... args) {
  try {
      Class<?> c = Class.forName(args[0]);
      int searchMods = 0x0;
      for (int i = 1; i < args.length; i++) {
    searchMods |= modifierFromString(args[i]);
      }

      Field[] flds = c.getDeclaredFields();
      out.format("Fields in Class '%s' containing modifiers:  %s%n",
           c.getName(),
           Modifier.toString(searchMods));
      boolean found = false;
      for (Field f : flds) {
    int foundMods = f.getModifiers();
    // Require all of the requested modifiers to be present
    if ((foundMods & searchMods) == searchMods) {
        out.format("%-8s [ synthetic=%-5b enum_constant=%-5b ]%n",
             f.getName(), f.isSynthetic(),
             f.isEnumConstant());
        found = true;
    }
      }

      if (!found) {
    out.format("No matching fields%n");
      }

        // production code should handle this exception more gracefully
  } catch (ClassNotFoundException x) {
      x.printStackTrace();
  }
    }

    private static int modifierFromString(String s) {
  int m = 0x0;
  if ("public".equals(s))           m |= Modifier.PUBLIC;
  else if ("protected".equals(s))   m |= Modifier.PROTECTED;
  else if ("private".equals(s))     m |= Modifier.PRIVATE;
  else if ("static".equals(s))      m |= Modifier.STATIC;
  else if ("final".equals(s))       m |= Modifier.FINAL;
  else if ("transient".equals(s))   m |= Modifier.TRANSIENT;
  else if ("volatile".equals(s))    m |= Modifier.VOLATILE;
  return m;
    }
}
```
这个例子的大致意思是查找输入类名是否具有输入的修饰符的成员变量，并把成员变量名，并且输出其是否是编译器生成的和是否输入枚举变量。
输入输出:
```data
$ java FieldModifierSpy FieldModifierSpy volatile
Fields in Class 'FieldModifierSpy' containing modifiers:  volatile
share    [ synthetic=false enum_constant=false ]

$ java FieldModifierSpy Spy public
Fields in Class 'Spy' containing modifiers:  public
BLACK    [ synthetic=false enum_constant=true  ]
WHITE    [ synthetic=false enum_constant=true  ]

$ java FieldModifierSpy FieldModifierSpy\$Inner final
Fields in Class 'FieldModifierSpy$Inner' containing modifiers:  final
this$0   [ synthetic=true  enum_constant=false ]

$ java FieldModifierSpy Spy private static final
Fields in Class 'Spy' containing modifiers:  private static final
$VALUES  [ synthetic=true  enum_constant=false ]
```
是否是编译器生成可以通过方法 field.isSynthetic()判断。
是否是枚举变量可以通过方法field.isEnumConstant()判断。
是否是编译器我想很多人都明白，什么是编译器生成的成员变量呢？
比如枚举类型，每个枚举类型都有一个VALUES成员变量，这个变量我们并没有显式定义，但是可以通过它获取这个枚举类对应的所有没有常量，VALUES就是编译器生成的。
# 检索Field的注解
获取所有的注解可以用field.getDeclaredAnnotations()方式。
获取单个的可以用:
  getAnnotatedType()
  getAnnotation(Class<T> annotationClass)
  getAnnotationsByType(Class<T> annotationClass)
实际上这几个方法都是从 class java.lang.reflect.AccessibleObject继承而来的，这里就不做详细介绍了。
# 设置和获取Field的值
set(Object obj, Object value)来设置Field，除了这个方式还有多种确定Field类型的方式，比如void  setDouble(Object obj, double d)
Object  get(Object obj)来获取Field的值，和set方法一直，get方法也有多种确定Field类型的方式，比如double  getDouble(Object obj)。
以上方法都可能抛出NoSuchFieldException和IllegalAccessException异常。
官方文档上有一句话是这样说的: 因为这种访问通常违反了该类的设计意图，因此应尽可能谨慎的使用它。
前面就讲过，反射是破坏封装性的，违反的类的设计原则，所以能少用就少用。
这里要提一下setXXXX()内部如果是基础类型时要小心，这个方法不会进行装箱和拆箱操作，因为装箱和拆箱操作是编译器做的，运行时，JVM并不能做这个事情。
比如下面的例子就会抛出异常。
```java
public class FieldTrouble {
    public Integer val;

    public static void main(String... args) {
        FieldTrouble ft = new FieldTrouble();
        try {
            Class<?> c = ft.getClass();
            Field f = c.getDeclaredField("val");
            f.setInt(ft, 42);               // IllegalArgumentException
        } catch (NoSuchFieldException x) {
            x.printStackTrace();
        } catch (IllegalAccessException x) {
            x.printStackTrace();
        }
    }
}
```
执行结果:
```data
Exception in thread "main" java.lang.IllegalArgumentException: Can not set java.lang.Integer field reflect.FieldTrouble.val to (int)42
  at sun.reflect.UnsafeFieldAccessorImpl.throwSetIllegalArgumentException(UnsafeFieldAccessorImpl.java:167)
  at sun.reflect.UnsafeFieldAccessorImpl.throwSetIllegalArgumentException(UnsafeFieldAccessorImpl.java:191)
  at sun.reflect.UnsafeObjectFieldAccessorImpl.setInt(UnsafeObjectFieldAccessorImpl.java:114)
  at java.lang.reflect.Field.setInt(Field.java:949)
  at reflect.FieldTrouble.main(FieldTrouble.java:14)
```
做set方式之前可以通过isAssignableFrom方法来进行检测，检测之后再进行处理:
```java
Integer.class.isAssignableFrom(int.class) == false;
int.class.isAssignableFrom(Integer.class) == false
```
另外，final 标识的成员变量是不能用set方法重新设置其值的，会抛出IllegalAccessException异常。
