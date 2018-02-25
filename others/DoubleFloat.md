今天线上出现了一个很奇怪的问题，业务需求是从一个服务方获取到商品的价格（元），是字符串形式，例如是String: "9.90"，通过一定的方法转换为分存到数据库里却变成了989。开始怀疑是服务给的数据问题，后台写个Test测试一下，尴尬的发现是自己的基础知识太不扎实，对float/double类型的计算认知不够。

测试代码如下:

```java
public class Test {
    public static void main(String []args){
        String price = "9.90";
        System.out.println(Float.parseFloat(price));
        System.out.println(Float.parseFloat(price)*100);
        System.out.println((int)(Float.parseFloat(price)*100));
    }

}
```

输出结果:
```data
9.9
989.99994
989
```

发现是float型进行运算时丢失精度，并且只有小数部分会丢失精度，把price改为小数部分为零，再测试:

```java
public class Test {
    public static void main(String []args){
        String price = "999.00";
        System.out.println(Float.parseFloat(price));
        System.out.println(Float.parseFloat(price)*100);
        System.out.println((int)(Float.parseFloat(price)*100));
    }
}

```

输出:
```java
999.0
99900.0
99900
```

怎么解决这个问题呢，google一下吧，发现可以使用BigDecimal这个类避免丢失精度。

```java
public class Test {
    public static void main(String []args){
        String price = "9.90";
        BigDecimal b = new BigDecimal(price);
        BigDecimal aa = b.multiply(new BigDecimal(100));
        System.out.println(aa.floatValue());
        System.out.println(aa.intValue());
    }
}
```

输出:
```java
990.0
990
```

除了使用BigDecimal解决这个问题还有其他方案吗，毕竟BigDecimal很占用内存和CPU，当然有，既然只有小数部分会丢失精度，只要最终的计算结果是Integer型，也就是最后的结果保证是整数，就可以先消除小数部分，再进行计算。

丢失精度示例:
```java
public class Test {
    public static void main(String []args){
        int rate = Integer.parseInt("100");
        int f = 510;
        System.out.println(f*1.0/rate*100);
        System.out.println((int)(f*1.0/rate*100));
    }
}
```

输出:
```java
510.0
510
```
只是把*100的计算提到前面，避免了小数部分的存在，就不会丢失精度。但是这个方法的前提是最终计算结果必须是整数，慎用！
