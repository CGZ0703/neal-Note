---
description: Lombok makes java cool again!
---

# Lombok

我们的开发中有很多时间在写一些重复冗余的代码，

这将浪费我们很大的一块时间。

比如实体类中的getter/setter方法，默认的构造器方法等

## 设置Lombok

设置Lombok十分简单，

Lombok是一个编译器插件，在编译器处理源代码之前，将其中的注释转换为java语句，所以在运行时不需要提供Lombok的依赖。因此，需要将Lombok添加到构建工具中，比如maven：

```xml
///gradle
plugins {
    id 'io.franzbecker.gradle-lombok' version '1.14'
    id 'java'
}
repositories {
    jcenter() // or Maven central, required for Lombok dependency
}
lombok {
    version = '1.18.4';
    sha256 = ""
}


///maven
<dependencies>
	<dependency>
		<groupId>org.projectlombok</groupId>
		<artifactId>lombok</artifactId>
		<version>1.18.8</version>
		<scope>provided</scope>
	</dependency>
</dependencies>
```

推荐给一些朋友用后，说是代码一直报错很烦，因为Lombok是编译器插件， IDE发现源代码中所使用的实例的方法在源代码中找不到定义，IDE会认为这是错误，所以可以需要通过安装插件来识别Lombok的注解

## Lombok使用

一个框架的学习方式最快的就是使用。

### 为POJO添加注解

我们使用java对象（pojo）将数据与处理的部分分开，使我们的代码更易于阅读，一个简单的POJO含有一些私有的属性及相应的getter和setter方法。

普遍的开发中，我们会写很多的toString，getter和setter方法，这会让我们的代码看起来很长，并且其中大部分都是重复的工作，使用Lombok可以使POJO更加简洁明了，无需编写更多的代码，使用Lombok，我们可以用@Data注释来简化最基本的POJO：

```java
@Data
public class User {
    private UUID userId;
    private String email;
}
```

该@Data注释包含有多个Lombok注释。

* @ToString 生成 toString\(\)方法，包含类名和每个字段及其值的对象的打印。
* @EqualsAndHashCode 生成equals和hashCode方法的实现，默认情况下，它们使用所有非静态，非transient字段，但是可配置。
* @Getter/@Setter为私有字段生成getter和setter方法。
* @RequiredArgsConstructor生成带参数的构造函数，其中需要参数是常亮字段和带@NonNull注释的字段

一个注解虽然简单，但是会让程序增加复杂性及限制了并发量，可以使用其他一些有用的Lombok注解来标注每一个属性。

```java
@Value
@Builder(toBuilder = true)
public class User {
    @NonNull
    UUID userId;
    @NonNull
    String email;
    @Singular
    Set<String> favoriteFoods;
    @NonNull
    @Builder.Default
    String avatar = "default.png"
}
```

@Value注释，跟@Data相似，只是所有字段都默认为private和final，并且不生成setter。这些特性使注释@Value的对象有效地不变，由于字段都是常量，因此没有无参的构造函数。与之替代的是Lombok使用了注解@AllArgsConstructor生成所有参数构造函数，这产生了一个功能完备，有效不可变的对象。

但是，如果只能用all args构造函数创建对象，那么不可变是不太有用的。Joshua Bloch在 《Effective Java》解释，当面临着许多构造函数时，应该使用建造者。这就是Lombok的@Builder的作用，自动生成构建器内部类：

```java
User user = User.builder()
    .userId(UUID.random())
    .email("cgz@cgz.com")
    .favoriteFood("burritos")
    .favoriteFood("dosas")
    .build()
```

使用Lombok生成的构建器可以轻松创建具有多个参数的对象，



### Lombok注解说明

`val`：用在局部变量前面，相当于将变量声明为final

 `@NonNull`：给方法参数增加这个注解会自动在方法内对该参数进行是否为空的校验，如果为空，则抛出NPE（NullPointerException）

 `@Cleanup`：自动管理资源，用在局部变量之前，在当前变量范围内即将执行完毕退出之前会自动清理资源，自动生成try-finally这样的代码来关闭流

 `@Getter/@Setter`：用在属性上，再也不用自己手写setter和getter方法了，还可以指定访问范围

 `@ToString`：用在类上，可以自动覆写toString方法，当然还可以加其他参数，例如@ToString\(exclude=”id”\)排除id属性，或者@ToString\(callSuper=true, includeFieldNames=true\)调用父类的toString方法，包含所有属性

 `@EqualsAndHashCode`：用在类上，自动生成equals方法和hashCode方法

 `@NoArgsConstructor, @RequiredArgsConstructor and @AllArgsConstructor`：用在类上，自动生成无参构造和使用所有参数的构造函数以及把所有@NonNull属性作为参数的构造函数，如果指定staticName = “of”参数，同时还会生成一个返回类对象的静态工厂方法，比使用构造函数方便很多

 `@Data`：注解在类上，相当于同时使用了`@ToString`、`@EqualsAndHashCode`、`@Getter`、`@Setter`和`@RequiredArgsConstrutor`这些注解，对于`POJO类`十分有用

 `@Value`：用在类上，是@Data的不可变形式，相当于为属性添加final声明，只提供getter方法，而不提供setter方法

 `@Builder`：用在类、构造器、方法上，为你提供复杂的builder APIs，让你可以像如下方式一样调用`Person.builder().name("Adam Savage").city("San Francisco").job("Mythbusters").job("Unchained Reaction").build();`更多说明参考[Builder](https://link.jianshu.com?t=https%3A%2F%2Fprojectlombok.org%2Ffeatures%2FBuilder.html)

 `@SneakyThrows`：自动抛受检异常，而无需显式在方法上使用throws语句

 `@Synchronized`：用在方法上，将方法声明为同步的，并自动加锁，而锁对象是一个私有的属性`$lock`或`$LOCK`，而java中的synchronized关键字锁对象是this，锁在this或者自己的类对象上存在副作用，就是你不能阻止非受控代码去锁this或者类对象，这可能会导致竞争条件或者其它线程错误

 `@Getter(lazy=true)`：可以替代经典的Double Check Lock样板代码

 `@Log`：根据不同的注解生成不同类型的log对象，但是实例名称都是log，有六种可选实现类

*  `@CommonsLog` Creates log = org.apache.commons.logging.LogFactory.getLog\(LogExample.class\);
*  `@Log` Creates log = java.util.logging.Logger.getLogger\(LogExample.class.getName\(\)\);
*  `@Log4j` Creates log = org.apache.log4j.Logger.getLogger\(LogExample.class\);
*  `@Log4j2` Creates log = org.apache.logging.log4j.LogManager.getLogger\(LogExample.class\);
*  `@Slf4j` Creates log = org.slf4j.LoggerFactory.getLogger\(LogExample.class\);
*  `@XSlf4j` Creates log = org.slf4j.ext.XLoggerFactory.getXLogger\(LogExample.class\);

  






















