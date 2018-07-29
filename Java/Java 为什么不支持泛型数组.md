# Java 为什么不支持泛型数组

##缘由
之前leetcode刷题的时候，某道题试图新建一个`HashMap`的数组，总是编译出错，然后突然反应过来，Java里是不能按照像通常新建基本类型的数组一样，直接新建泛型数组的。希望我能以比较简单明了的方式说明其中的原因。

##分析
首先对于Java数组，**数组必须明确知道内部元素的类型**，而且编译器会”记住“这个类型，每次往数组里插入新元素都会进行类型检查，不匹配会抛出`java.lang.ArrayStoreException`错误。

如果试图创建一个泛型数组，直接写成下面这样的话：
```
HashMap<Integer> map = new HashMap<Integer>[];
```
其实是无法编译通过的。
原因就是Java的泛型是通过**类型擦除**(type erasure)来实现的。什么是类型擦除呢，简单来说Java在编译期间，所有的泛型信息都会被擦除掉。

如在代码中定义的`List<object>`和`List<String>`等类型，在编译后都会变成`List`。

``` java
List<String> strList = new ArrayList<String>(); 
List<Object> objList = new ArrayList<Object>();
List rawList = new ArrayList();
```

比如上面3个List，擦除类型参数以后，List中的实际元素都是Object。JVM看到的只是List，而由泛型附加的类型信息对JVM来说是不可见的。

Java编译器会在**编译时**尽可能的发现可能出错的地方，但是仍然无法避免在运行时刻出现类型转换异常的情况。所以由于类型擦除的原因，Java是禁止直接创建泛型数组实例的。

## 泛型数组

如何创建真正的泛型数组呢？我们不能直接创建，但可以把数组强制转换成泛型类型。比如：

``` java
List<Integer>[] genericArray = (List<Integer>[])new ArrayList[10];
```

这样编译的时候并不会报错，因为Java虽然禁止直接创建泛型数组实例，但并没有禁止**声明一个泛型数组引用**。所以仍然可以通过强制转型原生类型数组的方式，绕过限制。

虽然编译期不会报错，但是这样做仍然有潜在的风险，因为类型擦除的存在，遇到下面这种情况运行期间就挂了：

``` java
List<Integer>[] genericArray = (List<Integer>[])new ArrayList[10];
genericArray[0] = new ArrayList<String>(Arrays.asList(new String[]{"Hello"}));
```

## 总结

Java 中不允许直接创建泛型数组，所以并不建议通过各种方式绕过编译器的限制，这样会带来在代码出错时，编译器也有可能无法及时发现错误，从而带来潜在的风险。

想要用泛型数组的时候，还是老老实实用`ArrayList`吧！。

参考文章：
[[1]][java为什么不支持泛型数组？ - 胖胖的回答 - 知乎](https://www.zhihu.com/question/20928981/answer/117521433)

[[2]][Java 泛型总结（二）：泛型与数组](https://segmentfault.com/a/1190000005179147)
