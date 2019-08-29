# Synchronized

在JDK5之前Java解决并发问题需要用Synchronized\(以下简称Sync\)解决，JDK5中引入了轻量级锁类Lock，但是在JDK1.6之后的版本又优化了Sync的流程，使其效率更高

## Sync的特性

1. **原子性**：确保线程互斥的访问同步代码；
2. **可见性**：保证共享变量的修改能够及时可见，其实是通过Java内存模型中的 **“对一个变量unlock操作之前，必须要同步到主内存中；如果对一个变量进行lock操作，则将会清空工作内存中此变量的值，在执行引擎使用此变量前，需要重新从主内存中load操作或assign操作初始化变量值”** 来保证的；
3. **有序性**：有效解决重排序问题，即 **“一个unlock操作先行发生\(happen-before\)于后面对同一个锁的lock操作”**；

## Sync的使用

Sync使用时需指定一个对象，在HotSpot JVM\(本文默认JVM为HotSpot JVM\)中作为Object Monitor\(监视器\)，监视器对象例子：this，"string"，Class对象，某参数对象

Sync内置锁 是一种 对象锁（锁的是对象而非引用变量），作用粒度是对象 ，可以用来实现对临界资源的同步互斥访问 ，是可重入的。其可重入最大的作用是避免死锁，如：子类同步方法调用了父类同步方法，如没有可重入的特性，则会发生死锁；

## Sync同步实现

Sync是在软件层面依赖JVM进行同步，而Lock类是在硬件层面依赖特殊的CPU指令进行同步

### 同步代码块

```java
package com.test;
public class SynchronizedDemo {
    public void method() {
        synchronized (this) {
            System.out.println("Method 1 start");
        }
    }
}
```

{% code-tabs %}
{% code-tabs-item title="反编译结果" %}
```elixir
Compiled from "SynchronizedDemo.java"
public class com.test.SynchronizedDemo {
  public com.test.SynchronizedDemo();
    Code:
      0: aload_0
      1: invokespecial #1       // Method java/lang/Object."<init>":()V
      4: return
  public void method();
    Code:
      0:  aload_0
      1:  dup
      2:  astore_1
      3:  monitorenter
      4:  getstatic      #2     // Field java/lang/System.out:Ljava/io/PrintStream;
      7:  Idc            #3     // String Method 1 start
      9:  invokevirtual  #4     // Method java/io/PrintStream.println:(Ljava/lang/String)V
      12: aload_1
      13: monitorexit
      14: goto           22
      17: astore_2
      18: aload_1
      19: monitorexit
      20: aload_2
      21: athrow
      22: return
```
{% endcode-tabs-item %}
{% endcode-tabs %}

1. **monitorenter\(3\)**：每个对象都是一个监视器锁（monitor）。当monitor被占用时就会处于锁定状态，线程执行monitorenter指令时尝试获取monitor的所有权，过程如下：
2. **monitorexit\(13\)**：执行monitorexit的线程必须是objectref所对应的monitor的所有者。**指令执行时，monitor的进入数减1，如果减1后进入数为0，那线程退出monitor，不再是这个monitor的所有者**。其他被这个monitor阻塞的线程可以尝试去获取这个 monitor 的所有权。

> 1.  **如果monitor的进入数为0**，则该线程进入monitor，然后将进入数设置为1，该线程即为monitor的所有者；
> 2.  **如果线程已经占有该monitor**，只是重新进入，则进入monitor的进入数加1；
> 3.  **如果其他线程已经占用了monitor**，则该线程进入阻塞状态，直到monitor的进入数为0，再重新尝试获取monitor的所有权；

> **monitorexit指令出现了两次，第1次为同步正常退出释放锁；第2次为发生异步退出释放锁**；

通过上面两段描述，我们应该能很清楚的看出Synchronized的实现原理，**Synchronized的语义底层是通过一个monitor的对象来完成，其实wait/notify等方法也依赖于monitor对象**，这就是为什么只有在同步的块或者方法中才能调用wait/notify等方法，**否则会抛出java.lang.IllegalMonitorStateException的异常的原因**。

### 同步方法

```java
package com.test;
public class SynchronizedMethod {
    public synchronized void method() {
        System.out.println("Hello World!");
    }
}
```

{% code-tabs %}
{% code-tabs-item title="反编译结果" %}
```elixir
public synchronized void method();
  descriptor: ()V
  flags: ACC_PUBLIC, ACC_SYHCHRONIZED
  Code:
    stack=2, locals=1, args_size=l
      0: getstatic      #2      // Field java/lang/System.out:Ljava/io/PrintStream;
      3: ldc            #3      // String Hello World!
      5: invokevirtual  #4      // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      8: return
    LineNumberTable:
      line 5: 0
      line 6: 8
      LocalVariableTable:
       Start  Length Slot   Name   Signature
           0      9     0   this   Lcom/test/SynchronizedMethod;
```
{% endcode-tabs-item %}
{% endcode-tabs %}

从编译的结果来看，方法的同步并没有通过指令 **`monitorenter`** 和 **`monitorexit`** 来完成（理论上其实也可以通过这两条指令来实现），不过相对于普通方法，其常量池中多了 **`ACC_SYNCHRONIZED`** 标示符。**JVM就是根据该标示符来实现方法的同步的**：

> 当方法调用时，**调用指令将会检查方法的 ACC\_SYNCHRONIZED 访问标志是否被设置**，如果设置了，**执行线程将先获取monitor**，获取成功之后才能执行方法体，**方法执行完后再释放monitor**。在方法执行期间，其他任何线程都无法再获得同一个monitor对象。

两种同步方式本质上没有区别，只是方法的同步是一种隐式的方式来实现，无需通过字节码来完成。**两个指令的执行是JVM通过调用操作系统的互斥原语mutex来实现，被阻塞的线程会被挂起、等待重新调度**，会导致“用户态和内核态”两个态之间来回切换，对性能有较大影响。





































