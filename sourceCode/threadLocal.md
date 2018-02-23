## 为什么要写这篇文章
网上很多文章都在讲ThreadLocal的意义所在，然后大部分都在说ThreadLocal是为了解决线程安全而生的，旨在解决并发安全问题，这种说法是片面的，导致很多人理解不到ThreadLocal真正用途
## ThreadLocal是什么
ThreadLocal翻译过来是线程局部变量，而不是本地线程。

ThreadLocal是为了解决在一个线程中，某个或者某些资源在不同层次的代码中通过参数来回传递的问题，但是要在没有增加性能损耗的情况下，保证线程安全。
## 为什么要用ThreadLocal
### 单资源单线程安全
比如我们有两个模块，一个把资源（静态变量）增加100，一个把资源减少100，但是我们不想把资源通过参数传入，只能把这个资源设置为静态变量。
单线程实例代码:
```java
import java.util.concurrent.atomic.AtomicInteger;

public class IncThread implements Runnable{
    private static  int VALUE = 0;
    @Override
    public void run() {
        iscr100();//模块1, 增加100
        descr100();//模块2, 减少100
    }

    public void descr100(){
        for(int i = 0; i< 1000; i++){
            VALUE--;
            try {
            //模拟IO，让CPC进行切换
            Thread.sleep(1);
            } catch (InterruptedException e) {
            }
        }

    }

    public void iscr100(){
        for(int i = 0; i< 1000; i++){
            VALUE++;
            try {
            //模拟IO，让CPC进行切换
            Thread.sleep(1);
            } catch (InterruptedException e) {
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        IncThread myInc = new IncThread();
        Thread thread = new Thread(myInc);
        thread.start();
        //保证主线程退出时，其他线程也执行完了
        Thread.sleep(10000);
        //打印最终结果
        System.out.println(VALUE);
    }
}
```
单线程下没有问题，上面的示例VALUE的最终结果肯定是0。
### 单资源多线程不安全
但是如果是多线程下会正确么？ 我们修改main方法，改用两个线程执行:
```java
    public static void main(String[] args) throws InterruptedException {
        IncThread myInc1 = new IncThread();
        Thread thread1 = new Thread(myInc1);
        Thread thread2 = new Thread(myInc1);
        thread1.start();
        thread2.start();
        //保证主线程退出时，其他线程也执行完了
        Thread.sleep(10000);
        //打印最终结果
        System.out.println(VALUE);
    }
```
执行的结果肯定不是0，因为int的++和--操作不是线程安全的。
当然为了解决这个问题，我们可以把VALUE设置成线程安全的类，比如AtomicInteger，这针对于资源是唯一的情况下可以这样做，虽然这么做有一定的性能损耗，这个是没有办法的，但是如果是多个资源，每个线程都可以分配到一个资源的情况下，使用线程安全类会降低性能，再说，万一我们的资源不是线程安全的呢，比如数据库连接。
### 多资源多线程怎么安全
这个时候ThreadLocal就登上历史舞台了。
```java
import java.util.concurrent.atomic.AtomicInteger;

public class IncThread implements Runnable{
    private static  ThreadLocal<Integer> VALUE = new ThreadLocal<>();
    @Override
    public void run() {
        VALUE.set(0);//初始化资源，比如获取数据库连接池
        iscr100();//模块1, 增加100
        descr100();//模块2, 减少100
        System.out.println(VALUE.get());
    }

    public void descr100(){
        for(int i = 0; i< 1000; i++){
            try {
                Integer integer = VALUE.get();
                integer++;
                //模拟IO，让CPC进行切换
            Thread.sleep(1);
            } catch (InterruptedException e) {
            }
        }

    }

    public void iscr100(){
        for(int i = 0; i< 1000; i++){
            Integer integer = VALUE.get();
            integer--;
            try {
            //模拟IO，让CPC进行切换
            Thread.sleep(1);
            } catch (InterruptedException e) {
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        IncThread myInc1 = new IncThread();
        Thread thread1 = new Thread(myInc1);
        Thread thread2 = new Thread(myInc1);
        thread1.start();
        thread2.start();
        //保证主线程退出时，其他线程也执行完了
        Thread.sleep(10000);
    }
}
```

到此为止，我们讲解了ThreadLocal的意义所在，不过有人会说了，像这种多个资源的同步的情况下，我们可以自己实现呀，比如使用Map，以线程名称作为KEY，以资源作为VALUE，提出这个方案的人员是很聪明的，ThreadLocal也确实是使用Map作为存储的，但是这个方案有两个坏处。
1. 通用性差: 每个框架的线程名称是不定的，比如Spring框架、Servlet等等，很难，也是没办法有一个通用的Map能实现这个功能。
2. 易用性低：我们自己实现一套机制，很难使用到别人已有的框架中，只能在我们自己的框架里使用。
那么，我们就不得不选择ThreadLocal了。

## ThreadLocal源码分析
既然ThreadLocal是为了解决多资源多线程下资源到处传递的问题，那么ThreadLocal至少要有set, get和delete这个三个方法，实际上ThreadLocal有5个方法:
| 描述符和返回值 | 方法名 | 描述 | 
| :-------------: | :-------------: | :-----:| 
|`T`|`get()`|Returns the value in the current thread's copy of this thread-local variable.|
|`protected T`| `initialValue()`|Returns the current thread's "initial value" for this thread-local variable.|
|`void`| `remove()`|Removes the current thread's value for this thread-local variable.|
|`void`| `set(T value)`|Sets the current thread's copy of this thread-local variable to the specified value.
|`static <S> ThreadLocal<S>`  |`withInitial(Supplier<? extends S> supplier)`| Creates a thread local variable.

### Set方法
```java
    public void set(T value) {
      //获取当前线程的索引
        Thread t = Thread.currentThread();
        //获取当前线程的ThreadLocalMap
        ThreadLocalMap map = getMap(t);
        if (map != null)
          //设置值
            map.set(this, value);
        else
          //第一次设置值
            createMap(t, value);
    }
```
可以看出，ThreadLocal能实现跨代码层次获取线程资源的根本原因是有了`Thread.currentThread()` 这个静态方法，这样ThreadLocal可以随时随地做任何操作，而不需要传入线程名称或者ID。

我们看下getMap(t)做了什么事情。
```java
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
```
而Thread类有一个成员变量:
`    ThreadLocal.ThreadLocalMap threadLocals = null;`
那么我们就能理解到，ThreadLocal存储的资源实际上存储到了Thread上，ThreadLocal只是作为一个能找到这个资源的索引。

### get()方法
我们知道了ThreadLocal实例是作为当前资源的索引，那么get()方法的源码就应运而生了。
```java
    public T get() {
        Thread t = Thread.currentThread();
        //获取到当前线程的ThreadLocalMap
        ThreadLocalMap map = getMap(t);
        //map存在时
        if (map != null) {
          //获取到资源
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        //map不存在或者没有set资源时，返回初始化的值
        return setInitialValue();
    }
```
setInitialValue方法定义: 
```java 
    private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }
```
如果get资源时不存在，则最终会调用initialValue()方法，初始化资源:
```java
    protected T initialValue() {
        return null;
    }
```
但是这个方法会最终返回null，所以如果我们set资源时，如果调用get会直接返回null。

### withInitial() 静态方法
为了不get到null，避免我们还要判断是否为null时再初始化ThreadLocal，ThreadLocal提供给我们一个静态方法，可以传入一个function来初始化这个ThreadLocal。
```java
    public static <S> ThreadLocal<S> withInitial(Supplier<? extends S> supplier) {
        return new SuppliedThreadLocal<>(supplier);
    }
```

### remove()方法
remove()就比较简单了，这里就不介绍了。
```java
     public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             // 当前线程的ThreadLocalMap存在时删除此资源。
             m.remove(this);
     }
```

## 最后

最后我们通过官方的解释来总结一下ThreadLocal:
ThreadLocal提供了线程局部变量，这些变量和普通的变量是不一样的，因为每一个线程的ThreadLocal都有自己的副本，并且是独立初始化的副本，其他线程是无法访问到的。使用ThreadLocal通常是与线程关联类中的私有静态字段，比如用户Id和事务Id等。

