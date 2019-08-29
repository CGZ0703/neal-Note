# 反射

### Class

1. 除了int等基本类型外，Java的其他类型全部都是class（包括interface）。仔细思考，我们可以得出结论：class（包括interface）的本质是数据类型（Type）。无继承关系的数据类型无法赋值。
2. 而class是由JVM在执行过程中动态加载的。JVM在第一次读取到一种class类型时，将其加载进内存。每加载一种class，JVM就为其创建一个Class类型的实例，并关联起来。注意：这里的Class类型是一个名叫Class的class。
3. 以String类为例，当JVM加载String类时，它首先读取String.class文件到内存，然后，为String类创建一个Class实例并关联起来

| Class Instance | ==&gt;  String的Class类 |
| :--- | :--- |
| name = "java.lang.String" |  |
| package = "java.lang" |  |
| super = "java.lang.Object" |  |
| interface = CharSequence... |  |
| field = value\[\],hash,... |  |
| method = indexOf\(\)... |  |

1. 这个Class实例是JVM内部创建的，如果我们查看JDK源码，可以发现Class类的构造方法是private，只有JVM能创建Class实例，我们自己的Java程序是无法创建Class实例的。所以，JVM持有的每个Class实例都指向一个数据类型（class或interface）
2. 一个Class实例包含了该class的所有完整信息\(String\)：
3. 由于JVM为每个加载的class创建了对应的Class实例，并在实例中保存了该class的所有信息，包括类名、包名、父类、实现的接口、所有方法、字段等，因此，如果获取了某个Class实例，我们就可以通过这个Class实例获取到该实例对应的class的所有信息。这种通过Class实例获取class信息的方法称为反射（Reflection）。

### 通过字节码文件获取对象

* 动态获取类的字节码文件，并对其成员进行抽象
* 整体含义：想通过字节码文件直接创建对象
* 过程
  1. 获取字节码文件对象
  2. 通过字节码文件对象获取对应的实例对象
  3. 给属性赋值（通过从属性中提取出来的类--Field
  4. 调用方法（通过从方法中提取出来的类--Method
* 获取字节码文件对象
  1. 通过Object提供的getClass\(\)方法
  2. 通过每种数据类型都有的一个class属性
  3. Class类提供的一个静态方法forName\("Package.ClassName"\)
* 通过字节码文件对象获取对应的实例对象
  1. 普通方式 new
  2. 通过反射创建普通对象
  3. 通过无参的构造方法创建实例对象
     * Object object = cls.newInstance\(\);
     * 相当于在newInstance方法的内部调用无参的构造方法
  4. 通过有参的构造方法创建实例对象
     * 先要得到有参的构造方法，这里要写参数的字节码文件对象形式，所有的类型都有字节码文件对象
     * Constructor constructor = cls.getConstructor\(String.class, int.class\);
     * Object object = constructor.newInstance\("string", 0\);
* 调用属性
  1. fieldName属性必须的public的
     * Field field = cls.getField\("fieldName"\);
  2. 如果field属性是私有的，可以用getDeclaredField忽略权限
     * Field field = cls.getDeclaredField\("FieldName"\);
     * field.setAccessible\(true\);
  3. 赋值 第一个参数为具体的对象，第二个参数为需要赋值的属性
     * field.set\(object, "property"\);
  4. 
* 调用方法
  1. 调用非静态无参
     * 获取实例对象
     * 通过反射得到方法
       * Method method = cls.getMethod\("methodName"\);
     * 调用方法，通过调用invoke方法实现
       * method.invoke\(object\);
  2. 调用非静态有参
     * 获取实例对象
     * 通过反射得到方法,参数中放入参数
       * Method method = cls.getMethod\("methodName", String.class...\);
     * 调用方法，通过调用invoke方法实现
       * method.invoke\(object, "string"\);
  3. 调用静态有参
     * 通过反射得到方法
       * Method method = cls.getMethod\("methodName", int.class\);
     * 调用方法，通过invoke方法实现
       * method.invoke\(null, 0\);

## 动态代理

### 静态代理/动态代理

* 作用：可以实现代理，根据OCP\(对扩展开放，对修改关闭\)的原则，在不改变原来类的基础上，给这个类增加额外的功能
* 缺点：代理对象要保证跟目标对象实现同样的接口，在维护的时候两个对象都要维护，而且代理对象实现的接口是死的，这时如果要给想实现不同功能的多个目标对象添加代理对象的话，要添加很多个类

### 动态代理

1. Agant实现InvocationHandler
   * 这个方法在调用接口方法的时候，会被自动调用
     1. 参数一：代理对象的引用
     2. 参数二：目标对象的方法
     3. 参数三：目标对象的方法参数

        ```java
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {        
         { // TODO }
         Object object = method.invoke(person, args);
         { // TODO }
         return object;
        }
        ```
   * 动态生成代理对象的方法，通过JDK内置的java.lang.reflect.Proxy动态代理类完成代理对象的创建
     1. 参数一：代表类加载器，代理类的类加载器要与目标类的类加载器一直，类加载器用来装载内存中字节码文件
     2. 参数二：代理类与目标类实现的接口必须有相同的，即指定给代理类的接口，目标类必须实现了
     3. 参数三：代理类的构造方法生成的对象---注意：指定给构造方法的参数要用Object

        ```java
        MyInterface object = (MyInterface)Proxy.newProxyInstance(myInterface.getClass().getClassLoader(), new Class[] {MyInterface.class}, new Agent(myInterface));
        object.myMethod();
        ```
2. 优化：直接使用InvocationHandler创建匿名内部类\(Lambda\)干活，不再需要Agent类 

