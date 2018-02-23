变长参数就是参数个数不确定，但是变长参数有条件的：

1. 变长的那一部分参数一定要具有相同的类型。
2. 变长参数必须位于方法参数列表的最后面。

变长参数同样是Java中的语法糖，其内部实现是Java数组。
```java
    static void printStrs(String... strs){
        for (String str:strs){
            System.out.println(str);
        }
    }
    public static void main(String[] args) {
        printStrs("我", "是", "中", "国", "人");
    }
```
class文件反编译之后的代码:
```java
    static transient void printStrs(String as[])
    {
        String as1[] = as;
        int i = as1.length;
        for(int j = 0; j < i; j++)
        {
            String s = as1[j];
            System.out.println(s);
        }

    }

    public static void main(String args[])
    {
        printStrs(new String[] {
            "\u6211", "\u662F", "\u4E2D", "\u56FD", "\u4EBA"
        });
    }
```

变长参数让我们少写了一些代码，很方便。
