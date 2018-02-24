Collection是所有列表类容器的顶层接口，在Collection框架的位置如下图所示![这里写图片描述](http://img.blog.csdn.net/20180131144934330?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZmVpZ2Vzd2p0dQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)，

没有类直接实现Collection接口的，Collection和真正的实现类中间会有其他接口或者抽象类存在，后面我们一个一个的讲。
Collection接口作为Collection框架的顶层接口，几乎声明了所有Collection型容器的所有方法，是否重复List和Set接口对它进行了补充，我们看下Collection有哪儿写方法声明。

| 描述符和返回值 | 方法名 | 方法描述 | 
| :---------------: | :---------: | :--------: |
|`boolean`|`add(E e)`|向容器里添加元素，如果容器不允许添加重复的元素，比如Set，则会返回失败，如果不是因为重复的原因返回失败，一定要抛出异常。|
|`boolean`|`addAll(Collection<? extends E> c)`|和add()方法一样，这个方法是增加一个容器里的内容到现有的容器中。|
|`void`|`clear()`|清空容器里的所有元素。|
|`boolean`|`contains(Object o)`|是否包括一个元素，比较方法一定是通过equals判断，而不是判断它的引用 |
|`boolean`|`containsAll(Collection<?> c)`|判断现有容器是否包括参数容器里的所有元素，如果容器类型不一样或者转换失败，会抛出异常。|
|`boolean`|  `equals(Object o)`| 重写了Object的equals方法。|
|`int`|  `hashCode()`| 重写Obejct的hashCode()方法。|
|`boolean`  |`isEmpty()` |容器是否为空。|
|`Iterator<E>`  |`iterator()`| 返回容器一个迭代器。|
|`default Stream<E>`  |`parallelStream()`| 返回一个并发流，用的比较少。|
|`boolean`  |`remove(Object o)` |如果包括一个元素，则会从容器中删除它。|
|`boolean`|  `removeAll(Collection<?> c)`| 和remove一样，清空容器c的所有元素|
|`default boolean`  |`removeIf(Predicate<? super E> filter)`| 清空满足传入的过滤器的所有元素。|
|`boolean`  |`retainAll(Collection<?> c)` |和removeAll相反，这个方法会保留在容器C里的元素。|
|`int`  | `size()` |返回容器中元素的个数|
|`default Spliterator<E>`|  `spliterator()`| 创建容器的一个并发迭代器，Java8之后才有。|
|`default Stream<E>`|  `stream()`| 返回Stream流，java8之后才有|
|`Object[]`|  `toArray()`|按照容器中元素的顺序返回一个包括所有元素的数组，返回的数组已经和容器没有任何关系了，可以随便修改，是数组和容器的桥梁方法。 |
|`<T> T[]`|  `toArray(T[] a)` | 和上面的方法功能一样，参数传入的数组和返回的数组是同一个。 |


