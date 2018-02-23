为什么要用sleep方法，主要是为了暂停当前线程，把cpu片段让出给其他线程，减缓当前线程的执行。
方法的定义:
public static void sleep(long millis);
```java
    public static native void sleep(long millis) throws InterruptedException;
```
通过定义可以看出sleep方法是本地方法，通过系统调用暂停当前线程，而不是java自己实现的。
sleep还有一个重载的方法:
public static void sleep(long millis, int nanos)
实现如下:
```java
    public static void sleep(long millis, int nanos)
    throws InterruptedException {
        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos >= 500000 || (nanos != 0 && millis == 0)) {
            millis++;
        }

        sleep(millis);
    }
```
从表面现象上来看，这个方法支持纳秒级别的暂定，但是内部的实现最终还是毫秒级别的执行，以500 000纳秒作为分割，大于这个值时，线程在millis的基础上多sleep 1毫秒，否则还是sleep millis毫秒，当然如果millis为0时，会sleep 1毫秒。
写个简单的demo来看线程的执行:
```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
public class ThreadTest implements Runnable{
    public static void main(String[] args) throws InterruptedException {
        Thread test1 = new Thread(new ThreadTest());
        Thread test2 = new Thread(new ThreadTest());
        test1.start();
        test2.start();
        test1.sleep(5000);
    }

    @Override
    public void run() {
       for (int i = 0; i < 5; i++){
           System.out.println(i);
       }
    }
}

```
执行结果:
```data
0
1
2
3
4
0
1
2
3
4
//此处会暂停5秒
end
```
值得注意的是:
1. sleep是帮助其他线程获得运行机会的最好方法，但是如果当前线程获取到的有锁，sleep不会让出锁。
2. 线程睡眠到期自动苏醒，并返回到可运行状态（就绪），不是运行状态。
3. 优先线程的调用，现在苏醒之后，并不会里面执行，所以sleep()中指定的时间是线程不会运行的最短时间，sleep方法不能作为精确的时间控制。
4. sleep()是静态方法，只能控制当前正在运行的线程（示例就是这样调用的，因为类对象可以调用类的静态方法）。
