try语句的定义和关闭资源，这种方式我们用的比较少，也介绍一下吧。
```java
public class Main {
    public static void main(String[] args) throws Exception {
        String s = "CYF";
        try (//创建对象输出流
             ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("b.bin"));
             //创建对象输入流
             ObjectInputStream ois = new ObjectInputStream(new FileInputStream("b.bin"));
        )
        {
            //序列化java对象
            oos.writeObject(s);
            oos.flush();
        }
    }
}
```
反编译之后的结果:
```java
import java.io.*;

public class Main
{

    public Main()
    {
    }

    public static void main(String args[])
        throws Exception
    {
        String s;
        ObjectOutputStream objectoutputstream;
        Throwable throwable;
        s = "CYF";
        objectoutputstream = new ObjectOutputStream(new FileOutputStream("b.bin"));
        throwable = null;
        ObjectInputStream objectinputstream;
        Throwable throwable3;
        objectinputstream = new ObjectInputStream(new FileInputStream("b.bin"));
        throwable3 = null;
        try
        {
            objectoutputstream.writeObject(s);
            objectoutputstream.flush();
        }
        catch(Throwable throwable5)
        {
            throwable3 = throwable5;
            throw throwable5;
        }
        if(objectinputstream != null)
            if(throwable3 != null)
                try
                {
                    objectinputstream.close();
                }
                catch(Throwable throwable4)
                {
                    throwable3.addSuppressed(throwable4);
                }
            else
                objectinputstream.close();
        break MISSING_BLOCK_LABEL_139;
        Exception exception;
        exception;
        if(objectinputstream != null)
            if(throwable3 != null)
                try
                {
                    objectinputstream.close();
                }
                catch(Throwable throwable6)
                {
                    throwable3.addSuppressed(throwable6);
                }
            else
                objectinputstream.close();
        throw exception;
        if(objectoutputstream != null)
            if(throwable != null)
                try
                {
                    objectoutputstream.close();
                }
                catch(Throwable throwable1)
                {
                    throwable.addSuppressed(throwable1);
                }
            else
                objectoutputstream.close();
        break MISSING_BLOCK_LABEL_215;
        Throwable throwable2;
        throwable2;
        throwable = throwable2;
        throw throwable2;
        Exception exception1;
        exception1;
        if(objectoutputstream != null)
            if(throwable != null)
                try
                {
                    objectoutputstream.close();
                }
                catch(Throwable throwable7)
                {
                    throwable.addSuppressed(throwable7);
                }
            else
                objectoutputstream.close();
        throw exception1;
    }
}
```
反编译之后的代码很长，长的都不想阅读了，简单的来说把try和关闭资源的代码分开，try代码会生成try/catch语句，和一个异常中奖变量，用这个中间变量保存异常信息，在最后关闭资源的代码执行后再抛出这个异常中奖变量保存的异常。

