[上一篇文章](http://blog.csdn.net/feigeswjtu/article/details/79198617)我们讲了Java容器的概况包括Collection和Map框架，Collection最顶上的接口是java.lang.Iterable，Collection里的其他类和接口都是在java.util里，但是Iterable确实在java.lang下。

Iterable翻译成中文就是可迭代的，就是说实现了Iterable接口的类必须是可迭代的类，Iterable声明了以下三个方法:

| 描述符和返回值类型 | 方法名和描述 |
| :---------------: | :-----------:|
| `default void` |  `forEach(Consumer<? super T> action)` 传入一个Function来遍历元素，直到遍历完或者抛出一个异常才结束，一般用于单纯的遍历操作|
| `Iterator<T>` |  `iterator()` 返回一个泛型元素为 T 迭代器迭代器|
| `default Spliterator<T>` | `spliterator()` 在iterator的基础上返回一个并行迭代器Spliterator|

其中`Spliterator<T>` 涉及到流的知识，这里就不介绍了，我们用的比较少。
