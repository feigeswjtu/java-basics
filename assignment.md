Java中变量分两种，一种是对象的引用，另一种是基本类型，对象的引用是通过操作符new出来一个对象，并且把这个对象的句柄(C++中的称呼)赋值给一个变量，基本变量就不多说了，比如int,float, char等等，下面分别解释一下引用和基本类型的赋值。

## 引用的赋值

引用就是一个对象的别名，操作这个引用就相当于在操作这个对象，如果有两个变量T1和T2他们都是同一个类new出来的，那么如果T1=T2;执行之后，那么T1和T2都是同一个对象的引用，T1原来指向的对象如果没有其他变量引用，那么它不久之后就会被垃圾回收器回收。所以对一个引用赋值一定要小心，可能会出现一些不可意料的结果。
```java
  Integer a = 1;
  Integer b = 2;
  a = b;
```


## 基本类型的赋值

基本类型的赋值就非常简单了，就是对一个基本类型进行简单的赋值而已，比如:
```java
  int a = 1;
  int b = 2;
  a = b;
```

第一步和第二步是一样的，都会在堆栈中生成一个长度为int型长度(java中都是32个字节)，第三步的在底层的操作就是把b在堆栈中的内存数据cpy到a所在的内存中，这样a,b的值是一致的。

## 方法参数的赋值

说到这，简单的说一下方法的形参分别是对象的引用和基本变量时，方法调用的时候会出现什么情况呢。


在说这个之前，先说一下方法调用的时候，其中对形参会做什么操作呢？

比如下面的代码：
```java
public class Main {
        static int func(int b) {
                b++;
                return b;
        }

        public static void main(String[] args) {
                int a = 1;
                int c = func(a);
                System.out.println("a = " + a);
                System.out.println("c = " + c);
        }
}
```
输出结果
```data
a = 1
c = 2
```

不管这个形参是基本类型还是引用，在方法调用的时候，都会执行赋值操作，也就是b=a;这个操作，那么就可以回归到上面的说明。

### 形参为引用类型
```java
class Test{
        public int ele;
};
public class Main {
        public static Test func(Test a) {
                a.ele++;
                return a;
        }
        public static void main(String[] args){
                Test a = new Test();
                a.ele= 1;
                Test b = func(a);
                System.out.println("a = " + a.ele);
                System.out.println("b = " + b.ele);
        }
}

```

输出结果:
```data
a = 2
b = 2
```
形参为对象的引用的时候，在方法执行中，b和a引用了同一个对象，那么对b的操作也就是对a的操作，这样说是不确切的，对b的操作和对a的操作都是对它们指向的相同对象操作，a和b的最终的输出结果都是2。

### 形参为基本类型的时
```java
public class Main {
    public static int func(int a){
        a++;
        return a;
    }
    public static void main(String[] args){
        int a = 1;
        int b = func(a);
        System.out.println("a = " + a);
        System.out.println("b = " + b);
    }

}

```
输出结果:

```data
a = 1
b = 2
```

由于基本类型的赋值是真的赋值操作，在内存基本上，就是copy变量b所在的内存数据到a所在的内存中，所以在执行之后，a和b的值就不同了分别是1和2。

以上就是对引用赋值和对基本变量的赋值的区别，在使用的时候一定要注意。




