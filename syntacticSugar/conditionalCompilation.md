我们知道C/C++的条件编译是通过#ifdef来实现的，Java的编译器在编译阶段就会处理条件编译，举个例子:

```java
    public static void main(String[] args) {
        if(true){
            System.out.println("true");
        }else {
            System.out.println("false");
        }
    }
```

反编译之后的代码:

```java
   public static void main(String[] var0) {
      System.out.println("true");
   }
```
由于条件编译的实现方式使用了if语句，所以它必须遵循最基本的Java语法，只能写在方法体内部，隐藏它只能实现语句的基本块的选择性编译，没办法根据条件调整整个Java类的结构，不管怎样，条件编译能提高运行效率就行。

