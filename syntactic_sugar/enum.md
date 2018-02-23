枚举类型可以看做是一组类型一样的常量，当一个变量有几种可能的取值时，可以将它定义为枚举类型。

枚举类型在编译之后会生成一个类，最终还是变为类。

```java
public enum Vehicle{
    CAR, BUS;
}
```
这个语法糖通过我们上面提到的网址已经做不到反编译了，使用[jad](https://varaneckas.com/jad/) 来进行反编译：
```java
// Decompiled by Jad v1.5.8g. Copyright 2001 Pavel Kouznetsov.
// Jad home page: http://www.kpdus.com/jad.html
// Decompiler options: packimports(3)+
// Source File Name:   Vehicle.java


public final class Vehicle extends Enum
{

    public static Vehicle[] values()
    {
        return (Vehicle[])$VALUES.clone();
    }

    public static Vehicle valueOf(String s)
    {
        return (Vehicle)Enum.valueOf(Vehicle, s);
    }

    private Vehicle(String s, int i)
    {
        super(s, i);
    }

    public static final Vehicle CAR;
    public static final Vehicle BUS;
    private static final Vehicle $VALUES[];

    static
    {
        CAR = new Vehicle("CAR", 0);
        BUS = new Vehicle("BUS", 1);
        $VALUES = (new Vehicle[] {
            CAR, BUS
        });
    }
}
```
可以看出在Java的字节码结构中，其实并没有枚举类型，枚举类型只是一个语法糖，在编译完成后被编译成一个普通的类。这个类继承了java.lang.Enum，并被final关键字修饰，是不可被继承的类。
