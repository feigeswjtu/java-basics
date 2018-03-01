# 背景
在看[阿里巴巴Java开发手册](https://wenku.baidu.com/view/9e06ca100812a21614791711cc7931b765ce7bc1)手册时，看到了有关hashCode()和equals()方法的使用规范。
- 只要重写 equals，就必须重写 hashCode。
- 因为 Set 存储的是不重复的对象，依据 hashCode 和 equals 进行判断，所以 Set 存储的对象必须重写这两个方法。
- 如果自定义对象做为 Map 的键，那么必须重写 hashCode 和 equals。

并且举了String 重写了 hashCode 和 equals 方法，所以我们可以非常愉快地使用 String 对象作为 key 来使用的例子。

我们看下String的hashCode()和equals()的源码:

# String的hashCode()
hashCode():
```java
    private int hash; // Default to 0
    public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;

            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }
```
代码就不一行一行的解释了，单纯的从代码角度上来说，我们也可以学到一些知识点。

- 缓存的重要性，这里的value属性就作为了缓存。
- 大多数情况下缓存是在使用时生成的。
- 计算hash时可以用类本身的属性的值与31乘积。

为什么是31，而不是32，33等其他数字呢？

- 31是一个素数，素数作用就是如果我用一个数字来乘以这个素数，那么最终的出来的结果只能被素数本身和被乘数还有1来整除。
- 31可以由`i*31 == (i<<5)-1`来表示,现在很多虚拟机里面都有做相关优化。
- 选择系数的时候要选择尽量大的系数。因为如果计算出来的hash地址越大，所谓的“冲突”就越少，查找起来效率也会提高。
- 并且31只占用5bits,相乘造成数据溢出的概率较小。


后面再说hashCode的设计原则，继续介绍equals()方法。

## String的equals()
```java
    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```

equals方法就没有什么好说的，为什么要这么写，这里就跟equals的设计原则有关了，后续会解释。
下面我们从开发者手册的三句话来解释hashCode和equals方法的特性，以及为什么要重新这两个方法。

## 只要重写equals，就必须重写hashCode
我们先举个例子:
Person类:
```java
public class Person {
    private String name;
    private int age;

    public Person() {
    }

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
    @Override
    public boolean equals(Object o){
        if(this == o){
            return true;
        }
        if(o instanceof Person){
            Person p = (Person)o;
            return this.age == p.getAge() && this.name.equals(p.getName());
        }
        return false;
    }
}
```
运行类:
```java
public class Application {
    public static void main(String[]args){
        Set<Person> set = new HashSet<>();
        Person p1 = new Person("Lilei", 25);
        Person p2 = new Person("Lilei", 25);
        set.add(p1);
        System.out.println("p1 equals p2: " + (p1 == p2));//1
        System.out.println("set contains p1: " + set.contains(p1));//2
        System.out.println("set contains p2: " + set.contains(p2));//3
    }
}
```

输出结果:
```java
p1 equals p2: false
set contains p1: true
set contains p2: false
```
可以看出来p1虽然等于p2（我们重写了Person类的equals方法），但是把p1放入一个Set之后，通过p2是无法取出来的，但是我们要的效果是能通过p2取出来，现实中肯定是有这样的使用场景的。

## 因为 Set 存储的是不重复的对象，依据 hashCode 和 equals 进行判断，所以 Set 存储的对象必须重写这两个方法

还是上面的例子，我们不重写Person类的equals方法，也不重写它的hashCode方法。 
Person类:
```java
public class Person {
    private String name;
    private int age;

    public Person() {
    }

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

运行类:
```java
public class Application {
    public static void main(String[]args){
        Set<Person> set = new HashSet<>();
        Person p1 = new Person("Lilei", 25);
        Person p2 = new Person("Lilei", 25);
        set.add(p1);
        set.add(p2);
        System.out.println("set size is: " + set.size());
        //System.out.println("p1 equals p2: " + (p1 == p2));
        //System.out.println("set contains p1: " + set.contains(p1));
        //System.out.println("set contains p2: " + set.contains(p2));

    }
}
```
运行结果:
```java
set size is: 2
```

Set里面是不重复的，如果不重写Person类的hashCode和equals方法，这里p1和p2是可以同时放入Set对象里的。 
那么我们只重写了Person的hashCode方法能不能行呢？

```java
    @Override
    public int hashCode(){
        return name.hashCode() * 31 + age;
    }
```

运行结果:
```java
set size is: 2
```

最后重写Person类的equals()方法:
```java
    @Override
    public boolean equals(Object o){
        if(this == o){
            return true;
        }
        if(o instanceof Person){
            Person p = (Person)o;
            return this.age == p.getAge() && this.name.equals(p.getName());
        }
        return false;
    }

    @Override
    public int hashCode(){
        return name.hashCode() * 31 + age;
    }
```

运行结果:
```java
set size is: 1
```

## 如果自定义对象做为 Map 的键，那么必须重写 hashCode() 和 equals()
```java
这一点和第二点实际上是一样的，这里就不举例介绍了。 
实际上，Set的对象和Map的key的操作和hashCode以及equals方法有关，比如查这个对象是否在Set里，先根据hashCode定位到一个段，在根据equals进行确定是否存在，这样不需要把Set里所有的对象都遍历一遍，效率太低，Map的key是同样道理。

那么hashCode和equals的设计原则就呼之欲出了。
```

# equals()的设计原则
* 对称性: 如果x.equals(y)返回是true，那么y.equals(x)也应该返回是true。
* 反射性: x.equals(x)必须返回是true。
* 类推性: 如果x.equals(y)返回是true，而且y.equals(z)返回是true，那么z.equals(x)也应该返回是true。
* 一致性: 如果x.equals(y)返回是true，只要x和y内容一直不变，不管你重复x.equals(y)多少次，返回都是true。
* 非空性: x.equals(null)，永远返回是false；x.equals(和x不同类型的对象)永远返回是false。

# hashCode()的设计原则
* 在一个Java应用的执行期间，如果一个对象提供给equals做比较的信息没有被修改的话，该对象多次调用hashCode()方法，该方法必须始终如一返回同一个integer。
* 如果两个对象根据equals(Object)方法是相等的，那么调用二者各自的hashCode()方法必须产生同一个integer结果。
* 并不要求根据equals(java.lang.Object)方法不相等的两个对象，调用二者各自的hashCode()方法必须产生不同的integer结果。然而，程序员应该意识到对于不同的对象产生不同的integer结果，有可能会提高hash table的性能。

