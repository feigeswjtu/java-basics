知道Spring的都知道Spring的切面编程AOP(Aspect Oriented Programming)，这里我们不讲Spring的切面，后面有机会我们再来解剖Spring的切面编程，我们想讲解一下普通的Java代码中怎么实现AOP，有两种方式实现AOP切面，一种是原生SDK实现，一种是基于三方包cglib。
# 基于JDK反射机制实现
先介绍一下JDK原生的，JDK原生的是基于接口编程:
先定义一个接口:
```java
public interface ISayHelloWorld {
    public String say();
}
```

实现这个接口:
```java
public class ManSayHelloWorld implements ISayHelloWorld{
    @Override
    public String say() {
        System.out.println("Hello world!");
        return "MAN";
    }
}
```
实现ManSayHelloWorld的切面代理，原生的AOP需要实现InvocationHandler接口，才能实现AOP。
```java
public class AOPHandle implements InvocationHandler{
    private Object obj;
    AOPHandle(Object obj){
        this.obj = obj;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //方法返回值
        System.out.println("前置代理");
        //反射调用方法
        Object ret=method.invoke(obj, args);
        //声明结束
        System.out.println("后置代理");
        //返回反射调用方法的返回值
        return ret;
    }
}

```

测试这个代码:
```java
public class Main {
    public static void main(String[] args) {
        ManSayHelloWorld sayHelloWorld = new ManSayHelloWorld();
        AOPHandle handle = new AOPHandle(sayHelloWorld);
        ISayHelloWorld i = (ISayHelloWorld) Proxy.newProxyInstance(ManSayHelloWorld.class.getClassLoader(), new Class[] { ISayHelloWorld.class }, handle);
        i.say();
    }
}
```


执行结果:
```java
前置代理
Hello world!
后置代理
```

# 基于cglib的AOP实现
接下来我们实现一个机遇cglib的切面，实现功能和上面一样，cglib不需要定义接口，普通的类就可以：
```java
public class SayHello {
    public void say(){
        System.out.println("hello world!");
    }
}
```

实现切面:
```java
public class CglibProxy implements MethodInterceptor {
    private Enhancer enhancer = new Enhancer();
    public Object getProxy(Class clazz){
        enhancer.setSuperclass(clazz);
        enhancer.setCallback(this);
        return enhancer.create();
    }
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("前置代理");
        //通过代理类调用父类中的方法
        Object result = methodProxy.invokeSuper(o, objects);

        System.out.println("后置代理");
        return result;
    }
}
```

测试代码:
```java
public class Main {
    public static void main(String[] args) {
        CglibProxy proxy = new CglibProxy();
        //通过生成子类的方式创建代理类
        SayHello proxyImp = (SayHello)proxy.getProxy(SayHello.class);
        proxyImp.say();
    }
}
```

执行结果:
```java
前置代理
hello world!
后置代理
```


那么这两种方式有什么相同点和区别呢？
相同点是都使用了代理模式。
# 代理模式
不同点是前者使用了接口代理，后者使用了继承代理，什么意思呢?
## 接口代理
代理会生成一个类，这个类继承这个接口，并且代理要切面的类，比如上面的例子，AOPHandle会生成一个类，姑且类名叫AspectSayHelloWorld implements ISayHelloWorld，实现say方法时实际上是调用了ManSayHelloWorld的say方法，只是在调用前和调用后做了一些事情而已。
看一下newProxyInstance方法的源码，就知道了:
```java
Objects.requireNonNull(h);

        final Class<?>[] intfs = interfaces.clone();
        final SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
        }

        /*
         * Look up or generate the designated proxy class.
         */
        Class<?> cl = getProxyClass0(loader, intfs);

        /*
         * Invoke its constructor with the designated invocation handler.
         */
        try {
            if (sm != null) {
                checkNewProxyPermission(Reflection.getCallerClass(), cl);
            }

            final Constructor<?> cons = cl.getConstructor(constructorParams);
            final InvocationHandler ih = h;
            if (!Modifier.isPublic(cl.getModifiers())) {
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        cons.setAccessible(true);
                        return null;
                    }
                });
            }
            return cons.newInstance(new Object[]{h});
        } catch (IllegalAccessException|InstantiationException e) {
            throw new InternalError(e.toString(), e);
        } catch (InvocationTargetException e) {
            Throwable t = e.getCause();
            if (t instanceof RuntimeException) {
                throw (RuntimeException) t;
            } else {
                throw new InternalError(t.toString(), t);
            }
        } catch (NoSuchMethodException e) {
            throw new InternalError(e.toString(), e);
        }
```

## 继承代理
继承代理是生成一个代理类，这个类继承于我们需要切面的类，比如上面的 SayHello proxyImp = (SayHello)proxy.getProxy(SayHello.class);这段代码就返回了SayHello的一个子类，代理了SayHello的say()方法，并且在这个方法前后做了一些事情。

## 其他代理方式

还有一些方式可以实现代理功能，但是不能称作为代理模式了，比如AspectJ，AspectJ的原理是在编译器修改被代理的类的字节码的方式，也就是重新实现这个类的功能，不需要在运行期做任何事情，这里就不具体介绍了，后续会出一个Spring系列，再详细介绍AspectJ。
