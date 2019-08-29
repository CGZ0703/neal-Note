# JAVA类加载器

JVM把描述类的数据从Class文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的Java类型。这就是类加载机制。

### Class文件

**Class文件结构中只有两种数据类型：**

> 1.  **无符号数**：属于基本的数据类型，以u1、u2、u4、u8分别代表1、2、4、8个字节的无符号数，可以用来描述数字、索引引用、数量值、或者按照UTF-8编码的字符串值；
> 2.  **表**：由多个无符号数或者其他表作为数据项构成的复合数据类型，所有表都习惯以"\_info"结尾；
> 3. 所以，Class文件本质上就是一张表；

  
**Class文件结构的内容组成：**

> 1.  **魔数**：每个Class文件的前4个字节，JAVA语言魔数：CAFEBABE，16进制数，唯一左右就是确认文件是否是一个被虚拟机接受的Class文件；
> 2.  **Class文件版本**：Class文件第5、6个字节是次版本号，7、8个字节是主版本号；
> 3.  **常量池**：从第9个字节开始，主要存放两大类常量：（1）字面量；（2）符号引用；
> 4.  **访问标志**：紧跟常量池之后的两个字节代表访问标志，代表类或接口的层次信息；
> 5.  **类索引、父类索引与接口索引集合**：紧跟访问标志之后的就是类索引、父类索引与接口索引，都是u2类型数据，Class文件中由这三项数据来确定类的继承关系；
> 6.  **字段表集合**：紧接着就是字段表集合，用于描述接口或类中声明的变量；
> 7.  **方法表集合**：紧接着就是方法表集合，用于描述接口或类中声明的方法；

### 触发类加载器的时机

* **创建类的实例**：使用new关键字实例化对象；
* **访问类的静态变量**：getstatic或putstatic，读取或设置一个类的静态变量（不包括被final修饰的静态变量）；
* **访问类的静态方法**：invokestatic调用一个类的静态方法；
* **使用java.lang.reflect进行反射调用**：如，Class.forName\("xxxxx"\)；
* **子类初始化时，会先初始化父类**；

### 被动引用

1. 通过子类引用父类的静态字段，不会导致子类初始化，只会初始化父类；
2. 通过数组定义来引用类，不会触发类的初始化；
3. 常量在编译阶段会存入调用类的常量池中，本质上并没有直接引用到定义常量的类，因为不会触发定义常量的类初始化；

![&#x7C7B;&#x52A0;&#x8F7D;&#x6B65;&#x9AA4;](../.gitbook/assets/image%20%284%29.png)

* 加载
  1. 通过类的全限定名称获取定义此类的二进制流；
  2. 将二进制流静态结构转化为方法区的运行时数据结构；
  3. 在内存中生成一个代表这个类的java.lang.Class对象。对于HotSpot虚拟机，Class对象是存放在方法区里的；
* 验证
  1. **文件格式验证**：对检查格式、版本；
  2. **元数据验证**：对字节码进行语义分析；
  3. **字节码验证**；
  4. **符号引用验证**；
* 准备
  1. 正式为类变量分配内存，内存在方法区中进行分配，类变量是指static修饰的变量；
  2. 设置类变量的初始值，这个初始值通常情况下是数据类型的零值，如：public static int value = 123；初始值是0，不是123，赋值123动作会在初始化阶段才会执行；
* 解析
  * 将Class文件的常量池内的符号引用替换为直接引用；
* 初始化
  * 真正开始执行类中定义的Java程序代码，初始化过程就是执行类构造器&lt;clinit&gt;\(\)方法的过程；
    1. &lt;clinit&gt;\(\)方法会自动收集类中所有类变量（static）的赋值动作和静态代码块，编译器收集顺序由代码出现顺序决定；
    2. &lt;clinit&gt;\(\)方法与类实例构造函数不同，虚拟机会保证父类的&lt;clinit&gt;\(\)执行完毕后再执行子类的&lt;clinit&gt;\(\)方法；









负责加载class文件，class文件在文件开头有特定的文件标示，并且ClassLoader只负责class文件的加载，至于它 是否可以运行，则由Execution Engine决定

![](../.gitbook/assets/image%20%285%29.png)

1. **根类加载器（Bootstrap ClassLoader）：**这个类加载器负责放在\lib目录中的，或者被-Xbootclasspath参数所指定的路径 中的，并且是虚拟机识别的类库。用户无法直接使用。\(就相当于是为什么我们可以通过new创建出来对象\)，比如：String,System这类
2. **拓展类加载器（Extension ClassLoader）**：这个类加载器由sun.misc.Launcher$AppClassLoader实现。它负责\lib\ext目录 中的，或者被java.ext.dirs系统变量所指定的路径中的所有类库。用户可以直接使用。 加载JRE的拓展类库
3. **系统类加载器（System ClassLoader）/用户类加载器（Application ClassLoader）：**这个类由sun.misc.Launcher$AppClassLoader实现。是ClassLoader中 getSystemClassLoader\(\)方法的返回值。它负责用户路径（ClassPath）所指定的类库。用户可以直接使用。如果 用户没有自己定义类加载器，默认使用这个。
4. 自定义加载器：用户自己定义的类加载器。\(只有做框架才会用到\)

### JVM双亲委派机制和沙箱机制

#### 双亲委托: 

一个类加载器查找class和resource时，是通过“委托模式”进行的，它首先判断这个class是不是已经加载成功，如果 没有的话它并不是自己进行查找，而是先通过父加载器，然后递归下去，直到Bootstrap ClassLoader，如果 Bootstrap classloader找到了，直接返回，如果没有找到，则一级一级返回，最后到达自身去查找这些对象。这种 机制就叫做双亲委托。

![](../.gitbook/assets/image%20%287%29.png)

1. 一个AppClassLoader查找资源时，先看看缓存是否有，缓存有从缓存中获取，否则委托给父加载器。 
2. 递归，重复第1部的操作。 
3. 如果ExtClassLoader也没有加载过，则由Bootstrap ClassLoader出面，它首先查找缓存，如果没有找到的话， 就去找自己的规定的路径下，也就是sun.mic.boot.class下面的路径。找到就返回，没有找到，让子加载器自己去 找。
4. Bootstrap ClassLoader如果没有查找成功，则ExtClassLoader自己在java.ext.dirs路径中去查找，查找成功就返 回，查找不成功，再向下让子加载器找。 
5. ExtClassLoader查找不成功，AppClassLoader就自己查找，在java.class.path路径下查找。找到就返回。如果没 有找到就让子类找，如果没有子类会怎么样？抛出各种异常。

#### 沙箱机制: 

沙箱机制也就是双亲委派模型的安全机制

在写代码中,系统会提供对应的系统类给我们使用,比如String字符串,那么我们来进行一个模拟操作 自定义一个java.lang.String类

```java
package java.lang;
/**
* 创建一个String类包名和类名完全和系统中的String一致
*/
public final class String {
    public static void main(String[] args) {
        System.out.println("this is my String");
    }
}
```

当我们执行的时候回发现: 

**错误:** 在类 java.lang.String 中找不到 main 方法, 请将 main 方法定义为: 

```java
public static void main(String[] args)
```

否则 JavaFX 应用程序类必须扩展javafx.application.Application 代码中是明明有main的为什么会出错就是时双亲委托中的沙箱机制了 自定义的java.lang.String类永远都不会被加载进内存。因为首先是最顶端的类加载器加载系统的java.lang.String 类，最终自定义的类加载器无法加载java.lang.String类 这样一来的好处就是可以保证不被恶意的修改系统中原有的类

了解:双亲委托和沙箱机制不是绝对安全的因为可以写自定义ClassLoader,自定义的类加载器里面强制加载自定义 的 java.lang.String 类，不去通过调用父加载器 ,完成类的加载. 

当ClassLoader加载成功后,Execution Engine执行引擎负责解释命令，提交操作系统执行



Java类随着加载它的类加载器一起具备了一种带有优先级的层次关系。比如，Java中的Object类，它存放在rt.jar之中,无论哪一个类加载器要加载这个类，最终都是委派给处于模型最顶端的启动类加载器进行加载，因此Object在各种类加载环境中都是同一个类。如果不采用双亲委派模型，那么由各个类加载器自己取加载的话，那么系统中会存在多种不同的Object类。

