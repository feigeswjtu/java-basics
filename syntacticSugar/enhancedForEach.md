# 普通数组的增强遍历
我们上一个例子就使用了普通数组的增强遍历循环，这让我们开发人员可以少些很多代码，可以看出普通数组的遍历循环最终是普通的for循环。

# Collection框架下list的增强遍历
那么Collection框架下的list或者set是怎么样个语法糖呢？我们举个例子:

```java
        List<String> list = Arrays.asList("我", "是", "中", "国", "人");
        for(String str:list){
            System.out.println(str);
        }
```
class文件反编译之后的结果:
```java
   public static void main(String[] var0) {
      List var1 = Arrays.asList(new String[]{"我", "是", "中", "国", "人"});
      Iterator var2 = var1.iterator();

      while(var2.hasNext()) {
         String var3 = (String)var2.next();
         System.out.println(var3);
      }

   }
```

可以看出Collection框架下的遍历循环最终还是使用了遍历器进行遍历。

值得提一下的是，如果我们使用了这个语法糖的遍历循环，就不能在循环体内删除元素，会报异常，这里就不做试验了。
