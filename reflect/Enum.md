# 什么是枚举

我们知道枚举是java中特殊的一个类，通常用来定义一组确定的常量，所有的枚举变量都继承于`java.lang.Enum类`，枚举常量是类型安全的，每个枚举常量对应的值不会再运行中变化。枚举也是类，在Java中枚举是语法糖，感兴趣的可以用反编译器来看下反编译之后的枚举类到底是什么样子的。

既然枚举是类，反射机制就不需要单独为枚举类型定义一个类似`java.lang,reflect.Enum`来操作枚举类，反射API只为枚举类提供了几个特殊的方法，分别是`Class.isEnum()`、`Class.getEnumConstants()`和`java.lang.reflect.Field.isEnumConstant()`，其他方法都与操作普通的类是一样的。

# 检测枚举类型

上面我们说过，枚举类型也是普通的类，只是一个语法糖，所以反射机制单独为枚举类型提供了三个特殊的API，分别是:
* `Class.isEnum()`: 判断Class是否是一个枚举类型。
* `Class.getEnumConstants()`: 按照定义顺序列举出枚举类定义的常量。
* `java.lang.reflect.Field.isEnumConstant()`: 判断一个成员变量是否是一个枚举类型

我们知道，如果要列举一个枚举类定义的常量，会调用枚举类的方法`values()`来列举，但是如果是反射中的Class，就没有办法直接获取了，需要借助`Class.getEnumConstants()`，举个例子:

```java
import java.util.Arrays;

enum Eon { HADEAN, ARCHAEAN, PROTEROZOIC, PHANEROZOIC }

public class Main{
    public static void main(String[] args) throws ClassNotFoundException {
        Class<?> c = (args.length == 0 ? Eon.class : Class.forName(args[0]));
        //通过反射机制getEnumConstants列举枚举类定义的常量
        System.out.format("Enum name:  %s%nEnum constants:  %s%n",
                c.getName(), Arrays.asList(c.getEnumConstants()));
        //通过枚举类的方法values列举枚举类定义的常量
        System.out.format("  Eon.values():  %s%n",
                Arrays.asList(Eon.values()));
    }
}

```
输出结果: 
```java
Enum name:  Eon
Enum constants:  [HADEAN, ARCHAEAN, PROTEROZOIC, PHANEROZOIC]
  Eon.values():  [HADEAN, ARCHAEAN, PROTEROZOIC, PHANEROZOIC]
```

# Get和Set枚举类型的成员变量

和普通的成员变量一样，枚举类型的成员变量也是使用`Field.set()`和`Field.get()`类Set和Get成员变量的值。
举个例子:
```java
import java.lang.reflect.Field;

//枚举类
enum TraceLevel { OFF, LOW, MEDIUM, HIGH, DEBUG }

//包含一个枚举成员变量的类
class MyServer {
    private TraceLevel level = TraceLevel.OFF;

}
public class Main{
    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
        TraceLevel newLevel = TraceLevel.valueOf("LOW");
        MyServer server = new MyServer();
        Class<?> c = server.getClass();
        Field f = c.getDeclaredField("level");
        f.setAccessible(true);
        //Get枚举类的成员变量
        TraceLevel oldLevel = (TraceLevel)f.get(server);
        System.out.format("Original trace level:  %s%n", oldLevel);

        if (oldLevel != newLevel) {
            //Set枚举类型的成员变量
            f.set(server, newLevel);
            System.out.format("    New  trace level:  %s%n", f.get(server));
        }
    }
}

```

输出:
```out
Original trace level:  OFF
    New  trace level:  LOW
```

枚举常量是单例的，在类加载中就会生成，所以 == 和 != 操作符合用来比较枚举类型的值。

# 枚举类型在反射机制常遇到的异常

枚举类型是一个比较特殊的类，所以在反射机制中也会遇到比较特殊的异常。

## 尝试初始化枚举常量

我们知道，枚举类型定义的常量在类加载到虚拟机的时候就已经生成了，在运行期间不能再进行初始化，非反射机制的代码会编译失败，反射机制虽然不会编译失败，但是同样不能初始化，会抛出时抛出IllegalArgumentException:
```java
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;
import static java.lang.System.out;

enum Charge {
    POSITIVE, NEGATIVE, NEUTRAL;
    Charge() {
        out.format("under construction%n");
    }
}

public class Main{

    public static void main(String... args) {
        try {
            Class<?> c = Charge.class;

            //获取构造器
            Constructor[] ctors = c.getDeclaredConstructors();
            for (Constructor ctor : ctors) {
                out.format("Constructor: %s%n",  ctor.toGenericString());
                //设置构造器可访问
                ctor.setAccessible(true);
                //初始化时会抛出异常
                ctor.newInstance();
            }

            // production code should handle these exceptions more gracefully
        } catch (InstantiationException x) {
            x.printStackTrace();
        } catch (IllegalAccessException x) {
            x.printStackTrace();
        } catch (InvocationTargetException x) {
            x.printStackTrace();
        }
    }
}
```
运行结果:
```java
Exception in thread "main" Constructor: private Charge()
java.lang.IllegalArgumentException: Cannot reflectively create enum objects
  at java.lang.reflect.Constructor.newInstance(Constructor.java:417)
  at Main.main(Main.java:25)
```

所以，在我们不知道一个类是否是枚举类时，在初始化这个类之前一定要通过`Class.isEnum()`来判断这个类是否是枚举类。

## 把其他枚举类型赋值给枚举类型的成员变量
如果把一个其他的枚举类型常量赋值给一个枚举类型的成员变量，也会抛出: IllegalArgumentException。
```java
import java.lang.reflect.Field;

enum E0 { A, B }
enum E1 { A, B }

class ETest {
    private E0 fld = E0.A;
}

public class Main{
    public static void main(String... args) {
        try {
            ETest test = new ETest();
            Field f = test.getClass().getDeclaredField("fld");
            f.setAccessible(true);
            f.set(test, E1.A);  // 抛出异常
        } catch (NoSuchFieldException x) {
            x.printStackTrace();
        } catch (IllegalAccessException x) {
            x.printStackTrace();
        }
    }
}
```

运行结果:

```java
Exception in thread "main" java.lang.IllegalArgumentException: Can not set E0 field ETest.fld to E1
  at sun.reflect.UnsafeFieldAccessorImpl.throwSetIllegalArgumentException(UnsafeFieldAccessorImpl.java:167)
  at sun.reflect.UnsafeFieldAccessorImpl.throwSetIllegalArgumentException(UnsafeFieldAccessorImpl.java:171)
  at sun.reflect.UnsafeObjectFieldAccessorImpl.set(UnsafeObjectFieldAccessorImpl.java:81)
  at java.lang.reflect.Field.set(Field.java:764)
  at Main.main(Main.java:16)
```

为了避免这个异常我们可以通过`X.class.isAssignableFrom(Y.class) == true`来判断两个类是否兼容。





