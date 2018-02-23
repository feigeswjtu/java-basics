之前写过一篇关于[Java中普通代码块和static代码块的区别](./others/codeBlock.md)，大致讲解了普通代码块和Static代码的区别，但是并没有讲它们的加载执行顺序，本章就细细的将一下类的加载机制（初始化顺序）。

# 类生命周期
类的字节码从被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期包括：加载（Loading）、验证（Verification）、准备(Preparation)、解析(Resolution)、初始化(Initialization)、使用(Using)和卸载(Unloading)7个阶段。其中准备、验证、解析3个部分统称为连接（Linking）。类的生命周期如下图所示:
![这里写图片描述](http://img.blog.csdn.net/20180105165447562?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZmVpZ2Vzd2p0dQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
其中，加载、验证、准备、初始化、卸载在类的生命周期的顺序是不变的，那解析呢，它在某些情况下可能在初始化之后开始。这里要强调“开始”这两个字，因为类的生命周期里都是交叉完成的，通常会在执行一个阶段的过程中开始另外一个阶段。
# 加载
在加载阶段（可以参考java.lang.ClassLoader的loadClass()方法），虚拟机需要完成以下3件事情：
1. 通过一个类的全限定名来获取定义此类的二进制字节流（这里并没有指明要从一个Class文件中获取，可以从其他渠道，譬如：jar包、zip文件、网络、动态生成、数据库等）；
2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构；
3. 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口；
加载阶段和连接阶段（Linking）的部分内容（如一部分字节码文件格式验证动作）是交叉进行的，加载阶段尚未完成，连接阶段可能已经开始，但这些夹在加载阶段之中进行的动作，仍然属于连接阶段的内容，这两个阶段的开始时间仍然保持着固定的先后顺序。
## 非数组类型加载
这里的类加载要区分数组类型和非数组类型，其中非数组类型是开发人员最容易控制的，比如第一点获取二进制字节流的方式，都是开发人员可以控制的，除了这点开发人员可以也可以指定类加载器进行加载。
## 数组类型的加载
数组类型的类加载实际上是虚拟机完成的，开发人员控制不了，能控制的是数组的元数据的类型，一个数组类的创建要遵循以下原则:
1. 如果数组的元素类型引用类型，加载器会先按照类的加载过程加载，并且数组也将按照该类的加载器进行标识。
2. 如果数组的元素类型是普通类型，比如(int[] a)，虚拟机则将数组标注为和引导类加载器相关。
3. 数组类的可见性和它的元素类型一致，如果元素是引用类型，数组类的可见性默认是public。
第三点很好理解，如果一个类的可见性是protected或者private，这个类不能在它的包外或者包里访问，那么这个类的数组类型也不能在类的包外或者包里访问。
# 验证
验证是连接阶段的第一步，这一阶段的目的是为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。
验证阶段大致会完成4个阶段的检验动作：
1. 文件格式验证：验证字节流是否符合Class文件格式的规范；例如：是否以魔术0xCAFEBABE开头、主次版本号是否在当前虚拟机的处理范围之内、常量池中的常量是否有不被支持的类型、是否有不符合UTF-8编码的数据、是否有被删除和附加的信息。
2. 元数据验证：对字节码描述的信息进行语义分析（注意：对比javac编译阶段的语义分析），以保证其描述的信息符合Java语言规范的要求；例如：这个类是否有父类，除了java.lang.Object之外。
3. 字节码验证：通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的。
4. 符号引用验证：确保解析动作能正确执行。
5. 验证阶段是非常重要的，但不是必须的，它对程序运行期没有影响，如果所引用的类经过反复验证，那么可以考虑采用-Xverifynone参数来关闭大部分的类验证措施，以缩短虚拟机类加载的时间，但是如果是服务器的代码建议开启。

# 准备
准备阶段是正式为类变量分配内存并设置类变量初始值的阶段，这些变量所使用的内存都将在方法区（常量区）中进行分配。这时候进行内存分配的仅包括类变量（被static修饰的变量），而不包括实例变量，实例变量将会在对象实例化时随着对象一起分配在堆中。其次，这里所说的初始值“通常情况”下是数据类型的零值，假设一个类变量的定义为：
`public static int value=123;`
那变量value在准备阶段过后的初始值为0而不是123。因为这时候尚未开始执行任何java方法，而把value赋值为123的putstatic指令是程序被编译后，存放于类构造器()方法之中，所以把value赋值为123的动作将在初始化阶段才会执行。
至于“特殊情况”是指：public static final int value=123，即当类字段的字段属性是ConstantValue时，会在准备阶段初始化为指定的值，所以标注为final之后，value的值在准备阶段初始化为123而非0，如果不赋值的话，默认值就为0

# 解析
解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符7类符号引用进行。
这里解释一下什么是符号引用，什么是直接引用:
1. 符号引用是用一组字符引用一个类、常量等，符号因为是以一定的规范规定在class的文件中的。
2. 直接引用，也就是我们常说的引用，就是内存引用。

# 初始化
对我们开发来说，我们真正关心的就是初始化阶段的过程，了解过类在字节码的结构的开发人员清楚，类本身也有构造器，而不是代码里的那个构造方法，这个在字节码里就是`<clinit>()`方法。

类初始化阶段是类加载过程的最后一步，到了初始化阶段，才真正开始执行类中定义的java程序代码。在准备阶段，变量已经赋过一次系统要求的初始值，而在初始化阶段，则根据程序猿通过程序制定的主管计划去初始化类变量和其他资源，或者说：初始化阶段是执行类构造器`<clinit>()`方法的过程.
`<clinit>()`方法是由编译器自动收集类中的所有类变量的赋值动作和静态语句块static{}中的语句合并产生的，编译器收集的顺序是由语句在源文件中出现的顺序所决定的，静态语句块只能访问到定义在静态语句块之前的变量，定义在它之后的变量，在前面的静态语句块可以赋值，但是不能访问。如下：
```java
public class Test
{
    static
    {
        i=0;
        System.out.println(i);//这句编译器会报错：Cannot reference a field before it is defined（非法向前应用）
    }
    static int i=1;
}
```
`<clinit>()`方法与实例构造器<init>()方法不同，它不需要显示地调用父类构造器，虚拟机会保证在子类<init>()方法执行之前，父类的`<clinit>()`方法方法已经执行完毕，这个也是“先行先发生”的原则。
举个例子:
```java
//SuperClass
public class SuperClass
{
    static
    {
        System.out.println("SuperClass init!");
    }

    public SuperClass()
    {
        System.out.println("init SuperClass");
    }
}
//Main class
public class Main {
    public static void main(String[] args) {
        SuperClass superClass = new SuperClass();
    }
}
```
执行结果: 
SuperClass init!
init SuperClass

由于父类的`<clinit>()`方法先执行，也就意味着父类中定义的静态语句块要优先于子类的变量赋值操作。
`<clinit>()`方法对于类或者接口来说并不是必需的，如果一个类中没有静态语句块，也没有对变量的赋值操作，那么编译器可以不为这个类生产`<clinit>()`方法。
接口中不能使用静态语句块，但仍然有变量初始化的赋值操作，因此接口与类一样都会生成`<clinit>()`方法。但接口与类不同的是，执行接口的`<clinit>()`方法不需要先执行父接口的`<clinit>()`方法。只有当父接口中定义的变量使用时，父接口才会初始化。另外，接口的实现类在初始化时也一样不会执行接口的`<clinit>()`方法。
虚拟机会保证一个类的`<clinit>()`方法在多线程环境中被正确的加锁、同步，如果多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的`<clinit>()`方法，其他线程都需要阻塞等待，直到活动线程执行`<clinit>()`方法完毕。如果在一个类的`<clinit>()`方法中有好事很长的操作，就可能造成多个线程阻塞，在实际应用中这种阻塞往往是隐藏的。

虚拟机规范严格规定了有且只有5中情况（jdk1.7）必须对类进行“初始化”，并且加载、验证、准备自然需要在此之前开始：
1. 遇到new,getstatic,putstatic,invokestatic这失调字节码指令时，如果类没有进行过初始化，则需要先触发其初始化。生成这4条指令的最常见的Java代码场景是：使用new关键字实例化对象的时候、读取或设置一个类的静态字段（被final修饰、已在编译器把结果放入常量池的静态字段除外）的时候，以及调用一个类的静态方法的时候。
2. 使用java.lang.reflect包的方法对类进行反射调用的时候，如果类没有进行过初始化，则需要先触发其初始化。
3. 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
4. 当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类。
5. 当使用jdk1.7动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果REF_getstatic,REF_putstatic,REF_invokeStatic的方法句柄，并且这个方法句柄所对应的类没有进行初始化，则需要先出触发其初始化。

为了解释这个我们举个例子:
```java
//SuperClass
public class SuperClass
{
    static
    {
        System.out.println("SuperClass init!");
    }

    public static final Integer Value = 111;

    public SuperClass()
    {
        System.out.println("init SuperClass");
    }
}
//Subclass
public class SubClass extends SuperClass{
}
//Main
public class Main {
    public static void main(String[] args) {
        System.out.println(SubClass.Value);
    }
}
```
执行结果:
```data
SuperClass init!
111
```
只执行了父类SuperClass的`<clinit>()`方法，子类SubClass的方法并没有执行，因为没有触发类的初始化。
那么，调用类的常量一定会触发类的初始化么？
我们修改一下SuperClass的Value常量，修改为String：
```java
```java
//SuperClass
public class SuperClass
{
    static
    {
        System.out.println("SuperClass init!");
    }

    public static final String Value = "111";

    public SuperClass()
    {
        System.out.println("init SuperClass");
    }
}
//Subclass
public class SubClass extends SuperClass{
}
//Main
public class Main {
    public static void main(String[] args) {
        System.out.println(SubClass.Value);
    }
}
```
输出结果:
```data
111
```
这里就是常量传播，也就是编译器做的优化，会把不可变的、占用内存小的常量转移并固化到使用的地方，不会在通过类去定位这个常量的值。
再举一个例子:
我们修改一下Main方法，在main方法里初始化一个数组:
```java
public class Main {
    public static void main(String[] args) {
        SubClass [] subClasses = new SubClass[2];
    }
}
```
因为不满足以上五个触发初始化的时机，所以这个例子不会输出任何结果。
最后我们举个终极例子:
```java
public class StaticTest
{
    public static void main(String[] args)
    {
        staticFunction();
    }

    static StaticTest st = new StaticTest();

    static
    {
        c=3;
        System.out.println("1");
    }

    {
        System.out.println("2");
        System.out.println(c);
    }

    StaticTest()
    {
        System.out.println("3");
        System.out.println("a="+a+",b="+b);
    }

    public static void staticFunction(){
        System.out.println("4");
    }

    int a=110;
    static int b =112;
    static int c;
}
```
待会儿我们给出输出结果，分析一下执行过程。
1. 这个类是入口方法类，所以在进入Main方法之前会进行准备阶段，b和c初始化为0，然后执行初始化`<clinit>()`方法。
2. 执行到`static StaticTest st = new StaticTest();` 会触发类的实例化。
3. 由于初始化`<clinit>()`方法已经在执行了，所以直接会初始化非static代码块，先执行`System.out.println("2");`打印出2。执行`System.out.println(c);`时，由于c只进行初始化，并没有进行赋值，所以打印出0。
4. 接着执行成员变量的初始化，a=110。
5. 紧接着执行构造方法，执行`System.out.println("3");`，打印出3，执行`System.out.println("a="+a+",b="+b);`，a的值为110，b是类的常量，只进行了初始化，未进行赋值为0，所以打印出a=110,b=0。
6. 静态成员变量st执行完之后，进行执行静态方法，c赋值为3，打印出1。
7. `<clinit>()`执行完毕。
8. 接着执行staticFunction方法，打印出4。
最终的执行结果:
```java
2
0
3
a=110,b=0
1
4
```

