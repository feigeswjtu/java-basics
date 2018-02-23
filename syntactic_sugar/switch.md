# 枚举的switch支持
我们知道switch只支持int型以及编译器能自己转换为int的类型，在老版本中，枚举变量和下面要介绍的字符串是不支持switch语句的，但是1.5以上的却支持，它是怎么做到的呢？我们探究以下，借用上面的枚举类型的java代码，我们实现一个switch语句。
```java
public class Main {
    public static void main(String[] args) {
        Vehicle vehicle = Vehicle.CAR;
        switch (vehicle){
            case BUS:
                System.out.println("BUS");
                break;
            case CAR:
                System.out.println("CAR");
                break;
            default:
                System.out.println("default");

        }
    }
}
```
class反编译之后的结果:
```java
public class Main
{

    public Main()
    {
    }

    public static void main(String args[])
    {
        Vehicle vehicle = Vehicle.CAR;
        static class _cls1
        {

            static final int $SwitchMap$Vehicle[];

            static
            {
                $SwitchMap$Vehicle = new int[Vehicle.values().length];
                try
                {
                    $SwitchMap$Vehicle[Vehicle.BUS.ordinal()] = 1;
                }
                catch(NoSuchFieldError nosuchfielderror) { }
                try
                {
                    $SwitchMap$Vehicle[Vehicle.CAR.ordinal()] = 2;
                }
                catch(NoSuchFieldError nosuchfielderror1) { }
            }
        }

        switch(_cls1..SwitchMap.Vehicle[vehicle.ordinal()])
        {
        case 1: // '\001'
            System.out.println("BUS");
            break;

        case 2: // '\002'
            System.out.println("CAR");
            break;

        default:
            System.out.println("default");
            break;
        }
    }
}
```
通过反编译之后的代码我们就能知道，原来枚举类型的语法糖生成的代码是通过一个数组实现，会把枚举变量的原始值作为下标，存的值是1、2、3...从1到枚举个数的整数值，最终还是使用的int型做switch操作。
# 字符串的switch支持
```java
    public static void main(String[] args) {
        String str = "bbb";
        switch (str){
            case "aaa":
                System.out.println("a");
                break;
            case "bbb":
                System.out.println("b");
                break;
            case "ccc":
                System.out.println("b");
                break;
            default:
                System.out.println("default");

        }
    }
```
jad反编译之后的结果:
```java
public class Main
{

    public Main()
    {
    }

    public static void main(String args[])
    {
        String s = "b";
        String s1 = s;
        byte byte0 = -1;
        switch(s1.hashCode())
        {
        case 97: // 'a'
            if(s1.equals("a"))
                byte0 = 0;
            break;

        case 98: // 'b'
            if(s1.equals("b"))
                byte0 = 1;
            break;

        case 99: // 'c'
            if(s1.equals("c"))
                byte0 = 2;
            break;
        }
        switch(byte0)
        {
        case 0: // '\0'
            System.out.println("a");
            break;

        case 1: // '\001'
            System.out.println("b");
            break;

        case 2: // '\002'
            System.out.println("b");
            break;

        default:
            System.out.println("default");
            break;
        }
    }
}
```
很容易就看明白了字符串(String)的switch的实现逻辑，先通过字符串的hashCode值定位到要case的字符串，再通过equal()确认这个字符串，确认后给一个int型(编译器能自己转换为int的类型)中间变量赋值，这个中间变量的值是从0到case的个数n-1。最终在通过switch这个int型(编译器能自己转换为int的类型)中间变量做switch，本例的中间变量是byte型。
