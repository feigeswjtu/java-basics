# 反射中数组

数组是一个特殊类型，数组有两个属性，一个是数组中元素的类型，一个是数组的长度。数组可以由元素一个一个的组装，也可以整个一起组装成。反射机制提供了一系列的方法来操作数组，这些方法在`java.lang.reflect.Array`中。

和前面一样本文也是按照以下几点进行开展:

- 介绍数组的描述符是如何确定一个类的实例是数组的类型。
- 介绍如何通过反射创建数组。
- 介绍如果操作数组，比如获取一个数组的原因或者给数组的某一个下标元素重复赋值。
- 介绍反射操作数组的异常和编程时的注意事项。

# 确定数组类型

我们知道，数组也是一个类。
可以通过`Class.isArray()`方法类判断一个类是否是数组类型，通过Class.getComponentType()获取数组元素的类型。
举个例子:
```java
import java.lang.reflect.Field;

import static java.lang.System.out;

public class Main{
    public static void main(String... args) {
        boolean found = false;
        try {
            //输入一个类名
            Class<?> cls = Class.forName(args[0]);
            //获取类型的成员变量
            Field[] flds = cls.getDeclaredFields();
            for (Field f : flds) {
                //获取成员变量的类型
                Class<?> c = f.getType();
                //如果成员变量的类型是个数组
                if (c.isArray()) {
                    found = true;
                    //打印出成员变量的信息
                    out.format("%s%n"
                                    + "           Field: %s%n"
                                    + "            Type: %s%n"
                                    + "  Component Type: %s%n",
                            f, f.getName(), c, c.getComponentType());
                }
            }
            if (!found) {
                out.format("No array fields%n");
            }

            // production code should handle this exception more gracefully
        } catch (ClassNotFoundException x) {
            x.printStackTrace();
        }
    }
}
```
比如类ByteBuffer有一个数组类成员变量hb，打印出来。

```java
$java ArrayFind java.nio.ByteBuffer
    final byte[] java.nio.ByteBuffer.hb
    Field: hb
    Type: class [B
    Component Type: byte
```

# 通过反射创建一个数组

创建一个数组用`Array.newInstance(c, n)`方法，其中c是数组元素的Class，n是数组的长度。

举个例子:
```java
public class Main{
    public static void main(String... args) {
        //数组的类型将为java.math.BigInteger
        String cName = "java.math.BigInteger";
        //数组的值
        String[] cVals = {"123", "456", "789"};
        try {
            //加载Class
            Class<?> aClass = Class.forName(cName);
            //初始化一个数组对象
            Object o = Array.newInstance(aClass, cVals.length);
            // 获取java.math.BigInteger的构造器
            Constructor constructor = aClass.getConstructor(String.class);
            //循环把构造的java.math.BigInteger放入数组中
            for (int i = 0; i < cVals.length; i++) {
                String val = cVals[i];
                Object n = constructor.newInstance(val);
                Array.set(o, i, n);
            }
            //把Object转换为Object[]
            Object[] oo = (Object[])o;
            //打印数组的内容
            out.format("%s[] = %s%n", cName, Arrays.toString(oo));
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
    }
}
```
初始化数组，也就是给数组赋值，`Array.set(o, i, val)`，其中o是数组对应的Object索引，i是赋值的下标(从0开始)，val是赋值数组元素的索引，后面我们会讲它的更多用法。

# Get和Set数组的以及它的元素
和非反射代码一样，一个数组类型的成员变量可以一个一个的Set或者Get，也可以整体Set和Get。整体的操作中，Get用`java.lang.reflect.Field.set(Object obj, Object value)`方法，Set用` Field.get(Object)`;单个的操作中Set和Get都是使用`java.lang.reflect.Array`的静态方法。

`java.lang.reflect.Array.` 提供类型setFoo() and getFoo()方法，用来Set和Get基本类型，例如 一个int型数组可以使用`Array.setInt(Object array, int index, int value)`来Set数组的元素，使用`Array.getInt(Object array, int index)`来获取数组的元素。

另外，`Array.setInt(Object array, int index, int value)`和`Array.getInt(Object array, int index)`可以自动扩展数组元素类型，比如，`Array.setShort()`获取的值可以赋值int型数组的元素，因为16字节的short型和无损的扩展为32字节的int型数据，然后，在一个int型数组上调用Array.setLong() 会抛出IllegalArgumentException异常，因为64字节的 long 不能被无损的存储在32字节的数组里。

引用型数组(包括数组的数组)， 可以用`Array.set(Object array, int index, int value)`和`Array.get(Object array, int index)`来Set和Get数组的元素。

## 如何Set一个数组类型的成员变量
下面的例子将展示如何替换一个数组类型的成员变量:
```java
public class Main{
    private static final int srcBufSize = 10 * 1024;
    private static char[] src = new char[srcBufSize];
    static {
        src[srcBufSize - 1] = 'x';
    }
    //初始化一个10*1024长度的CharArrayReader
    private static CharArrayReader car = new CharArrayReader(src);

    public static void main(String... args) {
        try {
            // 初始化BufferedReader
            BufferedReader br = new BufferedReader(car);

            //获取BufferedReader的Class，以便于使用反射机制
            Class<?> c = br.getClass();
            //获取BufferedReader的cb成员变量
            Field f = c.getDeclaredField("cb");

            // cb是一个私有变量，把它设置为可访问
            f.setAccessible(true);
            //获取br的cb成员变量的引用，并强制转换为char[]
            char[] cbVal = char[].class.cast(f.get(br));

            //copy出一个新的char数组
            char[] newVal = Arrays.copyOf(cbVal, cbVal.length * 2);
            //如果输入的参数是grow，则重新赋值
            if (args.length > 0 && args[0].equals("grow"))
                f.set(br, newVal);

            for (int i = 0; i < srcBufSize; i++)
                br.read();

            // 如果新的数组被设置为了br成员变量，打印出新数组的长度
            if (newVal[srcBufSize - 1] == src[srcBufSize - 1])
                out.format("Using new backing array, size=%d%n", newVal.length);
            else
                out.format("Using original backing array, size=%d%n", cbVal.length);

            // production code should handle these exceptions more gracefully
        } catch (FileNotFoundException x) {
            x.printStackTrace();
        } catch (NoSuchFieldException x) {
            x.printStackTrace();
        } catch (IllegalAccessException x) {
            x.printStackTrace();
        } catch (IOException x) {
            x.printStackTrace();
        }
    }
}
```
## 如何Get或者Set一个多维数组

反射框架提供了初始化多维数组的方式`Array.newInstance(Class<?> componentType, int... dimensions)`，但是并没有提供可以直接操作多维数组的Get和Set方法，必须自己降维后再通过Get和Get一维数组的方式处理。

```java
public class Main{
    public static void main(String... args) {
        //直接创建多维数组
        Object matrix = Array.newInstance(int.class, 2, 2);
        //降维处理
        Object row0 = Array.get(matrix, 0);
        Object row1 = Array.get(matrix, 1);

        //Set一维数组
        Array.setInt(row0, 0, 1);
        Array.setInt(row0, 1, 2);
        Array.setInt(row1, 0, 3);
        Array.setInt(row1, 1, 4);

        for (int i = 0; i < 2; i++)
            for (int j = 0; j < 2; j++)
                out.format("matrix[%d][%d] = %d%n", i, j, ((int[][])matrix)[i][j]);
    }
}
```

# 反射机制操作数组时常遇到的异常

## 反射机制不会自动装箱和拆箱

下面的例子将会抛出IllegalArgumentException异常，根本原因是装箱和拆箱是编译器做的，对反射机制来说，运行中才会确定数据的类型，并不会自动装箱和拆箱。
那么有什么办法可以确定在反射机制中，哪儿写是否可以转换的类型呢?
通过`Class.isAssignableFrom(Class)`，比如:

```java
Integer.class.isAssignableFrom(int.class) == false
```

```java
int.class.isAssignableFrom(Integer.class) == false
```

```java
public class Main{
    public static void main(String... args) {
        Integer[] ary = new Integer[2];
        Array.setInt(ary, 0, 1);  // IllegalArgumentException
    }
}
```

## 数组越界

写过代码的技术人员都清楚，数组最经常遇到的一个错误就是越界操作，会抛出ArrayIndexOutOfBoundsException异常，反射同样如此。
```java
public class Main{
    public static void main(String... args) {
        Object o = Array.newInstance(int.class, 0);
        Array.getInt(o, 0);  // ArrayIndexOutOfBoundsException
    }
}
```
上面的这段代码会抛出ArrayIndexOutOfBoundsException异常。

## 缩小调用

前面也说过，如果把一个long型的数值赋值给一个int型的数组，会抛出异常的，这就是缩小调用，但是如果把short型的数字赋值给int型数组，就不会有异常。

```java

public class Main{
    public static void main(String... args) {
        Object o = new int[2];
        Array.setShort(o, 0, (short)2);  // 扩宽操作，成功
        Array.setLong(o, 1, 2L);         // 缩小操作，失败，抛出异常
    }
}
```





