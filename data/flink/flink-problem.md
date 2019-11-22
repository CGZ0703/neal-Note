# Flink 中遇到的 坑

## Flink 算子 使用Java 8 Lambda

Java 8引入了几种新的语言功能，旨在实现更快，更清晰的编码。凭借最重要的功能，即所谓的“Lambda表达式”，它打开了函数式编程的大门。Lambda表达式允许以直接的方式实现和传递函数，而无需声明其他（匿名）类。

### 问题

学习Flink的过程中，打算操作一遍Java的API，一直习惯了用函数式编程解决数据处理的各种问题，遇到算子，当然是上Lambda啦。

先来一个 map() 美滋滋

不过遇到了需要压平的地方， 还是 flatMap() 用的爽

然后就

```log
Caused by: org.apache.flink.api.common.functions.InvalidTypesException: The generic type parameters of 'Collector' are missing. 
  In many cases lambda methods don't provide enough information for automatic type extraction when Java generics are involved. 
  An easy workaround is to use an (anonymous) class instead that implements the 'org.apache.flink.api.common.functions.FlatMapFunction' interface. 
  Otherwise the type has to be specified explicitly using type information.
```

微笑一个 ^_^

我不想用 匿名对象 来解决问题，`@FunctionalInterface`

### 解决

这时候，机智的我发现！

```log
Exception in thread "main" org.apache.flink.api.common.functions.InvalidTypesException:
  The return type of function 'main(MyFlinkDemo.java:26)' could not be determined automatically, due to type erasure. 
  You can give type information hints by using the returns(...) method on the result of the transformation call, or by letting your function implement the 'ResultTypeQueryable' interface.
```

说是需要给显式的`returns(...)`方法来指定返回值的类型。在`flatMap()`后面加上`return(Types.STRING)`，就可以了。

### 扩充

上网查阅资料发现：

Flink可以自动从方法签名OUT map(IN value)的实现中提取结果类型信息，因为OUT不是泛型而是整数。

不幸的是，像flatMap()这样带有签名void flatMap(IN value, Collector<OUT> out)的函数被Java编译器编译成void flatMap(IN value, Collector out)。这使得Java无法自动推断输出类型的类型信息。

在这种情况下，需要显式指定类型信息，否则输出将被视为Object导致无效序列化的类型。

使用带有泛型返回类型的map()函数时也会出现类似的问题。

解决方式：

1. 使用 `returns(...)`
2. 使用指定类型的类替代
   - 替代FunctionClass
   - 替代返回的SubClass
3. 使用匿名对象

官方给出的例子：

```java
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.tuple.Tuple2;

// use the explicit ".returns(...)"
env.fromElements(1, 2, 3)
    .map(i -> Tuple2.of(i, i))
    .returns(Types.TUPLE(Types.INT, Types.INT))
    .print();

// use a class instead
env.fromElements(1, 2, 3)
    .map(new MyTuple2Mapper())
    .print();

public static class MyTuple2Mapper extends MapFunction<Integer, Tuple2<Integer, Integer>> {
    @Override
    public Tuple2<Integer, Integer> map(Integer i) {
        return Tuple2.of(i, i);
    }
}

// use an anonymous class instead
env.fromElements(1, 2, 3)
    .map(new MapFunction<Integer, Tuple2<Integer, Integer>> {
        @Override
        public Tuple2<Integer, Integer> map(Integer i) {
            return Tuple2.of(i, i);
        }
    })
    .print();

// or in this example use a tuple subclass instead
env.fromElements(1, 2, 3)
    .map(i -> new DoubleTuple(i, i))
    .print();

public static class DoubleTuple extends Tuple2<Integer, Integer> {
    public DoubleTuple(int f0, int f1) {
        this.f0 = f0;
        this.f1 = f1;
    }
}
```

> ！！！ Flink supports the usage of lambda expressions for all operators of the Java API, however, whenever a lambda expression uses Java generics you need to declare type information explicitly.

### 总结

表示还是Scala写起来舒服 ^_^

