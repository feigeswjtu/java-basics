声明一个简单的断言:
```java
    public static void main(String[] args) {
        assert false : "执行错误";
    }
```
运行时加上参数-ea，执行结果:
```data
Exception in thread "main" java.lang.AssertionError: 执行错误
        at Main.main(Main.java:6)
```
那么断言是怎么实现的呢？通过jad看下反编译后的结果:
```java
public class Main
{

    public Main()
    {
    }

    public static void main(String args[])
    {
        if(!$assertionsDisabled)
            throw new AssertionError("\u6267\u884C\u9519\u8BEF");
        else
            return;
    }

    static final boolean $assertionsDisabled = !Main.desiredAssertionStatus();

}
```
最终还是语法糖，有异常时抛出AssertionError。
