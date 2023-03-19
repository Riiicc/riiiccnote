# 待办、
- ~~破坏双亲委派模型的理解 `jvm第三版7.4.3`~~
- 方法重写的本质 中 添加动态分派(重写)和静态分派(重载) 分类 `JVM第三版8.3.2`(这块基本要重新规划)
- 对象的内存布局不详细   
- 类加载机制整体重构，需根据第二章和第九章合并，同时参看`JVM第三版 第七章`

# 一、JVM整体结构，执行流程，生命周期
## 整体结构
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jvm3.png)

## java代码执行流程
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jvm2.png)

### 两种指令集架构
- 基于栈式架构
    - 大部分`零地址指令`，**指令集小，指令多，跨平台**，性能比寄存器架构略低
- 基于PC寄存器
    - 对硬件依赖大，**可移植性差，耦合高，性能高**，
    - 基于寄存器架构的指令集往往都是`一地址指令，二地址指令，三地址指令`为主

?> 由于跨平台性的设计，Java的执行都是根据栈来设计的，**不同CPU架构不同，所以不能设计为基于寄存器的**

## JVM生命周期
- 启动，通过`引导类加载器 bootstrap class loader`创建初始类来完成
- 执行
  - 执行一个java程序的时候，真正执行的是一个叫做Java虚拟机的进程
- 结束
    - 执行完成
    - 异常，错误
    - 操作系统错误，导致java虚拟机终止
    - 调用`Runtime/System.exit()` 或`Runtime`类的`halt`方法,halt方法最终调用一个native方法
  
## 几种虚拟机 
### 虚拟机始祖：Sun Classic/Exact VM
> Sun发布JDK 1.0，Java语言首次拥有了商用的正式运行环境，这个JDK中所带的虚拟机就是Classic VM
> Exact VM因它使用准确式内存管理（Exact Memory Management，也可以叫Non-Con-servative/Accurate Memory Management）而得名

### HotSpot VM
> 2006年，Sun陆续将SunJDK的各个部分在GPLv2协议下开放了源码，形成了Open-JDK项目，其中
> 当然也包括HotSpot虚拟机。HotSpot从此成为Sun/OracleJDK和OpenJDK两个实现极度接近的JDK项目
> 的共同虚拟机。Oracle收购Sun以后，建立了HotRockit项目来把原来BEA JRockit中的优秀特性融合到
> HotSpot之中。到了2014年的JDK 8时期，里面的HotSpot就已是两者融合的结果，HotSpot在这个过程
> 里移除掉**永久代**，吸收了**JRockit**的**Java Mission Control**监控工具等功能。


### Mobile/Embedded VM
> 面对移动和嵌入式市场，也有专门的Java虚拟机产品

### BEA JRockit/IBM J9 VM
> BEA System公司的JRockit与IBM公司的IBM J9

### BEA Liquid VM/Azul VM
> **Liquid VM**也被称为JRockit VE（Virtual Edition，VE），它是BEA公司开发的可以直接运行在自家Hypervisor系统上的JRockit虚拟机的虚拟化版本
> **Azul VM**是Azul Systems公司在HotSpot基础上进行大量改进，运行于Azul Systems公司的专有硬件Vega系统上的Java虚拟机

### Apache Harmony/Google Android Dalvik VM
> Apache Harmony是一个Apache软件基金会旗下以Apache License协议开源的实际兼容于JDK 5和JDK 6的Java程序运行平台，
> 它含有自己的虚拟机和Java类库API，用户可以在上面运行Eclipse、Tomcat、Maven等常用的Java程序

### Microsoft JVM、TaoBaoJVM及其他



# 二、虚拟机类加载机制
## 类加载子系统的作用
- 类加载子系统负责冲文件系统或者网络中加载Class文件
- ClassLoader只负责class文件的加载，至于是否可以运行由执行引擎`ExecutionEngine`决定 
- 加载的类信息存在于方法区

## 类加载过程  
> 一个类型从被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期将会经历**加载，验证，准备，解析，初始化，使用，卸载**七个阶段
> 其中 验证，准备，解析 三个部分统称为连接 

- 加载`Loading`
- 验证`Verification`
- 准备`Preparation`
- 解析`Resolution`
- 初始化`Initialization`
- 使用`Using`
- 卸载`Unloading`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jvm4.png)

> 加载,验证,准备,初始化和卸载这五个阶段的顺序是确定的,而解析阶段则不一定,在java语言运行时绑定(也成为动态绑定,晚期绑定)可以在初始化阶段之后再开始 

### 类的主动引用
- 只有如下**六种**情况必须立即对类进行初始化:
  - 遇到`new` `getstatic` `putstatic` `invokstatic` 这四条字节码指令时,如果类型没有进行过初始化,则需要先触发其初始化阶段，能够生成这四条指的典型场景有：
    - 使用new关键字实例化对象
    - 读取或者设置类静态字段（final 修饰的、已在编译器将结果放入常量池除外）
    - 调用类的静态方法
  - 使用`java.lang.reflect` 包的方法对类进行反射调用的时候，如果类型没有进行过初始化，则需要先触发其初始化 
  - 初始化一个类时，如果其父类还没有初始化，则需要先初始化父类。
  - 虚拟机启动时，用于需要指定一个包含 main() 方法的主类，虚拟机会先初始化这个主类。
  - 当使用 JDK 1.7 的动态语言支持时，如果一个 java.lang.invoke.MethodHandle 实例最后的解析结果为 REF_getStatic、REF_putStatic、REF_invokeStatic 的方法句柄，并且这个方法句柄所对应的类还没初始化，则需要先触发其初始化。 
  - 当一个接口中定义了JDK 8新加入的默认方法（被default关键字修饰的接口方法）时，如果有这个接口的实现类发生了初始化，那该接口要在其之前被初始化。（**第三版新增**）

- 这六种场景中的行为称为对一个类型进行**主动引用**。除此之外，**所有**引用类型的方式都不会触发初始化，称为**被动引用**。

### 类的被动引用案例
- 子类来引用父类中定义的静态字段，只会触发父类的初始化而不会触发子类的初始化 

```java
/**
 * 通过子类引用父类的静态字段，不会导致子类初始化。
 */
class SuperClass {
    static {
        System.out.println("SuperClass init!");
    }

    public static int value = 123;
}

class SubClass extends SuperClass {
    static {
        System.out.println("SubClass init!");
    }
}
/**
* 非主动使用类字段演示
**/
public class NotInitialization {

    public static void main(String[] args) {
        System.out.println(SubClass.value);
        // SuperClass init!
    }

}
```

> 只会输出“SuperClass init！”，而不会输出“SubClass init！”。
> 对于静态字段，只有直接定义这个字段的类才会被初始化，因此通过其子类来引用父类中定义的静态字段，只会触发父类的初始化而不会触发子类的初始化。

- 通过数组定义来引用类，不会触发此类的初始化

```java
package org.fenixsoft.classloading;
/**
* 被动使用类字段演示二：
* 通过数组定义来引用类，不会触发此类的初始化
**/
public class NotInitialization {
public static void main(String[] args) {
    SuperClass[] sca = new SuperClass[10];
}
}
```

> 这段代码没有触发类`org.fenixsoft.classloading.SuperClass`的初始化阶段。
> 但是这段代码里面触发了另一个名为`[Lorg.fenixsoft.classloading.SuperClass`的类的初始化阶段，它是一个由虚拟机自动生成的、直接继承于`java.lang.Object`的子类，创建动作由字节码指令`newarray`触发

- 常量在编译阶段会存入调用类的常量池中，本质上没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化  

```java
package org.fenixsoft.classloading;
/**
* 被动使用类字段演示三：
* 常量在编译阶段会存入调用类的常量池中，本质上没有直接引用到定义常量的类，因此不会触发定义常量的
类的初始化
**/
public class ConstClass {
    static {
        System.out.println("ConstClass init!");
    }
    public static final String HELLOWORLD = "hello world";

}
/**
* 非主动使用类字段演示
**/
public class NotInitialization {
    public static void main(String[] args) {
        System.out.println(ConstClass.HELLOWORLD);
    }
}
```

- **注意** 接口的加载过程和类加载过程略有不同，接口也有初始化过程，编译器会为接口生成`<clinit>()` 类构造器，用于初始化接口中所定义的成员变量，接口初始化时，并不要求其父类接口全部完成初始化，只有在**真正用到父类接口**的时候（如引用接口中定义的常量）才会初始化。


## 类加载过程
加载、验证、准备、解析和初始化这五个阶段所执行的具体动作 

### 加载 Loading
加载阶段，Java虚拟机需要完成以下三件事情：
- 通过一个类的全限定名（全类名）来获取定义此类的二进制字节流
- 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
- 在内存中生成一个代表整个类的`java.lang.Class`对象，作为方法区整个类的各种数据的访问入口

#### 获取二进制字节流
对于 Class 文件，虚拟机没有指明要从哪里获取、怎样获取。除了直接从编译好的 .class 文件中读取，还有以下几种方式：

- 从 zip 包中读取，如 jar、war 等；
- 从网络中获取，如 Applet；
- 通过动态代理技术生成代理类的二进制字节流；
- 由 JSP 文件生成对应的 Class 类；
- 从数据库中读取，如 有些中间件服务器可以选择把程序安装到数据库中来完成程序代码在集群间的分发

#### 数组类型和非数组类型的加载区别
- **非数组类型**的加载（获取二进制字节流）阶段，既可以使用虚拟机内置的引导类加载器来完成，也可以使用自定义类加载器（重写一个类加载器的findClass()或loadClass()方法）
- **数组类型**不通过类加载器创建，由Java虚拟机直接在内存中动态构造出来,再由类加载器创建数组中的元素类   

#### 数组类型加载详解 
详见 7.3.1 加载   
- 如果数组的组件类型是引用类型，那就递归采用通用加载过程去加载这个组件类型，数组将会被标识在加载该组件类型的类加载器的类名称空间上
- 如果数组的组件类型不是引用类型（如 `int[]`），java虚拟机将会把数组标记为与引导类加载器关联

#### 其他
> **加载阶段**与**连接阶段**的部分动作（如 字节码格式验证动作）是交叉进行的，加载阶段尚未完成，连接阶段可能已经开始，但这些加载阶段的动作仍然是连接阶段的一部分，这两个阶段的**开始时间**仍然保持着固定的先后顺序


### 验证 Verification
> **验证是连接阶段的第一步**，这一阶段的目的是确保Class文件的字节流中包含的信息符合《Java虚拟机规范》的全部约束要求，保证这些信息被当作代码运行后不会危害虚拟机自身的安全。

1. 文件格式验证  
   - 是否以魔数`0xCAFEBABE`开头
   - 主次版本号是否在当前Java虚拟机接受范围之内
   - 常量池的常量中是否有不被支持的常量类型
   - 指向常量的各种索引值中是否有指向不存在的常量或者不符合类型的常量
   - 。。。

2. 元数据验证
对字节码描述的信息进行**语义分析**

- 类是否有父类
- 类的父类是否继承了不允许被继承的类
- 如果这个类不是抽象类，是否实现了其父类或者接口中要求实现的所有方法
- 类的字段、方法是否与父类产生矛盾（例如覆盖了父类的final字段，或者出现不符合规则的方法重载，例如方法参数都一致，但返回值类型却不同等）

3. 字节码验证

第三阶段是整个验证过程中最复杂的一个阶段，主要目的是通过数据流分析和控制流分析，确定程序语义是合法的、符合逻辑的。

4. 符号引用验证

符号引用验证可以看作是对类自身以外（常量池中的各种符号引用）的各类信息进行匹配性校验，通俗来说就是，该类是否缺少或者被禁止访问它依赖的某些外部类、方法、字段等资源

> 如果无法通过符号引用验证，Java虚拟机将会抛出一个java.lang.IncompatibleClassChangeError的子类异常，典型的如：java.lang.IllegalAccessError、java.lang.NoSuchFieldError、java.lang.NoSuchMethodError等。


### 准备 Preparation
> 准备阶段是正式为类中定义的变量（即静态变量，被static修饰的变量）分配内存并设置类变量初始值的阶段

> 这时候进行内存分配的仅包括**类变量**，而不包括**实例变量**，实例变量将会在对象实例化时随着对象一起分配在Java堆中。
> 其次是这里所说的初始值“通常情况”下是数据类型的**零值**，假设一个类变量的定义为：
```java
    public static int value = 123;
```
> 变量value在准备阶段过后的**初始值为0而不是123**，因为这时尚未开始执行任何Java方法，而把value赋值为123的putstatic指令是程序被编译后，存放于类构造器<clinit>()方法之中，所以把value赋值为123的动作要到类的初始化阶段才会被执行。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jvm5.png) 

**关于初始值的特殊情况：**如果类字段的字段属性表中存在`ConstantValue`属性，那在准备阶段变量值就会被初始化为`ConstantValue`属性所指定的初始值，下面类变量value的初始值为123  

```java
    //编译时Javac将会为value生成ConstantValue属性，在准备阶段虚拟机就会根据ConstantValue的设置将value赋值为123。
    public static final int value = 123;
```

> 注：`ConstantValue` 是指 **同时使用final 、static来修饰**的变量（常量），并且这个变量的数据类型是**基本类型或者String类型**，就生成ConstantValue属性来进行初始化
```java
public class TestKM {
    public static int value1 = 1;
    public final static int value2 = 1;
    public static String str1 = "1";
    public final static String str2 = "1";

}
```

上面的代码编译后可以看到`value2` 和 `str2`有`ConstantValue`属性  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jvm6.png)

### 解析Resolution
解析阶段是Java虚拟机将常量池内的符号引用替换为直接引用的过程  
- 符号引用（Symbolic References）：符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可。
- 直接引用（Direct References）：直接引用是可以直接指向目标的指针、相对偏移量或者是一个能间接定位到目标的句柄。
> 解析动作主要针对**类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符**这7类符号引用进行，
> 分别对应于常量池的`CONSTANT_Class_info`、`CON-STANT_Fieldref_info`、`CONSTANT_Methodref_info`、`CONSTANT_InterfaceMethodref_info`、`CONSTANT_MethodType_info`、`CONSTANT_MethodHandle_info`、`CONSTANT_Dyna-mic_info`和`CONSTANT_InvokeDynamic_info` 8种常量类型 



### 初始化Initialization 
初始化阶段就是执行类构造器`<clinit>()`方法的过程  

- `<clinit>()`方法是由编译器自动收集类中的**所有类变量(静态变量)的赋值动作和静态语句块**（`static{}`块）中的语句合并产生的，
- 编译器收集的顺序是由语句在源文件中出现的顺序决定的，静态语句块中只能访问到定义在静态语句块之前的变量，定义在它之后的变量，在前面的静态语句块可以赋值，但是不能访问  

```java
public class Test{
    static{
        i = 0; // 给变量复制可以正常编译通过
        System.out.print(i); // 这句编译器会提示“非法向前引用”
    }
    static int i = 1;
}
```
- `<clinit>()`方法与类的构造函数（即在虚拟机视角中的实例构造器`<init>()`方法）不同，它不需要显式地调用父类构造器，
-  Java虚拟机会保证在子类的`<clinit>()`方法执行前，父类的`<clinit>()`方法已经执行完毕。因此在Java虚拟机中第一个被执行的`<clinit>()`方法的类型肯定是`java.lang.Object` 
-  由于父类的`<clinit>()`方法先执行，也就意味着**父类中定义的静态语句块要优先于子类的变量赋值操作**  

```java
static class Parent {
    public static int A = 1;
    static {
        A = 2;
    }
}

static class Sub extends Parent {
    public static int B = A;
}

public static void main(String[] args) {
    //证明需要加载完父类的clinit，才进行子类加载
    System.out.println(Sub.B); // 输出 2
}
```

- `<clinit>()`方法对于类或接口来说并不是必需的，如果一个类中没有静态语句块，也没有对变量的赋值操作，那么编译器可以不为这个类生成`<clinit>()`方法
-  接口中不能使用静态代码块，但接口也需要通过 `<clinit>()` 方法为接口中定义的静态成员变量显式初始化。但接口与类不同，接口的 `<clinit>()` 方法不需要先执行父类的 `<clinit>()` 方法，只有当父接口中定义的变量使用时，父接口才会初始化

- 虚拟机会保证一个类的 `<clinit>()` 方法在多线程环境中被正确加锁、同步。如果多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的 `<clinit>()` 方法。


## 类加载器 Class Loader 
对于任意一个类，都必须由**加载它的类加载器**和**类本身**一起确立其在java虚拟机中的唯一性，**比较两个类是否相同，只有在这两个类是由同一个类加载器加载的前提下才有意义**，两个类来源于同一个Class文件，被同一个Java虚拟机加载，**只要加载它们的类加载器不同，那这两个类就必定不相等**

?> 这里所指的“相等”，包括代表类的Class对象的equals()方法、isAssignableFrom()方法、isInstance()方法的返回结果，也包括了使用instanceof关键字做对象所属关系判定等各种情况

JVM支持两种类加载器，一种是`启动类加载器 (BootstrapClassLoader)`,另一种是剩余的所有类加载器(自定义类加载器)`扩展类加载器...`

### 启动（引导）类加载器 Bootstrap Class Loader
- 使用C/C++语言实现，嵌套在JVM内部
- 负责加载存放在`<JAVA_HOME>\lib`目录，或者被`-Xbootclasspath`参数所指定的路径中存放的，而且是Java虚拟机能够识别的（按照文件名识别，如rt.jar、tools.jar，名字不符合的类库即使放在lib目录中也不会被加载）类库加载到虚拟机的内存中  
- 不是继承自`java.lang.ClassLoader`,没有父加载器
  

### 扩展类加载器Extension Class Loader
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/扩展类加载器继承关系.png)

这个类加载器是在类`sun.misc.Launcher$ExtClassLoader`中以Java代码的形式实现的。它负责加载`<JAVA_HOME>\lib\ext`目录中，或者被`java.ext.dirs`系统变量所指定的路径中所有的类库。

### 应用程序类加载器 Application Class Loader
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/系统类加载器继承关系.png)

由于这个类加载器是 `ClassLoader` 中的 `getSystemClassLoader()` 方法的返回值，所以一般也称它为“系统类加载器”。它负责加载用户类路径（classpath）上所指定的类库，开发者可以直接使用这个类加载器，如果应用程序没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。    

```java
    ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
    System.out.println(systemClassLoader);//sun.misc.Launcher$AppClassLoader@18b4aac2
    
    ClassLoader parent = systemClassLoader.getParent();
    System.out.println(parent);//sun.misc.Launcher$ExtClassLoader@1540e19d

    ClassLoader classLoader = Test1.class.getClassLoader();//sun.misc.Launcher$AppClassLoader@18b4aac2 注意和上面的相同
    ClassLoader classLoader1 = String.class.getClassLoader();//null 无法获取引导类加载器
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jvm7.png ':size=30%')

> 上图中展示的各种类加载器之间的层次关系被称为类加载器的“双亲委派模型（Parents DelegationModel）”。双亲委派模型要求除了顶层的启动类加载器外，**其余的类加载器**都应有自己的父类加载器。**类加载器的父子关系和代码继承关系无关**  

### 双亲委派模型
类加载器的双亲委派模型在JDK1.2时期被引入

> 如果一个类加载器收到了类加载请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，父类再去委派父类，直到顶层的启动类加载器，只有当父类加载器反馈自己无法完成这个加载请求（范围内找不到对应的类），子类加载器才会尝试自己去加载。

用处/优势：  
- 避免类的重复加载
- 保护程序安全，防止核心API被篡改（如：自定义类`java.lang.String`）  

像 `java.lang.Object` 这些存放在 `rt.jar` 中的类，无论使用哪个类加载器加载，最终都会委派给最顶端的启动类加载器加载，从而使得不同加载器加载的 `Object` 类都是同一个。  
相反，如果没有使用双亲委派模型，由各个类加载器自行去加载的话，如果用户自己编写了一个称为 `java.lang.Object` 的类，并放在 `classpath` 下，那么系统将会出现多个不同的 `Object` 类，Java 类型体系中最基础的行为也就无法保证。

**ClassLoader.loadClass实现双亲委派模型的源码如下**

```java
protected Class<?> loadClass(String name, boolean resolve)throws ClassNotFoundException{
        synchronized (getClassLoadingLock(name)) {
            // 首先，检查请求的类是否已经被加载过了
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // 如果父类加载器抛出ClassNotFoundException
                    // 说明父类加载器无法完成加载请求
                }

                if (c == null) {
                    // 在父类加载器无法加载时
                    // 再调用本身的findClass方法来进行类加载
                    long t1 = System.nanoTime();
                    c = findClass(name);

                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

> **过程说明：**先检查请求加载的类型是否已经被加载过，若没有则调用父加载器的`loadClass()`方法，若父加载器为空则默认使用启动类加载器作为父加载器。假如父类加载器加载失败，抛出`ClassNotFoundException`异常的话，才调用自己的`findClass()`方法尝试进行加载

### 自定义类加载器
- 继承抽象类`java.lang.ClassLoader`，重写`loadClass()`方法或者`findClass()`方法
- 继承URLClassLoader类 




# 三、运行时数据区
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jvm1.png) 

> Java虚拟机定义了若干种程序运行期间会使用到的运行时数据区，其中一些会随着虚拟机的启动而创建(堆，方法区)`线程共享`,另外一些则与线程的生命周期对应(虚拟机栈，本地方法栈，PC寄存器)`线程隔离`

## JVM线程
线程是一个程序里的运行单元，JVM允许一个应用有多个线程并行的执行    
在Hotspot JVM里，每个线程都与操作系统的本地线程直接映射，当一个java线程准备好执行以后，此时一个操作系统的本地线程也同时创建，Java线程执行终止后，本地线程也会回收 

JVM系统线程(不包括main线程以及由main线程创建的线程)  
- 虚拟机线程
- 周期任务线程
- GC线程
- 编译线程
- 信号调度线程

## 程序计数器（PC寄存器）
`程序计数器（Program Counter Register）`是一块较小的内存空间，是当前线程正在执行的那条字节码指令的地址，若当前线程正在执行的是一个本地方法(Native Method)，那么此时程序计数器为`Undefined`  

Java虚拟机的多线程是通过线程轮流切换、分配处理器执行时间的方式来实现的，在任何一个确定的时刻，一个处理器（对于多核处理器来说是一个内核）都只会执**行一条线程中的指令**。因此，为了线程切换后能恢复到正确的执行位置，**每条线程都需要有一个独立的程序计数器，各条线程之间计数器互不影响，独立存储**，我们称这类内存区域为“线程私有”的内存。

### 作用
存储指向下一条指令的地址，由执行引擎读取下一条指令  

- 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制。
- 在多线程情况下，程序计数器记录的是当**前线程执行的位置**，从而当线程切换回来时，就知道上次线程执行到哪了。

### 特点
- 是一块较小的内存空间。
- 线程私有，每条线程都有自己的程序计数器。
- 生命周期：随着线程的创建而创建，随着线程的结束而销毁。
- 是唯一个不会出现`OutOfMemoryError`的内存区域

### 实例
一段java代码来理解程序计数器的作用

```java
public class Test2 {
    public static void main(String[] args) {
        int i = 10;
        int j = 20;
        int k = i+j;
    }
}
```

上述代码的class字节码文件反编译后结果为 

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/pc寄存器实例.png)

## Java虚拟机栈
虚拟机栈描述的是Java方法执行的线程内存模型：每个方法被执行的时候，Java虚拟机都会同步创建一个栈帧 `（Stack Frame）`用于**存储局部变量表、操作数栈、动态连接、方法出口等信息**。

每一个方法被调用直至执行完毕的过程，就对应着一个**栈帧在虚拟机栈中从入栈到出栈**的过程。

### 作用
主管java程序的运行，保存方法的局部变量(8种基本类型，对象的引用地址)、部分结果，并参与方法的调用和返回   
- 每个线程都一个虚拟机栈，线程私有，随着线程创建而创建，随着线程的结束而销毁，不存在数据一致问题和同步锁问题  
- 运行速度特别快,仅仅次于 PC寄存器。 
- 栈的操作只有入栈和出栈


### 虚拟机栈中的异常
- 两类异常状况：
  - 如果线程请求的栈深度大于虚拟机所允许的深度（请求分配的栈容量超过虚拟机栈允许的最大容量），将抛出`StackOverflowError`异常；
  - **如果**Java虚拟机栈容量可以动态扩展，当栈扩展时无法申请到足够的内存会抛出`OutOfMemoryError`异常

> 可以使用参数 `-Xss` 来设置线程的最大栈空间，栈的大小直接决定了函数调用的最大可用深度   
> `-Xms -Xmx`是设置堆空间大小的

```java
// java -Xss256k StackDeepTest.java  
// 执行后会报出java.lang.StackOverflowError
// 输出 deep of calling 4471
public class StackDeepTest {
    private static int count = 0;

    public static void recursion(){
        count++;
        recursion();
    }
    public static void main(String[] args) {
        try {
            recursion();
        }catch (Throwable e){
            System.out.println("deep of calling " + count);
            e.printStackTrace();
        }
    }

}
```

> **HotSpot虚拟机的栈容量是不可以动态扩展的**，以前的Classic虚拟机倒是可以。所以在HotSpot虚拟机上是不会由于虚拟机栈无法扩展而导致`OutOfMemoryError`异常——只要线程申请栈空间成功了就不会有OOM，但是如果申请时就失败，仍然是会出现OOM异常的(第三版新增差异)

### 栈的存储单位
- 每个线程都有自己的栈，栈中的数据都是以`栈帧(Stack Frame)`的格式存在,线程中正在执行的每个方法都对应一个栈帧，栈帧随方法的运行开始和结束进行入栈和出栈    
- 在一条活动的线程中，一个时间点上只有一个活动的栈帧，即当前正在执行的方法的栈帧，也称做栈顶栈帧或当前栈帧，对应的方法叫做当前方法，方法所在的类叫做当前类  
- 当前方法中调用了其他方法，对应的新的栈帧会被创建出来，放在栈顶，称为新的当前帧
- 栈帧之间不能相互引用
- 方法正常执行完毕(return)或抛出异常都会导致栈帧出栈，异常未被处理就会抛给上层方法  


### 栈的内部结构
- 局部变量表
- 操作数栈
- 动态链接(指向运行时常量池的方法引用)
- 方法返回地址
- 一些附加信息

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jvm8.png ':size=40%')  

#### 局部变量表 Local Variables Table
局部变量表（Local Variables Table）是一组变量值的存储空间，用于存放`方法参数`和`方法内部定义的局部变量`。

局部变量表存放了编译期可知的各种Java虚拟机基本数据类型`boolean、byte、char、short、int、float、long、double`、`对象引用`（reference类型，它并不等同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或者其他与此对象相关的位置）和returnAddress类型（指向了一条字节码指令的地址）。  

这些数据类型在局部变量表中的存储空间以局部变量槽（Slot）来表示，其中**64位长度的long和double类型的数据会占用两个变量槽**，其余的数据类型(32位)只占用一个。   
局部变量表所需的内存空间在编译期间完成分配，当进入一个方法时，这个方法需要在栈帧中分配多大的局部变量空间是完全确定的，**在方法运行期间不会改变局部变量表的大小（变量槽的数量）**。 

实例代码查看局部变量表

```java
public class Test2 {
    private int xxd = 0;

    public static void main(String[] args) {
        Test2 test2 = new Test2();
        int num = 0;
        test2.test(1);
    }

    public void test(int tt4) {
        int num = 4;
        xxd = xxd + 23;
    }
}
```

将上述文件的class字节码文件进行 `javap -v Test2.class` 可以看到main方法的局部变量表为:入参`args`,局部变量test2，num  
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/局部变量表实例main.png)

**对于 slot 的理解**：

- JVM 虚拟机会为局部变量表中的每个`slot`都分配一个访问索引，通过这个索引即可成功访问到局部变量表中指定的局部变量值。
- 如果当前帧是由`构造方法`或者`实例方法`创建的，那么该对象引用 `this`，会存放在index 为 0 的 slot 处，其余的参数表顺序继续排列。
上面代码示例中`test()`方法的局部变量表如下   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/局部变量表实例方法.png)

- 栈帧中的局部变量表中的槽位是可以重复的，如果一个局部变量过了其作用域，那么其作用域之后申明的新的局部变量就有可能会复用过期局部变量的槽位，从而达到节省资源的目的。

```java
public void test4() {
        int a = 0;
        {
            int b = 0;
            b = a + 1;
        }
        //变量c使用 之前已经销毁的变量b占据的slot的位置 局部变量表的长度为3
        int c = a + 1;
    }
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/slot的重复利用示例.png)  

?> 在栈帧中，与性能调优关系最密切的部分，就是局部变量表，方法执行时，虚拟机使用局部变量表完成方法的传递局部变量表中的变量也是重要的垃圾回收根节点，只要被局部变量表中直接或间接引用的对象都不会被回收

#### 操作数栈 Operand Stack 
1. 操作数栈是一个后入先出的栈结构，在方法执行过程中，根据字节码指令，往栈中写入数据或提取数据(入栈、出栈)    
2. 主要作用是保存计算过程中的中间结果，同时作为计算过程中变量的临时存储空间  

3. 操作数栈在新栈帧创建时是控的，每一个操作数栈会拥有一个明确的栈深度，用于存储数值，最大深度在编译期就定义好。32bit 类型占用一个栈单位深度，64bit 类型占用两个栈单位深度操作数栈。   

4. 操作数栈并非采用访问索引方式进行数据访问，而是只能通过标准的入栈、出栈操作完成一次数据访问 
5. 操作数栈中元素的数据类型必须与字节码指令的序列严格匹配  

代码实例    

```java
public class OperandStackTest {
    public void testAdd(){
        //byte、short、char、boolean：都以int型来保存
        byte i = 15;
        int j = 8;
        int k = i+j;
    }
}
```

字节码如下

```
 public void testAdd();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=4, args_size=1
        0: bipush        15
        2: istore_1
        3: bipush        8
        5: istore_2
        6: iload_1
        7: iload_2
        8: iadd
        9: istore_3
        10: return
```

1. 首先执行第一条语句，PC寄存器指向的是0，也就是指令地址为0，然后使用bipush让操作数15入操作数栈。  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/程序执行流程1.png)

2. 执行完后，让PC寄存器+1，指向下一行代码，下一行代码就是将操作数栈的元素存储到局部变量表索引1的位置，我们可以看到局部变量表的已经增加了一个元素  
3. 解释为什么局部变量表索引从1开始，因为该方法为实例方法，局部变量表索引为 `0`的位置存放的是`this`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/程序执行流程2.png) 

4. 然后PC寄存器+1，指向的是下一行。让操作数8也入栈，同时执行 istore 操作，存入局部变量表中

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/程序执行流程3.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/程序执行流程4.png)

5. 然后从局部变量表中，依次将数据取出放在操作数栈中，等待执行 add 操作 `iload_1` `iload_2`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/程序执行流程5.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/程序执行流程6.png)

6. 将操作数栈的两个元素出栈，执行iadd操作,即：执行引擎将字节码指令翻译成机器指令，然后被CPU进行运算，得出结果，重新放入操作数栈中  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/程序执行流程7.png)

7. 然后执行 istore 操作，将操作数23 存储到局部变量表索引为3的位置

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/程序执行流程8.png)

##### 栈顶缓存技术(Top Of Stack Cashing)
基于栈式架构的虚拟机所使用的零地址指令更加紧凑，但完成一项操作的时候必然需要使用更多的入栈和出栈指令，这同时也就意味着将需要**更多的指令分派（instruction dispatch）次数和内存读/写次数**

由于操作数是存储在内存中，频繁的进行内存读写操作影响执行速度，将栈顶元素全部缓存到物理 CPU 的寄存器中，以此降低对内存的读写次数，提升执行引擎的执行效率  


#### 动态连接Dynamic Linking

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/动态连接保存常量池方法引用.png)

- 每一个栈帧内部都包含一个指向运行时常量池中该栈帧所属方法的引用包含这个引用的目的就是为了支持当前方法的代码能够实现动态链接（Dynamic Linking），比如：`invokedynamic`指令
- 在Java源文件被编译到字节码文件中时，所有的变量和方法引用都作为符号引用（Symbolic Reference）保存在class文件的常量池里  
- 比如：描述一个方法调用了另外的其他方法时，就是通过常量池中指向方法的符号引用来表示的，那么`动态链接的作用就是为了将这些符号引用转换为调用方法的直接引用`

详细的解析查看 方法调用中的动态链接 静态链接

### 方法的调用 
- **静态链接**：当一个字节码文件被装载进 JVM 内部时，如果被调用的**目标方法在编译期可知**，且运行时期间保持不变，这种情况下降调用方的符号引用转为直接引用的过程称为静态链接。  
- **动态链接**：如果被调用的方法**无法在编译期被确定下来**，只能在运行期将调用的方法的符号引用转为直接引用，这种引用转换过程具备动态性，因此被称为动态链接

方法绑定： 
- **早期绑定**：被调用的目标方法如果在编译期可知，且运行期保持不变。
- **晚期绑定**：被调用的方法在编译期无法被确定，只能够在程序运行期根据实际的类型绑定相关的方法。  

```java
/**
 * 说明早期绑定和晚期绑定的例子
 */
class Animal {
    public void eat() {
        System.out.println("动物进食");
    }
}

interface Huntable {
    void hunt();
}

class Dog extends Animal implements Huntable {
    @Override
    public void eat() {
        System.out.println("狗吃骨头");
    }

    @Override
    public void hunt() {
        System.out.println("捕食耗子，多管闲事");
    }
}

class Cat extends Animal implements Huntable {
    public Cat() {
        super(); //表现为：早期绑定 invokespecial
    }

    public Cat(String name) {
        this(); //表现为：早期绑定
    }

    @Override
    public void eat() {
        super.eat(); //表现为：早期绑定
        System.out.println("猫吃鱼");
    }

    @Override
    public void hunt() {
        System.out.println("捕食耗子，天经地义");
    }
}

public class AnimalTest {
    public void showAnimal(Animal animal) {
        animal.eat(); //表现为：晚期绑定 invokevirtual 
    }

    public void showHunt(Huntable h) {
        h.hunt(); //表现为：晚期绑定 invokeinterface
    }
}

```

由上面的指令引入下面的概念**虚方法和非虚方法:**

- **非虚方法**：只要能被`invokestatic`和`invokespecial`指令调用的方法，都可以在解析阶段中确定唯一的调用版本，Java语言里符合这个条件的方法共有`静态方法`、`私有方法`、`实例构造器`、`父类方法`4种，再加上`被final修饰的方法`（尽管它使用invokevirtual指令调用），这5种方法调用会在类加载的时候就可以把符号引用解析为该方法的直接引用。这些方法统称为“非虚方法”`Non-Virtual Method`，与之相反，其他方法就被称为**虚方法**`Virtual Method`



| 编译期可知 | 编译期未知 |
| ---------- | ---------- |
| 静态链接   | 动态连接   |
| 早期绑定   | 晚期绑定   |
| 非虚方法   | 虚方法     |

#### 调用指令

- 普通调用指令：
  - `invokestatic`：调用静态方法，解析阶段确定唯一方法版本
  - `invokespecial`：调用`<init>`方法、私有及父类方法，解析阶段确定唯一方法版本
  - `invokevirtual`：调用所有虚方法
  - `invokeinterface`：用于调用接口方法，会在运行时再确定一个实现该接口的对象
- 动态调用指令
  - `invokedynamic`：先在运行时动态解析出调用点限定符所引用的方法，然后再执行该方法（Java7 引入）
  - Java8 后引入lambda表达式后，生成的字节码就是`invokedynamic`指令

区别: 
前四条指令逻辑固化在虚拟机内部，方法的调用执行不可人为干预   
`invokedynamic`指令逻辑是由用户设定的引导方法来决定的   
其中`invokestatic`指令和`invokespecial`指令调用的方法称为非虚方法，其余的（final修饰的除外）称为虚方法

#### 方法重写的本质 
- 找到操作数栈顶的第一个元素所执行的对象的实际类型，记做 C。如果在类型 C 中找到与常量池中描述符和简单名称都相符的方法，则进行访问权限校验。
- 如果通过则返回这个方法的直接引用，查找过程结束；如果不通过，则返回 `java.lang.IllegalAccessError` 异常。
- 否则，按照继承关系从下往上依次对 C 的各个父类进行上一步的搜索和验证过程。
- 如果始终没有找到合适的方法，则抛出 `java.lang.AbstractMethodError` 异常

#### 虚方法表
在面向对象编程中会很频繁的使用到动态分派，动态分派的方法版本选择过程需要运行时在接收者类型的方法元数据中搜索合适的目标方法，Java虚拟机实现基于执行性能的考虑，真正运行时一般不会如此频繁地去反复搜索类型元数据。面对这种情况，一种基础而且常见的优化手段是为类型在方法区中建立一个`虚方法表`（Virtual Method Table，也称为vtable，与此对应的，在invokeinterface执行时也会用到接口方法表——Interface Method Table，简称itable），使用虚方法表索引来代替元数据查找以提高性能  

虚方法表会在类加载的链接阶段(具体为解析)被创建并开始初始化，类的变量初始值准备完成之后，jvm会把该类的方法表也初始化完毕  

```java
public class Dispatch {
    static class QQ {
    }

    static class _360 {
    }

    public static class Father {
        public void hardChoice(QQ arg) {
            System.out.println("father choose qq");
        }

        public void hardChoice(_360 arg) {
            System.out.println("father choose 360");
        }
    }

    public static class Son extends Father {
        public void hardChoice(QQ arg) {
            System.out.println("son choose qq");
        }

        public void hardChoice(_360 arg) {
            System.out.println("son choose 360");
        }
    }

    public static void main(String[] args) {
        Father father = new Father();
        Father son = new Son();
        father.hardChoice(new _360());
        son.hardChoice(new QQ());
    }
}
```

上述代码的虚方法表如下

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/虚方法表示例.png)  

?> 子类重写或实现的方法虚方法表中的数据类型就是子类，否则就是父类或者共同父类...

### 方法出口（方法返回地址）  
方法返回地址就是存放调用该方法的PC寄存器的值  

一个方法的结束无非两种方式： 
- 正常执行完成
- 出现未处理异常，退出

无论通过那种方式退出，**退出后都需要返回方法被调用前的位置,程序才能继续执行**    
- 方法正常退出时，主调方法的PC计数器的值就可以作为返回地址，栈帧中很可能会保存这个计数器值。
- 而方法异常退出时，返回地址是要通过`异常处理器表`来确定的，栈帧中就一般不会保存这部分信息
  - 通过异常完成出口退出的不会给他的上层调用者产生任何的返回值 

> 本质上，方法的退出就是当前栈帧出栈的过程。此时，需要恢复上层方法的局部变量表、操作数栈、将返回值压入调用者栈帧的操作数栈、设置PC寄存器值等，让调用者方法继续执行下去

### 一些附加信息

虚拟机实现增加一些规范里没有描述的信息到栈帧之中，例如与调试、性能收集相关的信息，这部分信息完全取决于具体的虚拟机实现，这里不再详述。  
在讨论概念时，一般会把**动态连接、方法返回地址与其他附加信息全部归为一类**，称为**栈帧信息**

## 本地方法栈
本地方法栈是为 JVM 运行 `Native` 方法准备的空间，由于很多 `Native` 方法都是用 C 语言实现的，所以它通常又叫`C 栈`。它与 Java 虚拟机栈实现的功能类似，只不过本地方法栈是描述本地方法运行过程的内存模型。

本地方法栈也会在栈深度溢出或者栈扩展失败时分别抛出`StackOverflowError`和`OutOfMemoryError`异常

本地方法被执行时，在本地方法栈也会创建一块栈帧，用于存放该方法的局部变量表、操作数栈、动态链接、方法出口信息等。

## 堆 Heap
一个JVM实例只存在一个堆内存，堆也是Java内存管理的核心区域    

- Java堆区在JVM启动是就被创建，空间大小也就确定了,`堆是JVM管理的最大一块内存空间`
- 堆内存的大小是可以调节的(启动前可以通过参数 `-Xms` `-Xmx` 调节)
- 堆可以处于物理上不连续的内存空间中，但在逻辑上它应该被视为连续的
- 所有的线程共享Java堆，但是这里还可以划分`线程私有的缓冲区`（Thread Local Allocation Buffer，TLAB）
- **所有的对象实例以及数组都应当在运行时分配在堆上** ---《Java虚拟机规范》
- 但是实际情况是：`几乎`**所有的对象实例都在这里分配内存**。因为还有一些对象是在栈上分配的（`逃逸分析`，`标量替换`）
- **数组和对象可能永远不会存储在栈上**，因为栈帧中保存引用，这个引用指向对象或者数组在堆中的位置
- 在方法结束后，堆中的对象不会马上被移除，仅仅在垃圾回收`GC`的时候才会被移除
  - 如果堆中对象马上被回收，那么用户线程就会收到影响，因为有`stop the word`(回收动作会暂停所有java程序运行)
- 堆，是GC（Garbage Collection，垃圾收集器）执行垃圾回收的重点区域。


### 查看堆内存
通过`/jdk/bin/jvisualvm.exe` 即`Java VisualVM`程序查看当前运行的所有java程序信息,同时可以安装插件实现查看GC   

### 堆的细分内存结构

在JDK7 以及之前的版本，堆内存逻辑上分为三部分：新生代、老年代、永久代
- `Young Generation Space` 新生区 `Young/New`
  - 又被划分为 `Eden区` 和 `Survivor区`
- `Tenure generation space` 养老区 `Old/Tenure`
- `Permanent generation Space`永久区 `Perm`

在JDK8 以及之后的版本，堆内存逻辑上分为三部分：新生代、老年代、元空间
- `Meta Space` 元空间 `Meta`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jdk7和8的堆结构区别.png)

?> 堆空间的内存主要在新生代和老年代，逻辑上包括 `永久代/元空间`，实际上控制不到

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/堆空间结构和方法区的关系.png)


#### 设置堆内存大小与OOM
- Java堆区用于存储Java对象实例，那么堆的大小在JVM启动时就已经设定好了，大家可以通过选项`-Xms`和`-Xmx`来进行设置。
  - `-Xms` 用于表示堆区的初始内存，等价于` -XX:InitialHeapSize`
  - `-Xmx`则用于表示堆区的最大内存，等价于 `-XX:MaxHeapSize`
- 一旦堆区中的内存大小超过`-Xmx`所指定的最大内存时，将会抛出`OutofMemoryError`异常。
- 通常会将`-Xms`和`-Xmx`两个参数配置相同的值，其目的是为了能够在Java垃圾回收机制清理完堆区后不需要重新分隔计算堆区的大小，从而提高性能。
- 默认情况下:
  - 初始内存大小：`物理电脑内存大小/64` 16g内存电脑的初始内存`1/4g`
  - 最大内存大小：`物理电脑内存大小/4`

#### 查看堆内存大小
两种查看堆内存的方式
- 方式一：命令行依次执行如下两个指令
  - `jps`
  -` jstat -gc 进程id`  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/命令查看堆内存大小信息.png)

- 方式二：设置虚拟机参数 -XX:+PrintGCDetails   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/虚拟机参数查看堆内存大小信息.png)

> 为什么设置 600MB ，算出来只有 575MB 呢？   
> from区和to区只能有一个区存放对象，所以相加的时候只能加上一个区的大小,可以看到新生区的大小 = 伊甸园区大小 + 幸存者 from/to 区大小  
> 即 `179200KB = 153600KB + 25600KB` => `179200KB + 409600kb = 588800kb` =>  `588800kb/1024 = 575Mb`  

`OutOfMemoryError`堆内存溢出的变化过程

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/堆内存溢出变化过程.gif ':size=60%')


### 年轻代与老年代
- 存储在JVM中的Java对象可以被划分为两类：
  - 一类是生命周期较短的瞬时对象，这类对象的创建和消亡都非常迅速
  - 另外一类对象的生命周期却非常长，在某些极端的情况下还能够与JVM的生命周期保持一致

Java堆区进一步细分的话，可以划分为`年轻代（YoungGen）`和`老年代（oldGen）`   
其中年轻代又可以划分为`Eden空间`(伊甸园空间)、`Survivor0空间`(幸存者0区)和`Survivor1空间`(幸存者1区)（有时也叫做from区、to区）  
from区和to区的不是固定的，是相对的定义  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/JVM的年轻代和老年代.png)


#### 参数调整
配置`新生代`与`老年代`在堆结构的占比（下面这些参数在开发中一般不会调）

- 默认`-XX:NewRatio=2`，表示新生代占1，老年代占2，新生代占整个堆的1/3
- 可以修改`-XX:NewRatio=4`，表示新生代占1，老年代占4，新生代占整个堆的1/5

?> 当发现在整个项目中，生命周期长的对象偏多，那么就可以通过调整老年代的大小，来进行调优  

配置`新生代`中，`Eden空间`和另外两个`Survivor空间`占比     
- 在HotSpot中，`Eden空间`和另外两个`Survivor空间`缺省所占的比例是`8 : 1 : 1`
- 可以通过选项`-XX:SurvivorRatio`调整这个空间比例。比如`-XX:SurvivorRatio=8` 
- 虚拟机默认开启了自适应的内存分配策略，会导致`SurvivorRatio`配置失效，使用参数`-XX:-UseAdaptiveSizePolicy`关闭自适应内存分配

?> 几乎所有的Java对象都是在Eden区被new出来的，绝大部分的Java对象的销毁都在新生代进行了（有些大的对象在Eden区无法存储时候，将直接进入老年代）    

#### 对象分配的流程
- 创建对象
- 放入Eden区 
- Eden区空间填满时，再创建对象时，JVM垃圾回收器将堆Eden区进行垃圾回收(Young GC/Minor GC),将没有引用的对象进行销毁
- 然后将Eden区中的剩余对象移动到Survivor0区
  - 此时To区就是S0区
- 如果Eden再满，继续进行GC(回收Eden和S0)，将剩余有引用的对象移动到S1区(S0区的对象age+1)，清空Eden区
  - 此时To区是S1区
- 如果Eden再满，继续进行GC(回收Eden和S1)，将剩余有引用的对象(S1内的对象age+1)移动到S0区，清空Eden区(如此往复)
  - 此时To区是S0区
- 当对象age=15时 放入老年代同时age+1

#### 对象分配的特殊情况 

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/对象分配的特殊情况.png)

- 对于超大对象如果Eden放不下可直接放入老年代 
- 如果老年代放不下，进行`Full GC`,仍然放不下就`OOM`
- MinorGC 若S区放不下对象，可能直接将部分放入老年代


#### 常用的 JVM 调优工具
- JDK命令行
- Eclipse：Memory Analyzer Tool
- Jconsole
- Visual VM（实时监控 推荐~）
- Jprofiler（推荐~）
- Java Flight Recorder（实时监控）
- GCViewer
- GCEasy

### GC垃圾回收器
`Minor GC`、`Major GC`、`Full GC`   
要尽量的避免垃圾回收，因为在垃圾回收的过程中，容易出现`STW(Stop the World)`的问题，而 `Major GC 和 Full GC`出现STW的时间，是Minor GC的**10倍以上**

JVM在进行GC时，并非每次都对上面三个内存(`新生代、老年代、方法区`)区域一起回收的，大部分时候回收的都是指新生代。    
针对Hotspot VM的实现，它里面的GC按照回收区域又分为两大种类型：一种是部分收集(`Partial GC`)，一种是整堆收集(`FullGC`)    

部分收集:  
- 新生代收集(Minor GC/Young GC)只是新生代(Eden、S0/S1)的垃圾收集
- 老年代收集(Major GC/Old GC)：只是老年代的垃圾收集
  - 很多时候Major GC会和Full GC混淆使用，需要具体分辨是老年代回收还是整堆回收
  - `只有CMS GC会有单独收集老年代的行为`
- 整堆收集(Full GC)：收集整个java堆和方法区的垃圾收集  

#### Minor GC 
- 当年轻代空间不足时，就会触发Minor GC，这里的年轻代满指的是`Eden`区满，**Survivor区满不会触发GC**。(每次Minor GC会清理年轻代的内存)
- 因为Java对象大多都具备朝生夕灭的特性，所以**Minor GC非常频繁**，一般**回收速度也比较快**。
- Minor GC会引发STW，`暂停其它用户的线程，等待垃圾回收线程结束，用户线程才恢复运行`


#### Major GC
- 指发生在老年代的GC，对象从老年代消失时，我们说`Major GC` 或 `Full GC`发生了
- 出现了MajorGc，经常会伴随至少一次的Minor GC   
  - 但非绝对的，在Parallel Scavenge收集器的收集策略里就有直接进行Major GC的策略选择过程
  - 也就是在老年代空间不足时，会先尝试触发Minor GC，如果之后空间还不足，则触发Major GC
- Major GC的速度一般会比Minor GC慢10倍以上，`STW的时间更长`
- 如果Major GC后，内存还不足，就报OOM了
  - 在 OOM 之前，一定会触发一次 Full GC ，因为只有在老年代空间不足且进行垃圾回收后仍然空间不足的时候，才会爆出OOM异常

#### Full GC
Full GC 触发机制后续详细讲解   

**触发Full GC执行的情况有如下五种：**

- 调用System.gc( )时，系统建议执行Full GC，但是不必然执行
- 老年代空间不足
- 方法区空间不足
- 通过Minor GC后进入老年代的平均大小 大于 老年代的可用内存
- 由Eden区、survivor space0(From Space)区 向survivor space1(To Space)区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小

### 堆空间的分代思想
什么要把Java堆分代？不分代就不能正常工作了吗？   
**分代的唯一理由就是优化GC性能**   

经研究，不同对象的生命周期不同。70%-99%的对象是临时对象，分代可以让无用对象快速移除，避免全部GC带来的性能损耗   

### 内存分配策略
- 如果对象在Eden出生并经过第一次Minor GC后仍然存活，并且能被Survivor容纳的话，将被移动到Survivor空间中，并将对象年龄设为1。   
- 对象在Survivor区中每熬过一次MinorGC，年龄就增加1岁，当它的年龄增加到一定程度(默认为15岁，其实每个JVM、每个GC都有所不同)时，就会被晋升到老年代
- 对象晋升老年代的年龄阀值，可以通过选项`-XX:MaxTenuringThreshold`来设置

针对不同年龄段的对象分配原则如下所示：

- 优先分配到`Eden`
- 大对象直接分配到老年代,尽量避免程序中出现过多的大对象
- 长期存活的对象分配到老年代  

```java
/**
 * 测试：大对象直接进入老年代
 * -Xms60m -Xmx60m -XX:NewRatio=2 -XX:SurvivorRatio=8 -XX:+PrintGCDetails
    新生代 20m 老年代40m  eden 20*0.8=16m s0/s1 20*0.1 = 2m
 */
public class YoungOldAreaTest {
    public static void main(String[] args) {
        //20m的数据直接进入老年代不会进行垃圾回收 也不会报oom
        byte[] buffer = new byte[1024 * 1024 * 20]; 

    }
}
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/大对象直接进入老年代示例.png)

- 动态对象年龄判断：
  - 如果Survivor区中相同年龄的所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象可以直接进入老年代，无须等到`MaxTenuringThreshold`中要求的年龄。
- 空间分配担保：
  - `-XX:HandlePromotionFailure` ，也就是经过Minor GC后，所有的对象都存活，因为Survivor比较小，所以就需要将Survivor无法容纳的对象，存放到老年代中
  - JDK7及以后这个参数就失效


### TLAB 线程本地分配缓存区
堆是线程全局共享的，在同一时间，可能会有多个线程在堆上申请空间，但每次的对象分配需要同步的进行，会导致线程安全问题  
为了避免多个线程操作同一个地址，需要加锁等机制，进而影响分配速度   

为了解决上述问题，**JVM为每个线程分配了一个私有的缓存区域**,包含在Eden空间内    
多线程同时分配内存时，**使用TLAB可以避免一系列的非线程安全问题，同时还能够提升内存分配的吞吐量**，因此我们可以将这种内存分配方式称之为`快速分配策略`  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/TLAB分配Eden示例图.png)

> TLAB 的全称是 `Thread Local Allocation Buffer`，即线程**本地分配缓存区**，是属于Eden区的，这是一个线程专用的内存分配区域，线程私有，**默认开启**     
> 默认情况下TLAB空间只占Eden的1%，可以通过配置参数`-XX:TLABWasteTargetPercent`进行配置占比   
> 参数配置`-XX:+UseTLAB` 使用TLAB，`-XX:+TLABSize` 设置TLAB大小 

> 并不是所有的对象都可以在 TLAB 中分配内存成功，如果失败了就会使用加锁的机制来保持操作的原子性

### 堆空间的常用参数  
- `-XX:+PrintFlagsInitial`：查看所有的参数的默认初始值
- `-XX:+PrintFlagsFinal`：查看所有的参数的最终值（可能会存在修改，不再是初始值）
- `-Xms`：初始堆空间内存（默认为物理内存的1/64）
- `-Xmx`：最大堆空间内存（默认为物理内存的1/4）
- `-Xmn`：设置新生代的大小（初始值及最大值）
- `-XX:NewRatio`：配置新生代与老年代在堆结构的占比
- `-XX:SurvivorRatio`：设置新生代中Eden和S0/S1空间的比例
- `-XX:MaxTenuringThreshold`：设置新生代垃圾的最大年龄
- `-XX:+PrintGCDetails`：输出详细的GC处理日志
- `-XX:+PrintGC` 或 `-verbose:gc` ：打印gc简要信息
- `-XX:HandlePromotionFalilure`：是否设置空间分配担保


### 逃逸分析

#### 堆是分配对象存储的唯一选择吗？
随着JIT编译期的发展与逃逸分析技术逐渐成熟，`栈上分配、标量替换优化技术`将会导致一些微妙的变化，所有的对象都分配到堆上也渐渐变得不那么“绝对”了

在Java虚拟机中，对象是在Java堆中分配内存的，这是一个普遍的常识。但是，有一种特殊情况，那就是如果经过逃逸分析`Escape Analysis`后发现，**一个对象并没有逃逸出方法的话，那么就可能被优化成栈上分配。这样就无需在堆上分配内存，也无须进行垃圾回收了**。这也是最常见的堆外存储技术


#### 逃逸分析概述
逃逸分析`Escape Analysis`是目前Java虚拟机中比较前沿的优化技术，它并不是直接优化代码的手段，而是**为其他优化措施提供依据的分析技术**

逃逸分析的基本原理是：  
- 分析对象动态作用域，当一个对象在`方法里面被定义`后，它可能被外部方法所引用，例如作为调用参数传递到其他方法中，这种称为`方法逃逸`；
- 还有可能被外部线程访问到，譬如赋值给可以在其他线程中访问的实例变量，这种称为`线程逃逸`；
- 若对象只在方法内部使用，则认为没有逃逸

?> 从不逃逸、方法逃逸到线程逃逸，称为对象由低到高的不同逃逸程度

```java
//没有逃逸
public void my_method() {
    V v = new V();
    // use v
    // ....
    v = null;
}
//发生逃逸
public static StringBuffer createStringBuffer(String s1, String s2) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    return sb;
}
//sb对象 没有逃逸 sb对象没有逃出方法内部 
public static String createStringBuffer(String s1, String s2) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    return sb.toString();
}

```

?> 逃逸的几种情况，主要看方法内部创建的对象是否逃到方法外(方法返回对象，传递对象)，方法内是否引用外来对象(成员变量的赋值，引用)  

!> 开发中能使用局部变量的，就不要使用在方法外定义

#### 逃逸分析参数 
- 在JDK 1.7 版本之后，HotSpot中默认就已经开启了逃逸分析
- 如果使用的是较早的版本，开发人员则可以通过：
  - 选项`-XX:+DoEscapeAnalysis`显式开启逃逸分析
  - 通过选项`-XX:+PrintEscapeAnalysis`查看逃逸分析的筛选结果


#### 栈上分配
JIT编译器在编译期间根据逃逸分析的结果，发现如果**一个对象并没有逃逸出方法**的话，就可能被优化成`栈上分配`    
分配完成后，继续在调用栈内执行，最后线程结束，栈空间被回收，局部变量对象也被回收。这样就无须进行垃圾回收了  

**示例**

```java
/**
 * 栈上分配测试
 * -Xmx256m -Xms256m -XX:-DoEscapeAnalysis -XX:+PrintGCDetails
 */
public class StackAllocation {
    public static void main(String[] args) {
        long start = System.currentTimeMillis();

        for (int i = 0; i < 10000000; i++) {
            alloc();
        }
        // 查看执行时间
        long end = System.currentTimeMillis();
        System.out.println("花费的时间为： " + (end - start) + " ms");
        // 为了方便查看堆内存中对象个数，线程sleep
        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e1) {
            e1.printStackTrace();
        }
    }

    private static void alloc() {
        User user = new User(); //未发生逃逸
    }

    static class User {

    }
}
```

首先查看**未开启逃逸分析**的情况,JVM 参数设置:`-Xmx256m -Xms256m -XX:-DoEscapeAnalysis -XX:+PrintGCDetails` 关闭逃逸分析，并打印gc参数

```
[GC (Allocation Failure) [PSYoungGen: 65536K->904K(76288K)] 65536K->912K(251392K), 0.0016251 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[GC (Allocation Failure) [PSYoungGen: 66440K->920K(76288K)] 66448K->928K(251392K), 0.0009567 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
花费的时间为： 64 ms
```

可以看到耗时64m 通过JVisualVM查看堆内存图如下，有大量的对象在堆中  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/逃逸分析示例图.png)

开启逃逸分析后再次尝试`-Xmx256m -Xms256m -XX:+DoEscapeAnalysis -XX:+PrintGCDetails`  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/逃逸分析示例图2.png) 

可以看到执行时间很短，也没有进行垃圾回收操作，堆中的对象也减少很多(下图)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/逃逸分析示例图3.png)


#### 同步省略

- 线程同步的代价是相当高的，同步的后果是降低并发性和性能。
- 在动态编译同步块的时候，JIT编译器可以借助`逃逸分析`来判断同步块所使用的锁对象是否只能够被一个线程访问而没有被发布到其他线程。
- 如果没有，那么JIT编译器在编译这个同步块的时候就会取消对这部分代码的同步。这样就能大大提高并发性和性能。这个取消同步的过程就叫`同步省略`，也叫`锁消除`。

下面的代码就会采用同步省略优化

```java
public void f() {
    Object hellis = new Object();
    synchronized(hellis) {
        System.out.println(hellis);
    }
}
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/同步省略字节码文件.png)

?> 字节码文件中并没有进行优化，可以看到**加锁和释放锁的操作依然存在**，**同步省略操作是在解释运行时发生的**


#### 分离对象或标量替换
- 标量（scalar）是指一个无法再分解成更小的数据的数据。**Java中的原始数据类型就是标量**。
- 相对的，那些还可以分解的数据叫做聚合量（Aggregate），**Java中的对象就是聚合量**，因为他可以分解成其他聚合量和标量。
- 在JIT阶段，如果经过逃逸分析，发现一个对象不会被外界访问的话，那么经过JIT优化，就会把这个对象拆解成若干个其中包含的若干个成员变量来代替。这个过程就是标量替换

```java
public static void main(String args[]) {
    alloc();
}
class Point {
    private int x;
    private int y;
}
private static void alloc() {
    Point point = new Point(1,2);
    System.out.println("point.x" + point.x + ";point.y" + point.y);
}

```

**经过标量替换就会变成**

```java
private static void alloc() {
    int x = 1;
    int y = 2;
    System.out.println("point.x = " + x + "; point.y=" + y);
}
```

标量替换参数 `-XX:+ElimilnateAllocations` 开启了标量替换（默认打开），允许将对象打散分配在栈上。

#### 逃逸分析参数设置总结
- 参数 `-server`：启动Server模式，因为在server模式下(可以通过`java -version`查看)，才可以启用逃逸分析。
- 参数 `-XX:+DoEscapeAnalysis`：启用逃逸分析
- 参数 `-Xmx10m`：指定了堆空间最大为10MB
- 参数 `-XX:+PrintGC`：将打印GC日志。
- 参数 `-XX:+EliminateAllocations`：开启了标量替换（默认打开），允许将对象打散分配在栈上，比如对象拥有id和name两个字段，那么这两个字段将会被视为两个独立的局部变量进行分配

#### 不足
根本原因就是无法保证逃逸分析的性能消耗一定能高于他的消耗。虽然经过逃逸分析可以做标量替换、栈上分配、和锁消除。但是逃逸分析自身也是需要进行一系列复杂的分析的，这其实也是一个相对耗时的过程。一个极端的例子，就是经过逃逸分析之后，发现没有一个对象是不逃逸的。那这个逃逸分析的过程就白白浪费掉了

> 在实际的应用程序中，尤其是大型程序中反而发现实施逃逸分析可能出现效果不稳定的情况，或分析过程耗时但却无法有效判别出非逃逸对象而导致性能（即时编译的收益）下降  --《深入理解Java虚拟机》11.4.3


## 方法区
方法区`Method Area`与Java堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的`类型信息`、`常量`、`静态变量`、`即时编译器编译后的代码缓存`等数据     

!> 虽然`《Java虚拟机规范》`中把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫作`非堆`(Non-Heap)，目的是与Java堆区分开来。   
**方法区看作是一块独立于Java堆的内存空间**    

### 栈、堆、方法区的交互关系
- Person 类的 .class 信息存放在方法区中
- Person 变量存放在 Java 栈的局部变量表中
- 真正的 Person 对象存放在 Java 堆中

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/堆栈方法区的关系.png ':size=50%')

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/堆栈方法区的关系1.png ':size=50%')


### 方法区的理解
**方法区主要存放的是 Class，而堆中主要存放的是实例化的对象**   
- 方法区`Method Area`与Java堆一样，是各个线程**共享的内存区域**
- 多个线程同时加载统一个类时，只能有一个线程能加载该类，其他线程只能等等待该线程加载完毕，然后直接使用该类，即**类只能加载一次**。
- 方法区在JVM启动的时候被创建，并且它的实际物理内存空间和Java堆区一样都可以是不连续的。
- 方法区的大小，跟堆空间一样，可以选择固定大小或者可扩展。
- 方法区是接口，元空间或者永久代是方法区的实现(见下图)
- 方法区的大小决定了系统可以保存多少个类，如果系统定义了太多的类，导致方法区溢出，虚拟机同样会抛出内存溢出错误：
  - `java.lang.OutofMemoryError:PermGen space`JDK7之前
  - `java.lang.OutOfMemoryError:Metaspace`JDK8之后
- 生命周期和JVM相同

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/方法区-永久代-元空间.png)

### Hotspot中方法区的演进过程
- 在 JDK7 及以前，习惯上把`方法区`，称为`永久代`。JDK8开始，使用`元空间`取代了`永久代`。JDK 1.8之后，**元空间存放在堆外内存中**
- 将方法区类比为Java中的接口，将永久代或元空间类比为Java中具体的实现类
- 本质上，**方法区和永久代并不等价**。仅是对`Hotspot`而言的可以看作等价。
  - 《Java虚拟机规范》对如何实现方法区，不做统一要求。例如：BEAJRockit / IBM J9 中不存在永久代的概念。
- 使用永久代，导致Java程序更容易OOm（超过-XX:MaxPermsize上限）
- 而到了JDK8，完全废弃了`永久代`的概念，改用与JRockit、J9一样在本地内存中实现的`元空间（Metaspace）`来代替
- 元空间的本质和永久代类似，都是对**JVM规范中方法区的实现**。
  - 不过元空间与永久代最大的区别在于：**元空间不在虚拟机设置的内存中，而是使用本地内存**
  - 永久代、元空间二者并不只是名字变了，内部结构也调整了
- 根据《Java虚拟机规范》的规定，如果方法区无法满足新的内存分配需求时，将抛出OOM异常

### 设置方法区大小与OOM
方法区的大小不必是固定的，JVM可以根据应用的需要动态调整,下面只讨论固定的情况

- JDK7 永久代设置
  - 通过`-XX:Permsize`来设置永久代初始分配空间。默认值是`20.75M`
  - `-XX:MaxPermsize`来设定永久代最大可分配空间。32位机器默认是`64M`，64位机器模式是`82M`
  - 当JVM加载的类信息容量超过了这个值，会报异常`OutofMemoryError:PermGen space`。

- JDK8 元空间
  - 元数据区大小可以使用参数 `-XX:MetaspaceSize` 和 `-XX:MaxMetaspaceSize` 指定
  - 默认值依赖于平台，Windows下，`-XX:MetaspaceSize` 约为21M，`-XX:MaxMetaspaceSize`的值是-1，即没有限制。
  - 与永久代不同，如果不指定大小，默认情况下，虚拟机会耗尽所有的可用系统内存。如果元数据区发生溢出，虚拟机一样会抛出异常`OutOfMemoryError:Metaspace`
  - `-XX:MetaspaceSize`：设置初始的元空间大小。对于一个 64位的服务器端JVM来说，其默认的 `-XX:MetaspaceSize`值为21MB。这就是初始的高水位线，一旦触及这个水位线，Full GC将会被触发并卸载没用的类（即这些类对应的类加载器不再存活），然后这个高水位线将会重置。新的高水位线的值取决于GC后释放了多少元空间。
    - 如果释放的空间不足，那么在不超过MaxMetaspaceSize时，适当提高该值。
    - 如果释放空间过多，则适当降低该值。
    - 如果初始化的高水位线设置过低，上述高水位线调整情况会发生很多次。通过垃圾回收器的日志可以观察到Full GC多次调用。为了避免频繁地GC，建议将`-XX:MetaspaceSize`设置为一个相对较高的值

### 方法区的内部结构
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/方法区结构概述.png)

方法区的存储内容包括`类型信息`、`常量`(常量池)、`静态变量`、`即时编译器编译后的代码缓存`等数据

#### 类型信息
对每个加载的类型（类class、接口interface、枚举enum、注解annotation），JVM必须在方法区中存储以下类型信息：  
- 这个类型的完整有效名称（全类名=包名.类名）
- 这个类型直接父类的完整有效名（对于interface或是java.lang.Object，都没有父类）
- 这个类型的修饰符（public，abstract，final的某个子集）
- 这个类型直接接口的一个有序列表

#### 域(Field)信息
- JVM必须在方法区中保存类型的所有域的相关信息以及域的声明顺序。
- 域信息通俗来讲是类的成员变量
- 域的相关信息包括：
  - 域名称
  - 域类型
  - 域修饰符(public，private，protected，static，final，volatile，transient的某个子集)

#### 方法(Method)信息
VM必须保存所有方法的以下信息，同域信息一样包括声明顺序：
- 方法名称
- 方法的返回类型(包括 void 返回类型)，void 在 Java 中对应的类为 void.class
- 方法参数的数量和类型(按顺序
- 方法的修饰符(public，private，protected，static，final，synchronized，native，abstract的一个子集)
- 方法的字节码(bytecodes)、操作数栈、局部变量表及大小(abstract和native方法除外)
- 异常表(abstract和native方法除外)，异常表记录每个异常处理的开始位置、结束位置、代码处理在程序计数器中的偏移地址、被捕获的异常类的常量池索引



### 方法区示例

```java
/**
 * 测试方法区的内部构成
 */
public class MethodInnerStrucTest extends Object implements Comparable<String>, Serializable {
    //属性
    public int num = 10;
    private static String str = "测试方法的内部结构";

    //构造器没写

    //方法
    public void test1() {
        int count = 20;
        System.out.println("count = " + count);
    }

    public static int test2(int cal) {
        int result = 0;
        try {
            int value = 30;
            result = value / cal;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }

    @Override
    public int compareTo(String o) {
        return 0;
    }
}
```

> 上述代码通过`javap -v -p MethodInnerStrucTest.class`反编译后

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/反编译方法区常量池.jpg)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/方法区反编译域信息.png)

### 运行时常量池
**运行时常量池(Runtime Constant Pool)**是**方法区**的一部分。Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池表(Constant Pool Table)，用于**存放编译期生成的各种字面量与符号引用**，这部分内容将在`类加载`后存放到方法区的`运行时常量池`中

- 在加载类和接口到虚拟机后，就会创建对应的运行时常量池
- **JVM为每个已加载的类型（类或接口）都维护一个常量池。池中的数据项像数组项一样，是通过索引访问的**(上面的图)
- 运行时常量池中包含多种不同的常量，包括编译期就已经明确的数值字面量，也包括到**运行期解析后才能够获得的方法或者字段引用**。此时不再是常量池中的符号地址了，这里换为**真实地址**   

运行时常量池相对于Class文件常量池的另外一个重要特征是**具备动态性**，Java语言并不要求常量一定只有编译期才能产生，也就是说，并非预置入Class文件中常量池的内容才能进入方法区运行时常量池，**运行期间也可以将新的常量放入池中**，这种特性被开发人员利用得比较多的便是`String类的intern()`方法  

### 方法区的演进

**只有Hotspot才有永久代**

| JDK版本      | 演变细节                                                                                                     |
| ------------ | ------------------------------------------------------------------------------------------------------------ |
| JDK1.6及以前 | 有永久代（permanent generation），`静态变量`，`运行时常量池(包括字符串常量池)`都存储在永久代上，使用虚拟内存 |
| JDK1.7       | 有永久代，但已经逐步 “去永久代”，`字符串常量池`、`静态变量`从永久代中移除，保存在`堆`中，使用虚拟内存        |
| JDK1.8       | 无永久代，类型信息，字段，方法，常量保存在本地内存的元空间，但字符串常量池、静态变量仍然在堆中，使用本地内存 |

### 问题

#### 永久代为什么要被元空间替代 
官方解释  https://openjdk.org/jeps/122

- 为永久代设置空间大小是很难确定的
  - 某些场景下可能导致动态加载很多类，导致OOM
- 元空间并不在虚拟机中，而是使用本地内存。因此，默认情况下，**元空间的大小仅受本地内存限制**。
- 对永久代进行调优是很困难的
  - 方法区垃圾回收主要是两部分：常量池和类型的回收，元空间相对较大(不受`-XX:MaxPermSize`的限制)降低FullGC频率

#### 字符串常量池 StringTable为什么要调整位置？
字符串常量池在JDK7 由永久代迁移到堆空间中，并在JDK8中仍然保持在堆空间    

- JDK7中将StringTable放到了堆空间中。因为**永久代的回收效率很低，在Full GC的时候才会执行永久代的垃圾回收**，而Full GC是老年代的空间不足、永久代不足时才会触发。
- 这就导致StringTable回收效率不高，而我们开发中会有大量的字符串被创建，回收效率低，导致永久代内存不足。放到堆里，能及时回收内存

#### 静态变量位置

```java
/**
 * 结论：
 *  静态变量在jdk6/7存在与永久代中，在jdk8存在于堆中 //private static byte[] arr
 *  静态引用对应的对象实体始终都存在堆空间 //new byte[1024 * 1024 * 100];
 *
 * jdk7：
 * -Xms200m -Xmx200m -XX:PermSize=300m -XX:MaxPermSize=300m -XX:+PrintGCDetails
 * jdk 8：
 * -Xms200m -Xmx200m -XX:MetaspaceSize=300m -XX:MaxMetaspaceSize=300m -XX:+PrintGCDetails
 */
public class StaticFieldTest {
    private static byte[] arr = new byte[1024 * 1024 * 100]; //100MB

    public static void main(String[] args) {
        System.out.println(StaticFieldTest.arr);
    }
}
```

### 方法区的垃圾回收
方法区的垃圾收集主要回收两部分内容：`废弃的常量`和`不再使用的类型`。
- 回收废弃常量与回收Java堆中的对象非常类似。

> **举个常量池中字面量回收的例子**:假如一个字符串`java`曾经进入常量池中，但是当前系统又没有任何一个字符串对象的值是“java”，换句话说，已经没有任何字符串对象引用常量池中的`java`常量，且虚拟机中也没有其他地方引用这个字面量。如果在这时发生内存回收，而且垃圾收集器判断确有必要的话，这个“java”常量就将会被系统清理出常量池。常量池中其他类（接口）、方法、字段的符号引用也与此类似

- 不再被使用的类,类回收就比较**苛刻**
  - 该类所有的实例都已经被回收，也就是Java堆中不存在该类及其任何派生子类的实例。
  - 加载该类的类加载器已经被回收，这个条件除非是经过精心设计的可替换类加载器的场景，如OSGi、JSP的重加载等，否则通常是很难达成的。
  - 该类对应的`java.lang.Class`对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法

> 在**大量使用反射、动态代理、CGLib等字节码框架**，动态生成JSP以及OSGi这类频繁自定义类加载器的场景中，通常都需要Java虚拟机具备类型卸载的能力，以保证不会对方法区造成过大的内存压力


## 对象的实例化内存布局与访问定位

### 创建对象的方式
- `new`：最常见的方式、单例类中调用getInstance的静态类方法、XXXFactory的静态方法
- Class的`newInstance`方法：反射的方式，在JDK9里面被标记为过时的方法，因为**只能调用空参构造器，并且权限必须为 public**
- `Constructor`的`newInstance(Xxxx)`：反射的方式，可以调用空参或带参的构造器，权限没有要求

```java
    Test oomTest1 = Test.class.newInstance();
            
    Constructor<?>[] constructors = Test.class.getConstructors();
    Test oomTest = (Test) constructors[0].newInstance();
```

- 使用`clone()`：不调用任何的构造器，要求当前的类需要实现Cloneable接口中的clone( )方法
- 使用反序列化：序列化一般用于Socket的网络传输
- 第三方库 `Objenesis`

### 创建对象的步骤

```java
public class ObjectTest {
    public static void main(String[] args) {
        Object obj = new Object();
    }
}
```

> 上述代码通过 `javap`解析字节码文件  

```
  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=1
         0: new           #2                  // class java/lang/Object 去常量池找Object
         3: dup     //复制栈中变量引用
         4: invokespecial #1                  // Method java/lang/Object."<init>":()V 调用Object 构造器
         7: astore_1 //变量从操作数栈放入局部变量表
         8: return
      LineNumberTable:
        line 8: 0
        line 9: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  args   [Ljava/lang/String;
            8       1     1   obj   Ljava/lang/Object;

```

### 步骤详细说明
- 判断对象对应的类是否加载、链接、初始化
  - 虚拟机遇到一条new指令，首先去检查这个指令的参数能否在Metaspace的常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已经被加载，解析和初始化。（即判断类元信息是否存在）。
  - 如果该类没有加载，那么在双亲委派模式下，使用当前类加载器以`ClassLoader+包名+类名`为key进行查找对应的.class文件，如果没有找到文件，则抛出`ClassNotFoundException`异常，如果找到，则进行类加载，并生成对应的Class类对象。
- 为对象分配内存
  - 首先计算对象占用空间的大小，接着在堆中划分一块内存给新对象。如果实例成员变量是引用变量，仅分配引用变量空间即可，即4个字节大小
  - 如果内存规整：采用指针碰撞分配内存
  - 如果内存不是规整的，已使用的内存和未使用的内存相互交错，那么虚拟机将采用的是空闲列表来为对象分配内存
- 处理并发安全问题
  - 采用CAS+失败重试、区域加锁保证更新的原子性
  - 每个线程预先分配TLAB — 通过设置 -XX:+/-UseTLAB参数来设置（区域加锁机制）在Eden区给每个线程分配一块区域
- 初始化分配到的空间
- 所有属性设置默认值，保证对象实例字段在不赋值时可以直接使用
- 设置对象的对象头
  - 将对象的所属类(即类的元数据信息)、对象的HashCode和对象的GC信息、锁信息等数据存储在对象的对象头中。这个过程的具体设置方式取决于JVM实现
- 执行init方法进行初始化  

P102-104 

### 对象的内存布局

#### 对象头
对象头包含两部分：`运行时元数据(Mark Word)`和`类型指针`

- 运行时元数据
  - 哈希值（HashCode），可以看作是堆中对象的地址
  - GC分代年龄（年龄计数器）
  - 锁状态标志
  - 线程持有的锁
  - 偏向线程ID
  - 偏向时间戳
- 类型指针
  - 指向类元数据InstanceClass`obj.getClass()`，确定该对象所属的类型。指向的其实是方法区中存放的类元信息

?> 如果对象是数组，还需要记录数组的长度

####  实例数据
对象真正存储的有效信息，包括程序代码中定义的各种类型的字段（包括从父类继承下来的和本身拥有的字段）

- 相同宽度的字段总是被分配在一起
- 父类中定义的变量会出现在子类之前（父类在子类之前加载）
- 如果CompactFields`-XX:CompactFields`参数为true（默认为true）：子类的窄变量可以插入到父类变量的空隙
  - CompactFields选项参数在JDK14中以被标记为过期了

#### 对齐填充
占位符，为了对齐长度补充占位符   

#### 总结示例

```java
public class Customer{
    int id = 1001;
    String name;
    Account acct;

    {
        name = "匿名客户";
    }
    
    public Customer(){
        acct = new Account();
    }

}

public class ObjectTest {
    public static void main(String[] args) {
        Customer cust = new Object();
    }
}
```

内存占位如下

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/创建对象的内存布局.png)


#### 对象的访问定位
P106  略
- 句柄访问
- 直接指针

## 直接内存(Direct Memory)

### 直接内存概述
- 直接内存不是虚拟机运行时数据区的一部分，也不是《Java虚拟机规范》中定义的内存区域
- 直接内存是在Java堆外的、直接向系统申请的内存区间
- 来源于NIO，通过存在堆中的DirectByteBuffer操作Native内存
- 通常，访问直接内存的速度会优于Java堆。即读写性能高
  - 因此出于性能考虑，**读写频繁的场合**可能会考虑使用直接内存
  - Java的NIO库允许Java程序使用直接内存，用于数据缓冲区

> 在JDK 1.4中新加入了NIO（New Input/Output）类，引入了一种基于通道（Channel）与缓冲区（Buffer）的I/O方式，它可以使用Native函数库直接分配堆外内存，然后通过一个存储在Java堆里面的`DirectByteBuffer`对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为**避免了在Java堆和Native堆中来回复制数据** 

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/直接内存的优化图示.png)



### 直接内存的OOM
- 直接内存也可能导致OutofMemoryError异常`java.lang.OutOfMemoryError: Direct buffer memory`
- 由于直接内存在Java堆外，因此它的大小不会直接受限于`-Xmx`指定的最大堆大小，但是系统内存是有限的，Java堆和直接内存的总和依然**受限于操作系统能给出的最大内存**
- JDK8 中`元空间`直接使用本地内存
- java程序进程所占的`内存空间 = 本地内存(元空间+直接内存) + 堆空间`

直接内存的缺点为：   
- 分配回收成本较高
- 不受JVM内存回收管理
- 直接内存大小可以通过`MaxDirectMemorySize`设置`-Xmx20m -XX:MaxDirectMemorySize=10m`
- 如果不指定，默认与堆的最大值`-Xmx`参数值一致


# 四、执行引擎
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/执行引擎图例.png)

执行引擎是Java虚拟机核心的组成部分之一  
执行引擎（Execution Engine）的任务就是**将字节码指令解释/编译为对应平台上的本地机器指令**才可以。简单来说，**JVM中的执行引擎就充当了将高级语言翻译为机器语言的译者**

> 这里的编译不是将java编译成字节码文件，而是将字节码编译成对应平台的机器指令 

## 执行引擎的工作流程
- 执行引擎在执行的过程中究竟需要执行什么样的字节码指令完全依赖于`PC寄存器`
- 每当执行完一项指令操作后，PC寄存器就会更新下一条需要被执行的指令地址
- 当然方法在执行的过程中，执行引擎有可能会通过存储在局部变量表中的对象引用准确定位到存储在Java堆区中的对象实例信息，以及通过对象头中的元数据指针定位到目标对象的类型信息  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/执行引擎执行过程.png)

## Java代码编译和执行过程
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/解释执行和编译执行.png)

大部分的程序代码转换成物理机的目标代码(机器指令)或虚拟机能执行的指令集之前，都需要经过上图中的各个步骤    
说明：绿色部分是解释执行的过程，蓝色部分是编译执行的过程

## 解释器（Interpreter）和JIT编译器
见《深入理解Java虚拟机》11章

- `解释器`：当Java虚拟机启动时会根据预定义的规范对字节码采用逐行解释的方式执行，将每条字节码文件中的内容“翻译”为对应平台的本地机器指令执行
- JIT`Just In Time Compiler`编译器：就是虚拟机将源代码直接编译成和本地机器平台相关的机器语言

> **为什么说Java是半编译型半解释型语言？**
> JDK1.0时代，将Java语言定位为`解释执行`还是比较准确的。再后来，Java也发展出可以**直接生成本地代码的编译器**   
> 现在JVM在执行Java代码的时候，通常都会将解释执行与编译执行二者结合起来进行,效率更高

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/编译解释过程示例.png)


### 解释器 

- 解释器真正意义上所承担的角色就是一个运行时“翻译者”，将字节码文件中的内容“翻译”为对应平台的本地机器指令执行
- 当一条字节码指令被解释执行完成后，接着再根据PC寄存器中记录的下一条需要被执行的字节码指令执行解释操作  

### JIT编译器
即时编译技术（JIT，Just In Time）将代码编译成机器码后再执行

> `HotSpot VM`是目前市面上高性能虚拟机的代表作之一。它采用解释器与即时编译器并存的架构。在Java虚拟机运行时，**解释器和即时编译器能够相互协作**，各自取长补短，尽力去选择最合适的方式来权衡编译本地代码的时间和直接解释执行代码的时间  

解释器与编译器两者各有优势：   
- 当程序需要迅速启动和执行的时候，解释器可以首先发挥作用，省去编译的时间，立即运行。
- 当程序启动后，随着时间的推移，编译器逐渐发挥作用，把越来越多的代码编译成本地代码，这样可以减少解释器的中间损耗，获得更高的执行效率。

### HotSpot JVM的执行方式  
HotSpot虚拟机中内置了两个（或三个）即时编译器，其中有两个编译器存在已久，分别被称为“客户端编译器”（Client Compiler）和“服务端编译器”（Server Compiler），或者简称为C1编译器和C2编译器（部分资料和JDK源码中C2也叫Opto编译器），第三个是在JDK 10时才出现的、长期目标是代替C2的Graal编译器。

# 五、StringTable

## 基本特性
- String声明为final的，不可被继承
- String实现了Serializable接口：表示字符串是支持序列化的
- String实现了Comparable接口：表示string可以比较大小
- String在jdk8及以前内部定义了`final char[ ] value`用于存储字符串数据。JDK9时改为`byte[ ]`


## String在jdk9中存储结构变更
String再也不用`char[ ] `来存储了，改成了`byte[ ]`加上`编码标记`，节约了一些空间   
这纯粹是一个实现更改，**对现有的公共接口没有任何更改。没有计划添加任何新的公共 API 或其他接口**。
迄今为止完成的原型设计工作证实了**内存占用的预期减少**、**GC 活动的大幅减少**以及在某些极端情况下的轻微性能回归。

## String的基本特性
- String代表不可变的字符序列。简称：不可变性
  - 当对字符串重新赋值时，需要重写指定内存区域赋值，不能使用原有的value进行赋值
  - 当对现有的字符串进行连接操作时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值
  - 当调用string的replace( )方法修改指定字符或字符串时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值
- 通过字面量的方式(区别于new)给一个字符串赋值`String s = "123"`，此时的字符串值声明在字符串常量池中
- 字符串常量池是不会存储相同内容的字符串的
  - String的String Pool是一个固定大小的Hashtable，默认值大小长度是1009。如果放进String Pool的String非常多，就会造成Hash冲突严重，从而导致链表会很长，而链表长了后直接会造成的影响就是当调用String.intern时性能会大幅下降
  - 使用`-XX:StringTablesize`可设置StringTable的长度
  - 在JDK6中StringTable是固定的，就是1009的长度，所以如果常量池中的字符串过多就会导致效率下降很快。StringTablesize设置没有要求
  - 在JDK7中，StringTable的长度默认值是60013，StringTablesize设置没有要求
  - 在JDK8开始，设置StringTable长度的话，1009是可以设置的最小值`介于1009和2305843009213693951之间`


## String的内存分配
- 在Java语言中有8种基本数据类型和一种比较特殊的类型String。这些类型为了使它们在运行过程中速度更快、更节省内存，都提供了一种常量池的概念
- 常量池就类似一个Java系统级别提供的缓存。8种基本数据类型的常量池都是系统协调的，String类型的常量池比较特殊。它的主要使用方法有两种
  - 直接使用双引号声明出来的String对象会直接存储在常量池中`String info = "baidu.com"`
  - 如果不是用双引号声明的String对象，可以使用String提供的`intern( )`方法。
- Java 6及以前，字符串常量池存放在永久代
- Java 7中将字符串常量池的位置调整到Java堆内
- Java8元空间，字符串常量在堆空间中

> `StringTable`为什么要调整？   
> - `permSize`默认比较小    
> - 永久代垃圾回收频率低  

## 字符串拼接操作
- 常量与常量的拼接结果在常量池，原理是编译期优化
- 常量池中不会存在相同内容的变量
- 只要其中有一个是变量，结果就在堆中。变量拼接的原理是`StringBuilder`
- 如果拼接的结果调用`intern( )`方法，则主动将常量池中还没有的字符串对象放入池中，并返回此对象地址

```java
	@Test
    public void test1(){
        String s1 = "a" + "b" + "c"; //编译期优化：等同于"abc"
        String s2 = "abc"; //"abc"一定是放在字符串常量池中，将此地址赋给s2
        /*
         * 最终.java编译成.class,再执行.class
         * String s1 = "abc";
         * String s2 = "abc"
         */
        System.out.println(s1 == s2); //true
        System.out.println(s1.equals(s2)); //true
    }

```

```java
	@Test
    public void test2(){
        String s1 = "javaEE";
        String s2 = "hadoop";

        String s3 = "javaEEhadoop";
        String s4 = "javaEE" + "hadoop";//编译期优化
        //如果拼接符号的前后出现了变量，则相当于在堆空间中new String()，具体的内容为拼接的结果：javaEEhadoop
        String s5 = s1 + "hadoop";
        String s6 = "javaEE" + s2;
        String s7 = s1 + s2;

        System.out.println(s3 == s4);//true
        System.out.println(s3 == s5);//false
        System.out.println(s3 == s6);//false
        System.out.println(s3 == s7);//false
        System.out.println(s5 == s6);//false
        System.out.println(s5 == s7);//false
        System.out.println(s6 == s7);//false
        //intern():判断字符串常量池中是否存在javaEEhadoop值，如果存在，则返回常量池中javaEEhadoop的地址；
        //如果字符串常量池中不存在javaEEhadoop，则在常量池中加载一份javaEEhadoop，并返回此对象的地址。
        String s8 = s6.intern();
        System.out.println(s3 == s8);//true
    }

```

```java
	@Test
    public void test3(){
        String s1 = "a";
        String s2 = "b";
        String s3 = "ab";
        /*
        如下的s1 + s2 的执行细节：(变量s是我临时定义的）
        ① StringBuilder s = new StringBuilder();
        ② s.append("a")
        ③ s.append("b")
        ④ s.toString()  --> 约等于 new String("ab")

        补充：在jdk5.0之后使用的是StringBuilder,在jdk5.0之前使用的是StringBuffer
         */
        String s4 = s1 + s2;//
        System.out.println(s3 == s4);//false
    }

```

> 在jdk5.0之后使用的是StringBuilder,在jdk5.0之前使用的是StringBuffer

```java
		/*
    1. 字符串拼接操作不一定使用的是StringBuilder!
       如果拼接符号左右两边都是字符串常量或常量引用，则仍然使用编译期优化，即非StringBuilder的方式。
    2. 针对于final修饰类、方法、基本数据类型、引用数据类型的量的结构时，能使用上final的时候建议使用上。
     */
    @Test
    public void test4(){
        final String s1 = "a";
        final String s2 = "b";
        String s3 = "ab";
        String s4 = s1 + s2; //s4:常量
        System.out.println(s3 == s4);//true
    }

```

### 拼接效率
通过StringBuilder的append()的方式添加字符串的效率要远高于使用String的字符串拼接方式！
- StringBuilder的append()的方式：自始至终中只创建过一个StringBuilder的对象
- 使用String的字符串拼接方式：创建过多个StringBuilder和String的对象
  - 内存中由于创建了较多的StringBuilder和String的对象，内存占用更大；如果进行GC，需要花费额外的时间。
- 在实际开发中，如果基本确定要前前后后添加的字符串长度不高于某个限定值highLevel的情况下,建议使用构造器实例化：
  - `StringBuilder s = new StringBuilder(highLevel);` highLevel长度

## intern()

### 概述

当调用intern方法时，如果字符串常量池里已经包含了一个与这个String对象相等的字符串(equeals()方法返回true)，那么常量池里的字符串会被返回。否则，这个String对象被添加到池中，并返回这个String对象的引用。

- 如果在当前类的常量池中**存在**与调用intern()方法的字符串等值的字符串(equals判断)，就直接返回常量池中相应字符串的引用 
- 否则
  - JDK6在常量池中复制一份该字符串，并将其引用返回；
  - JDK7+中会直接在常量池中保存当前字符串的引用地址(引用返回的是该字符串对象地址)


- `intern()` 是一个native方法，调用的是底层 C 的方法。
  - `public native String intern();`
- 如果不是用双引号声明的String对象，可以使用String提供的intern方法，它会从字符串常量池中查询当前字符串是否存在，若不存在就会将当前字符串放入常量池中
  - `String myInfo = new string("I love alibaba").intern();`
- 如果在任意字符串上调用`String.intern`方法，那么其返回结果所指向的那个类实例，必须和直接以常量形式出现的字符串实例完全相同。因此，下列表达式的值必定是true
  - `("a"+"b"+"c").intern() == "abc"`

?> `Interned string`就是确保字符串在内存里只有一份拷贝，这样可以节约内存空间，加快字符串操作任务的执行速度。注意，这个值会被存放在字符串内部池`String Intern Pool`

> 如何保证变量s指向的是字符串常量池中的数据呢？    
> - 通过字面量定义`String s = "123"`
> - 调用`intern()` `String s = new String("shkstart").intern();` `String s = new StringBuilder("shkstart").toString().intern();`


### 面试题
1. `new String("ab")`会创建几个对象?2个   
- 一个对象是：new 关键字在堆空间中创建
- 另一个对象：字符串常量池中的对象`ab`

```java
public class StringNewTest {
    public static void main(String[] args) {
        String str = new String("ab");
    }
}
```

得到字节码解析

```
 0 new #2 <java/lang/String> //创建String 对象
 3 dup
 4 ldc #3 <av> 字符串常量池中创建对象
 6 invokespecial #4 <java/lang/String.<init>> //构造器初始化
 9 astore_1
10 return
```

2. `String s = new String("a") + new String("b");`创建几个对象？ 6个   
- StringBuilder
- new String 
- 常量池中的a
- new String
- 常量池中的b
- 最终的由StringBuilder 转为String的`toString()`方法,此方法实际调用`String(char value[], int offset, int count)` 
  - **此时字符串常量池中没有"ab"**

```
 0 new #2 <java/lang/StringBuilder> //1
 3 dup
 4 invokespecial #3 <java/lang/StringBuilder.<init>> 
 7 new #4 <java/lang/String> //2
10 dup
11 ldc #5 <a>  //3
13 invokespecial #6 <java/lang/String.<init>>
16 invokevirtual #7 <java/lang/StringBuilder.append>
19 new #4 <java/lang/String>  //4
22 dup
23 ldc #8 <b>  //5
25 invokespecial #6 <java/lang/String.<init>>
28 invokevirtual #7 <java/lang/StringBuilder.append>
31 invokevirtual #9 <java/lang/StringBuilder.toString>  //6
34 astore_1
35 return
```

3. 下面代码的输出

```java
public class StringIntern {

    public static void main(String[] args) {

        /**
         * ① String s = new String("1")
         * 创建了两个对象
         * 		堆空间中一个new对象
         * 		字符串常量池中一个字符串常量"1"（注意：此时字符串常量池中已有"1"）
         * ② s.intern()由于字符串常量池中已存在"1"
         *
         * s  指向的是堆空间中的对象地址
         * s2 指向的是堆空间中常量池中"1"的地址
         * 所以不相等
         */
        String s = new String("1");//这里字符串常量池已经创建了 1 
        String s1 = s.intern();//调用此方法之前，字符串常量池中已经存在了"1"
        String s2 = "1";
        System.out.println(s == s2);//jdk6：false   jdk7/8：false
        System.out.println(s1 == s);//false
        System.out.println(s1 == s2);//true

        /**
         * ① String s3 = new String("1") + new String("1")
         * 等价于new String（"11"），但是，常量池中并不生成字符串"11"；
         *
         * ② s3.intern()
         * 由于此时常量池中并无"11"，所以把s3中记录的对象的地址存入常量池
         * 所以s3 和 s4 指向的都是一个地址
         */
        String s3 = new String("1") + new String("1");//s3变量记录的地址为：new String("11")
        //执行完上一行代码以后，字符串常量池中，是否存在"11"呢？答案：不存在！！
        s3.intern();//在字符串常量池中生成"11"。如何理解：jdk6:在常量池中真正创建了一个新的对象"11",也就有新的地址。
                                            //         jdk7:此时常量池中并没有真正创建"11",而是创建一个指向堆空间中new String("11")的地址
        String s4 = "11";//s4变量记录的地址：使用的是上一行代码代码执行时，在常量池中生成的"11"的地址
        System.out.println(s3 == s4);//jdk6：false  jdk7/8：true
    }
}
```

- JDK1.6中，将这个字符串对象尝试放入字符串常量池中。
  - 如果字符串常量池中有，则并不会放入。返回已有的字符串常量池中的对象的地址
  - 如果没有，会把此对象复制一份，放入字符串常量池，并返回字符串常量池中的对象地址
- JDK1.7起，将这个字符串对象尝试放入字符串常量池中。
  - 如果字符串常量池中有，则并不会放入。返回已有的字符串常量池中的对象的地址
  - 如果没有，则会把对象的引用地址复制一份，放入字符串常量池，并返回字符串常量池中的引用地址

### intern效率
对于程序中大量使用存在的字符串时，尤其存在很多已经重复的字符串时，使用`intern( )`方法能够节省内存空间。

大的网站平台，需要内存中存储大量的字符串。比如社交网站，很多人都存储：北京市、海淀区等信息。这时候如果字符串都调用`intern( )`方法，就会很明显降低内存的大小。

### G1中的String去重操作  
测量表明，在这些类型的应用程序中，`大约25%的Java堆实时数据集被String'对象所消耗`。此外，这些 "String "对象中**大约有一半是重复的**，其中重复意味着 "string1.equals(string2)=true "。**堆上存在重复的String对象必然是一种内存的浪费**

> 注意这里说的重复，指的是在堆中的数据，而不是常量池中的，因为常量池中的本身就不会重复

- 当垃圾收集器工作的时候，会访问堆上存活的对象。对每一个访问的对象都会检查是否是候选的要去重的String对象
- 如果是，把这个对象的一个引用插入到队列中等待后续的处理。一个去重的线程在后台运行，处理这个队列。处理队列的一个元素意味着从队列删除这个元素，然后尝试去重它引用的string对象。
- 使用一个hashtable来记录所有的被String对象使用的不重复的char数组。当去重的时候，会查这个hashtable，来看堆上是否已经存在一个一模一样的char数组。
- 如果存在，String对象会被调整引用那个数组，释放对原来的数组的引用，最终会被垃圾收集器回收掉。
- 如果查找失败，char数组会被插入到hashtable，这样以后的时候就可以共享这个数组了。

# 六、垃圾回收概念

## 概述
### 什么是垃圾
- 垃圾是指在运行程序中没有任何指针指向的对象，这个对象就是需要被回收的垃圾。
- An object is considered garbage when it can no longer be reached from any pointer in the running program

> 如果不及时对内存中的垃圾进行清理，那么，这些垃圾对象所占的内存空间会一直保留到应用程序结束，被保留的空间无法被其它对象使用，甚至可能导致内存溢出。


### 为什么需要GC
- 对于高级语言来说，一个基本认知是如果不进行垃圾回收，内存迟早都会被消耗完，因为不断地分配内存空间而不进行回收，就好像不停地生产生活垃圾而从来不打扫一样。
- 除了释放没用的对象，垃圾回收也可以清除内存里的记录碎片。碎片整理将所占用的堆内存移到堆的一端，以便JVM将整理出的内存分配给新的对象。
- 随着应用程序所应付的业务越来越庞大、复杂，用户越来越多，没有GC就不能保证应用程序的正常进行。而经常造成STW的GC又跟不上实际的需求，所以才会不断地尝试对GC进行优化。

### 回收机制
**自动化的内存分配和来及回收方式已经成为了线代开发语言必备的标准**     

降低了内存泄漏和内存溢出的风险

### GC主要关注的区域
**GC主要关注于 方法区 和 堆 中的垃圾收集**

- 垃圾收集器可以对年轻代回收，也可以对老年代回收，甚至是全栈和方法区的回收。
  - 其中，Java堆是垃圾收集器的工作重点
- 从次数上讲：
  - 频繁收集Young区
  - 较少收集Old区
  - 基本不收集Perm区（元空间）


## 标记阶段
> 在堆里存放着几乎所有的Java对象实例，在GC执行垃圾回收之前，首先需要区分出内存中哪些是存活对象，哪些是已经死亡的对象。只有被标记为己经死亡的对象，GC才会在执行垃圾回收时，释放掉其所占用的内存空间，因此这个过程我们可以称为**垃圾标记阶段**。

> 那么在JVM中究竟是如何标记一个死亡对象呢？简单来说，当一个对象已经不再被任何的存活对象继续引用时，就可以宣判为已经死亡。
> 判断对象存活一般有两种方式：`引用计数算法`和`可达性分析算法`


### 引用计数算法

> 在对象中添加一个引用计数器，每当有一个地方引用它时，计数器值就加一；当引用失效时，计数器值就减一；任何时刻计数器为零的对象就是不可能再被使用的

`引用计数算法(Reference Counting)`虽然占用了一些额外的内存空间来进行计数，但它的**原理简单，判定效率也很高**，在大多数情况下它都是一个不错的算法

> 主流的Java虚拟机里面都没有选用引用计数算法来管理内存，主要原因是，这个看似简单的算法有很多例外情况要考虑，必须要配合大量额外处理才能保证正确地工作，譬如单纯的引用计数就很难解决对象之间相互`循环引用`的问题

> 当p的指针断开的时候，内部的引用形成一个循环，这就是循环引用

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/循环引用示例.png)


### 可达性分析算法和GC Roots
可达性分析算法又称根搜索算法、追踪性垃圾收集  
 
当前主流的商用程序语言`Java、C#`的内存管理子系统，都是通过可达性分析`Reachability Analysis`算法来**判定对象是否存活**的。      

这个算法的基本思路就是通过一系列称为`GC Roots`的根对象作为起始节点集，从这些节点开始，根据引用关系向下搜索，搜索过程所走过的路径称为`引用链`(Reference Chain)，如果某个对象到`GC Roots`间没有任何`引用链`相连，或者用图论的话来说就是从GC Roots到这个对象不可达时，则证明此对象是不可能再被使用的

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/可达性分析算法的GCRoot.png)

?> 可以有效地解决在引用计数算法中循环引用的问题，防止内存泄漏的发生

**GC Roots包括：**   
- 在虚拟机栈（栈帧中的本地变量表）中引用的对象
  - 各个线程被调用的方法堆栈中使用到的参数、局部变量、临时变量等。
- 在方法区中类静态属性引用的对象
  - Java类的引用类型静态变量。
- 在方法区中常量引用的对象
  - 字符串常量池`String Table`里的引用。
- 在本地方法栈中JNI(即通常所说的Native方法)引用的对象。
- Java虚拟机内部的引用，如基本数据类型对应的Class对象，一些常驻的异常对象(比如`NullPointExcepiton`、`OutOfMemoryError`)等，还有系统类加载器
- 所有被同步锁`synchronized关键字`持有的对象。
- 反映Java虚拟机内部情况的`JMXBean`、`JVMTI`中注册的回调、本地代码缓存等  
- 除了这些固定的GC Roots集合以外，根据用户所选用的垃圾收集器以及当前回收的内存区域不同，还可以有其他对象“临时性”地加入，共同构成完整GC Roots集合。比如：分代收集和局部回收（PartialGC）。
  - 如果只针对Java堆中的某一块区域进行垃圾回收（比如：典型的只针对新生代），必须考虑到内存区域是虚拟机自己的实现细节，更不是孤立封闭的，这个区域的对象完全有可能被其他区域的对象所引用，这时候就需要一并将关联的区域对象也加入GCRoots集合中去考虑，才能保证可达性分析的准确性。
  - 典型的只针对新生代：因为新生代除外，还有关联的老年代，所以需要将老年代也一并加入GC Roots集合中

**关于STW的说明**   
- 如果要**使用可达性分析算法来判断内存是否可回收**，那么分析工作**必须在一个能保障一致性(某一刻的静止状态)的快照中进行**。这点不满足的话分析结果的准确性就无法保证。
- 这点也是导致GC进行时必须`stop The World`的一个重要原因。
- 即使是号称（几乎）不会发生停顿的CMS收集器中，枚举根节点时也是必须要停顿的

### 对象的finalization机制
- Java语言提供了对象终止（finalization）机制来允许开发人员提供对象被销毁之前的自定义处理逻辑。
- 当垃圾回收器发现没有引用指向一个对象，即：垃圾回收此对象之前，总会先调用这个对象的`finalize()`方法。
- `finalize()` 方法允许在子类中被重写，用于在对象被回收时进行资源释放。通常在这个方法中进行一些资源释放和清理的工作，比如**关闭文件、套接字和数据库连接**等。
- 永远不要主动调用某个对象的`finalize()`方法，应该交给垃圾回收机制调用。理由包括下面三点：
  - 在`finalize( )`执行时可能会导致对象复活。
  - `finalize( )`方法的执行时间是没有保障的，它完全由GC线程决定，极端情况下，若不发生GC，则`finalize( )`方法将没有执行机会。
  - 一个糟糕的`finalize( )`会严重影响GC的性能。
- 由于finalize()方法的存在，虚拟机中的对象一般处于`三种`可能的状态

> 如果从所有的根节点都无法访问到某个对象，说明对象己经不再使用了。一般来说，此对象需要被回收。但事实上，也并非是“非死不可”的，这时候它们暂时处于“缓刑”阶段。`一个无法触及的对象有可能在某一个条件下“复活”自己`，如果这样，那么对它的回收就是不合理的，为此，定义虚拟机中的对象可能的三种状态。

- `可触及的`：从根节点开始，可以到达这个对象(不是垃圾)。
- `可复活的`：对象的所有引用都被释放，但是对象有可能在 `finalize()` 中复活。
- `不可触及的`：对象的 `finalize()` 被调用，并且没有复活，那么就会进入不可触及状态。不可触及的对象不可能被复活，因为 `finalize()` 只会被调用一次。

### 判定过程
- 判定一个`对象 objA` 是否可回收，至少要经历**两次标记**过程：
  - 如果对象 objA 到 GC Roots 没有引用链，则进行第一次标记。
  - 进行筛选，判断此对象是否有必要执行 finalize() 方法
    - 如果对象 objA 没有重写 finalize() 方法，或者 finalize() 方法已经被虚拟机调用过，则虚拟机视为“没有必要执行”，objA 被判定为不可触及的。
    - 如果对象 objA 重写了 finalize() 方法，且还未执行过，那么 objA 会被插入到 F-Queue 队列中，由一个虚拟机自动创建的、低优先级的 `Finalizer 线程`触发其 finalize() 方法执行。
  - finalize() 方法是对象逃脱死亡的最后机会，稍后 GC 会对 F-Queue 队列中的对象进行第二次标记。如果 objA 在 finalize() 方法中与引用链上的任何一个对象建立了联系，那么在第二次标记时，objA 会被移出“即将回收”集合。之后，对象如果再次出现没有引用存在的情况。在这个情况下， finalize() 方法不会被再次调用，对象会直接变成不可触及的状态，也就是说，一个对象的 finalize() 方法只会被调用一次

## 清除阶段 
当成功区分出内存中存活对象和死亡对象后，GC 接下来的任务就是执行垃圾回收，释放掉无用对象所占用的内存空间，以便有足够的可用内存空间为新对象分配内存。目前在 JVM 中比较常见的三种垃圾收集算法是：  

- 标记一清除算法（Mark-Sweep）
- 复制算法（Copying）
- 标记-压缩算法（Mark-Compact）

### 标记-清除算法(Mark-Sweep)
算法分为“标记”和“清除”两个阶段：
- 首先标记出所有需要回收的对象 
- 在标记完成后，统一回收掉所有被标记的对象
- 也可以反过来，标记存活的对象，统一回收所有未被标记的对象。

> 标记过程就是对象是否属于垃圾的判定过程

当堆中的有效内存空间（Available Memory）被耗尽的时候，就会停止整个程序（也被称为 Stop The World），然后进行两项工作，第一项则是标记，第二项则是清除。   
`标记`：Collector 从引用根节点开始遍历，标记所有被引用的对象。一般是在对象的 Header 中记录为可达对象。标记的是引用的对象
`清除`：Collector 对堆内存从头到尾进行线性的遍历，如果发现某个对象在其 Header 中没有标记为可达对象，则将其回收

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/标记清除算法过程.png)

- 缺点：
  - 标记清除算法的效率不算高
  - 在进行 GC 的时候，需要停止整个应用程序，导致用户体验较差
  - 这种方式清理出来的空闲**内存是不连续的**，产生内存碎片，需要维护一个`空闲列表`

> 这里所谓的清除并不是真的置空，而是把需要清除的对象地址保存在空闲的地址列表里。下次有新对象需要加载时，判断垃圾的位置空间是否够，如果够，就存放覆盖原有的地址  

关于空闲列表是在为对象分配内存的时候：  
- 如果内存规整
  - 采用指针碰撞的方式进行内存分配
- 如果内存不规整
  - 虚拟机需要维护一个空闲列表,通过空闲列表分配

### 复制算法（Copying）
将活着的内存空间分为两块，每次只使用其中一块，在垃圾回收时将正在使用的内存中的存活对象复制到未被使用的内存块中，之后清除正在使用的内存块中的所有对象，交换两个内存的角色，最后完成垃圾回收

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/标记复制算法过程.png)


**优点**  
- 没有标记和清除过程，实现简单，运行高效
- 复制过去以后保证空间的连续性，不会出现“碎片”问题

**缺点**  
- 此算法的缺点也是很明显的，就是**需要两倍的内存空间**。
- 对于 G1 这种拆分成为大量 `region` 的 GC，复制而不是移动，意味着 GC 需要维护 `region` 之间对象的引用关系，不管是内存占用或者时间开销也不小
  - `region`见分区算法
- 如果系统中的存活对象非常多，复制算法可能不会很理想(全部复制)  

> 现在的商用Java虚拟机大多都优先采用了这种收集算法去回收新生代   
> 详细查看《深入理解3.3.3》

### 标记-整理(压缩)算法（Mark - Compact）
`复制算法`的高效性是建立在存活对象少、垃圾对象多的前提下的。这种情况在新生代经常发生   
但是在老年代，更常见的情况是大部分对象都是存活对象。如果依然使用复制算法，由于存活对象较多，复制的成本也将很高。因此，**基于老年代垃圾回收的特性，需要使用其他的算法**

标记清除算法的确可以用在老年代中，但是该算法效率低下，老年代大部分对象都是存活的，内存回收后还会产生内存碎片，碎片过多导致无法存放大的对象

- 第一阶段和标记清除算法一样，从根节点开始标记所有被引用对象
- 第二阶段将所有的存活对象压缩到内存的一端，按顺序排放(移动过程中，需要全程暂停用户应用程序。即：STW)。
- 清理边界外所有的空间。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/标记压缩算法.png)

`标记清除算法`和`标记压缩算法`的本质差异在于标记-清除算法是一种非移动式的回收算法，标记-压缩是移动式的。是否移动回收后的存活对象是一项优缺点并存的风险决策。   
标记的存活对象将会被整理，按照内存地址依次排列，而未被标记的内存会被清理掉。如此一来，当我们需要给新对象分配内存时，JVM 只需要持有一个内存的起始地址即可(标记压缩算法内部使用指针碰撞)，这比维护一个`空闲列表`显然少了许多开销  

### 总结
- 效率上来说，复制算法是当之无愧的老大，但是却浪费了太多内存。
- 而为了尽量兼顾上面提到的三个指标，标记-整理算法相对来说更平滑一些，但是效率上不尽如人意，它比复制算法多了一个标记的阶段，比标记-清除多了一个整理内存的阶段。
- 综合我们可以看到，没有最好的算法，只有最合适的算法


## 分代收集算法
前面所有这些算法中，并没有一种算法可以完全替代其他算法，它们都具有自己独特的优势和特点。分代收集算法应运而生。

> 分代收集算法，是基于这样一个事实：不同的对象的生命周期是不一样的。因此，**不同生命周期的对象可以采取不同的收集方式**，以便提高回收效率。一般是把 Java 堆分为新生代和老年代，这样就可以根据各个年代的特点使用不同的回收算法，以提高垃圾回收的效率。

?> 目前几乎所有的 GC 都采用分代收集算法执行垃圾回收的

在 HotSpot 中，基于分代的概念，GC 所使用的内存回收算法必须结合年轻代和老年代各自的特点。

**年轻代（Young Gen）**   
- 年轻代特点：区域相对老年代较小，对象生命周期短、存活率低，回收频繁。

- 这种情况复制算法的回收整理，速度是最快的。复制算法的效率只和当前存活对象大小有关，因此很适用于年轻代的回收。而复制算法内存利用率不高的问题，通过 HotSpot 中的两个 Survivor 的设计得到缓解。

**老年代（Tenured Gen）**   
- 老年代特点：区域较大，对象生命周期长、存活率高，回收不及年轻代频繁。

- 这种情况存在大量存活率高的对象，复制算法明显变得不合适。一般是由`标记-清除`或者是`标记-清除与标记-整理`的混合实现。
  - Mark(标记)阶段的开销与存活对象的数量成正比。
  - Sweep(清除)阶段的开销与所管理区域的大小成正相关。
  - Compact(整理)阶段的开销与存活对象的数据成正比。

> 以 HotSpot 中的 CMS 回收器为例，CMS 是基于 Mark-Sweep 实现的，对于对象的回收效率很高。而对于碎片问题，CMS 采用基于 Mark-Compact 算法的 Serial Old 回收器作为补偿措施：当内存回收不佳（碎片导致的 Concurrent Mode Failure 时），将采用 Serial Old 执行 Full GC 以达到对老年代内存的整理。

!> 分代的思想被现有的虚拟机广泛使用，几乎所有的垃圾回收器都区分新生代和老年代。

## 增量收集算法

上述现有的算法，在垃圾回收过程中，应用软件将处于一种 Stop the World 的状态。在 Stop the World 状态下，应用程序所有的线程都会挂起，暂停一切正常的工作，等待垃圾回收的完成。如果垃圾回收时间过长，应用程序会被挂起很久，将严重影响用户体验或者系统的稳定性

如果一次性将所有的垃圾进行处理，需要造成系统长时间的停顿，那么就可以让垃圾收集线程和应用程序线程交替执行。每次，**垃圾收集线程只收集一小片区域的内存空间，接着切换到应用程序线程。依次反复，直到垃圾收集完成**

总的来说，**增量收集算法的基础仍是传统的标记-清除和复制算法**。增量收集算法通过对线程间冲突的妥善处理允许垃圾收集线程以分阶段的方式完成标记、清理或复制工作

> 使用这种方式，由于在垃圾回收过程中，间断性地还执行了应用程序代码，所以能减少系统的停顿时间。但是，因为线程切换和上下文转换的消耗，会**使得垃圾回收的总体成本上升，造成系统吞吐量的下降**

## 分区算法
一般来说，在相同条件下，堆空间越大，一次 GC 时所需要的时间就越长，有关 GC 产生的停顿也越长。为了更好地控制 GC 产生的停顿时间，将一块大的内存区域分割成多个小块(region)，根据目标的停顿时间，每次合理地回收若干个小区间，而不是整个堆空间，从而减少一次 GC 所产生的停顿。

?> **分代算法将按照对象的生命周期长短划分成两个部分，分区算法将整个堆空间划分成连续的不同小区间**。  
每一个小区间都独立使用，独立回收。这种算法的好处是可以控制一次回收多少个小区间。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/分区算法示意图.png)

## System.gc()的理解
- 在默认情况下，通过 `System.gc()`或者 `Runtime.getRuntime().gc()` 的调用，会显式触发 Full GC，同时对老年代和新生代进行回收，尝试释放被丢弃对象占用的内存。
  - `System.gc()`实际上调用的就是`Runtime.getRuntime().gc()`
- 然而 `System.gc()` 调用附带一个免责声明，**无法保证对垃圾收集器的调用**。（不能确保立即生效）
- JVM 实现者可以通过 System.gc() 调用来决定 JVM 的 GC 行为。而一般情况下，垃圾回收应该是自动进行的，无须手动触发，否则就太过于麻烦了。在一些特殊情况下，如我们正在编写一个性能基准，我们可以在运行之间调用 System.gc()

```java
public class SystemGCTest {

    public static void main(String[] args) {
        new SystemGCTest();
        System.gc();//提醒jvm的垃圾回收器执行gc,但是不确定是否马上执行gc
        //System.gc()与Runtime.getRuntime().gc() 的作用一样。

        System.runFinalization();//强制调用失去引用对象的finalize()方法
    }

    //GC回收之前会调用finalize()方法
    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("SystemGCTest 重写了finalize()");
    }

}
//执行结果： SystemGCTest 重写了finalize()
```

- 运行程序，不一定会触发垃圾回收，但是调用 `System.runFinalization()` 会强制调用失去引用对象的`finalize( )`方法
- System.gc() 与System.runFinalization() 是一起使用的

## 手动 GC 来理解不可达对象的回收

```java
public class LocalVarGC {

    public void localvarGC1() {
      //  不会回收 full gc后进入老年代
      byte[] buffer = new byte[10 * 1024 * 1024];//10MB
      System.gc();
    }

    public void localvarGC2() {
      //触发Young GC的时候就已经将该对象回收
      byte[] buffer = new byte[10 * 1024 * 1024];
      buffer = null;
      System.gc();
    }

    public void localvarGC3() {
      //不会回收进入老年代，它还存放在局部变量表索引为 1 的槽中
      {
        byte[] buffer = new byte[10 * 1024 * 1024];
      }
      System.gc();
    }

    public void localvarGC4() {
      //会回收buffer,由value代替其在局部变量表中的位置
      {
        byte[] buffer = new byte[10 * 1024 * 1024];
      }
      int value = 10;
      System.gc();
    }

    public void localvarGC5() {
      //未回收进入老年代
      localvarGC1();
      //把老年代中的该对象给回收了
      System.gc();
    }

    public static void main(String[] args) {
        LocalVarGC local = new LocalVarGC();
        local.localvarGC1();
    }

}

```

## 内存溢出与内存泄露

### 内存溢出（OOM）
- 内存溢出相对于内存泄漏来说，尽管更容易被理解，但是同样的，内存溢出也是引发程序崩溃的罪魁祸首之一。
- 由于 GC 一直在发展，所以一般情况下，除非应用程序占用的内存增长速度非常快，造成垃圾回收已经跟不上内存消耗的速度，否则不太容易出现 OOM 的情况。
- 大多数情况下，GC 会进行各种年龄段的垃圾回收，实在不行了就放大招，来一次独占式的 Full GC 操作，这时候会回收大量的内存，供应用程序继续使用。
- JavaDoc 中对 `OutOfMemoryError` 的解释是，**没有空闲内存，并且垃圾收集器也无法提供更多内存**。 

**主要存在几种情况：**
- Java 虚拟机的堆内存设置不够。
  - 比如：可能存在内存泄漏问题；也很有可能就是堆的大小设置不合理，比如我们要处理比较可观的数据量，但是没有显式指定 JVM 堆大小或者指定数值偏小。我们可以通过参数 -Xms 、-Xmx 来调整。
- 代码中创建了大量大对象，并且长时间不能被垃圾收集器收集。（存在被引用） 
- 虚拟机栈和本地方法栈的栈容量不足申请无法扩展
- 方法区和运行时常量池溢出
- 本机直接内存溢出


###  内存泄露 Memory Leak

严格来说，只有对象不会再被程序用到了，但是 GC 又不能回收他们的情况，才叫内存泄漏  
尽管内存泄漏并不会立刻引起程序崩溃，但是一旦发生内存泄漏，程序中的可用内存就会被逐步蚕食，直至耗尽所有内存，最终出现 OutOfMemory 异常，导致程序崩溃   
实际情况可能导致对象的生命周期变长，最终内存占用过多，三分钟回收的对象变为10分钟回收，导致OOM   

> Java 使用可达性分析算法，最上面的数据不可达，就是需要被回收的。后期有一些对象不用了，按道理应该断开引用，但是存在一些链没有断开，从而导致没有办法被回收

**案例：**   
- 单例模式
  - 单例的生命周期和应用程序是一样长的，所以单例程序中，如果持有对外部对象的引用的话，那么这个外部对象是不能被回收的，则会导致内存泄漏的产生。一些提供 close 的资源未关闭导致内存泄漏
- 数据库连接（dataSourse.getConnection()），网络连接（Socket）和 IO 连接必须手动 close，否则是不能被回收的。
- 静态集合类,不断向其中添加对象
- 内部类持有外部类
  - 一个外部类的实例对象的方法返回了一个内部类的实例对象，这个内部类被长期引用，即使那个外部类实例对象不再被使用，由于内部类持有外部类的实例对象，这个外部类将不会被垃圾回收
- 变量不合理的作用域
  - 变量定义作用范围大于实际作用域，类成员变量只在某个方法内部使用，方法结束后没有即使赋null导致内存泄漏
- 改变哈希值
  - 对象存储进入hashset中后修改了某些影响对象hash值的属性(参与计算hash值的属性)，导致hash变化无法取出(删除)原对象，造成内存泄漏
- 缓存泄漏

## STW(Stop The World)
`Stop-The-World`，简称 STW，指的是 GC 事件发生过程中，会产生应用程序(用户线程)的停顿。停顿产生时整个应用程序线程都会被暂停，没有任何响应，有点像卡死的感觉，这个停顿称为 STW。

> 可达性分析算法中枚举根节点（GC Roots）会导致所有 Java 执行线程停顿。

- 被 STW 中断的应用程序线程会在完成 GC 之后恢复，频繁中断会让用户感觉像是网速不快造成电影卡带一样，所以我们需要**减少 STW 的发生**。
- STW 事件和采用哪款 GC 无关，**所有的 GC 都有这个事件**。
- 哪怕是 G1 也不能完全避免 Stop-The-World 情况发生，只能说垃圾回收器越来越优秀，回收效率越来越高，尽可能地缩短了暂停时间。
- STW 是 JVM 在后台自动发起和自动完成的。在用户不可见的情况下，把用户正常的工作线程全部停掉。
- 开发中不要用 `System.gc()`; 会导致 Stop-The-World 的发生。


## 垃圾回收中的并行与并发

- `并发`指的是多个事情，在同一时间段内同时发生了,同一个cpu核心采用时间片轮流执行不同任务(一个时间点只有一个任务执行)
- `并行`指的是多个事情，在同一时间点上同时发生了,多个核心同时执行多个任务


并发和串行，在谈论垃圾收集器的上下文语境中，它们可以解释如下：

- 并行（Parallel）：指多条垃圾收集线程并行工作，但此时用户线程仍处于等待状态。如 ParNew、Parallel Scavenge、Parallel Old；
- 串行（Serial）：相较于并行的概念，单线程执行。如果内存不够，则程序暂停，启动 JVM 垃圾回收器进行垃圾回收。回收完，再启动程序的线程。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/并行垃圾回收和串行.png)

- 并发（Concurrent）：指用户线程与垃圾收集线程同时执行（但不一定是并行的，可能会交替执行），垃圾回收线程在执行时不会停顿用户程序的运行。
  - 用户程序在继续运行，而垃圾收集程序线程运行于另一个 CPU 上，如：CMS、G1


## 安全点SafePoint 、安全区域 SafeRegion

## 安全点
程序执行时并非在所有地方都能停顿下来开始 GC，只有在特定的位置才能停顿下来开始 GC，这些位置称为`安全点（SafePoint）`

`SafePoint` 的选择很重要，如果太少可能导致 GC 等待的时间太长，如果太频繁可能导致运行时的性能问题。大部分指令的执行时间都非常短暂，通常会根据“是否具有让程序长时间执行的特征”为标准。比如：选择一些执行时间较长的指令作为 Safe Point，如方法调用、循环跳转和异常跳转等

### 安全点方案
抢先式中断（Preemptive Suspension）和主动式中断（Voluntary Suspension）   

>` 抢先式中断`不需要线程的执行代码主动去配合，在垃圾收集发生时，系统首先把所有用户线程全部中断，如果发现有用户线程中断的地方不在安全点上，就恢复这条线程执行，让它一会再重新中断，直到跑到安全点上。现在几乎没有虚拟机实现采用抢先式中断来暂停线程响应GC事件

> `主动式中断`的思想是当垃圾收集需要中断线程的时候，不直接对线程操作，仅仅简单地设置一个标志位，各个线程执行过程时会不停地主动去轮询这个标志，一旦发现中断标志为真时就自己在最近的安全点上主动中断挂起


### 安全区域 
安全区域是指能够确保在某一段代码片段之中，引用关系不会发生变化，因此，在这个区域中任意地方开始垃圾收集都是安全的。我们也可以把安全区域看作被扩展拉伸了的安全点


## 引用
当内存空间还足够时，能保留在内存之中，如果内存空间在进行垃圾收集后仍然非常紧张，那就可以抛弃这些对象  

在 JDK 1.2 版之后，Java 对引用的概念进行了扩充，将引用分为：   
- 强引用`Strong Reference`
- 软引用`Soft Reference`
- 弱引用`Weak Reference`
- 虚引用`Phantom Reference`
这 4 种引用强度依次逐渐减弱。除强引用外，其他 3 种引用均可以在 `java.lang.ref` 包中找到它们的身影。如下图，显示了这 3 种引用类型对应的类，开发人员可以在应用程序中直接使用它们

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/Reference类的子类.png)

> `FinalReference`终结器引用，它用于实现对象的 finalize() 方法，也可以称为终结器引用   
> 在 GC 时，终结器引用入队。由 `Finalizer` 线程通过终结器引用找到被引用对象调用它的 `finalize()` 方法，第二次 GC 时才回收被引用的对象

- 强引用是最传统的引用的定义，是指在程序代码之中普遍存在的引用赋值，即类似`Objectobj=new Object()`这种引用关系。无论任何情况下，**只要强引用关系还存在，垃圾收集器就永远不会回收掉被引用的对象**。
- 软引用是用来描述一些还有用，但非必须的对象。只被软引用关联着的对象，在**系统将要发生内存溢出异常前，会把这些对象列进回收范围之中进行第二次回收**，如果这次回收还没有足够的内存，才会抛出内存溢出异常。在JDK 1.2版之后提供了`SoftReference`类来实现软引用。
- 弱引用也是用来描述那些非必须对象，但是它的强度比软引用更弱一些，**被弱引用关联的对象只能生存到下一次垃圾收集发生为止**。当垃圾收集器开始工作，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。在JDK 1.2版之后提供了`WeakReference`类来实现弱引用。
- 虚引用也称为“幽灵引用”或者“幻影引用”，它是最弱的一种引用关系。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。**为一个对象设置虚引用关联的唯一目的只是为了能在这个对象被收集器回收时收到一个系统通知**。在JDK 1.2版之后提供了`PhantomReference`类来实现虚引用


### 强引用（Strong Reference）不回收
- 在 Java 程序中，最常见的引用类型是强引用（普通系统 99% 以上都是强引用），也就是我们最常见的普通对象引用，也是**默认的引用类型**。
- 当在 Java 语言中使用 new 操作符创建一个新的对象，并将其赋值给一个变量的时候，这个变量就成为指向该对象的一个强引用。
- 强引用的对象是可触及的，垃圾收集器就永远不会回收掉被引用的对象。
- 对于一个普通的对象，如果没有其他的引用关系，只要超过了引用的作用域或者显式地将相应（强）引用赋值为 null，就是可以当做垃圾被收集了，当然具体回收时机还是要看垃圾收集策略。
- 相对的，软引用、弱引用和虚引用的对象是软可触及、弱可触及和虚可触及的，在一定条件下，都是可以被回收的。所以，**强引用是造成 Java 内存泄漏的主要原因之一**。

```java
/**
 * 强引用的测试
 */
public class StrongReferenceTest {
    public static void main(String[] args) {
        StringBuffer str = new StringBuffer("Hello,阿里巴巴");
        StringBuffer str1 = str;

        str = null;
        System.gc(); //调用垃圾回收

        try {
            Thread.sleep(3000); //设置3秒钟延迟保证GC垃圾回收能够实现
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(str1);
    }
}
```

局部变量 str 指向 StringBuffer 实例所在堆空间，通过 str 可以操作该实例，那么 str 就是 StringBuffer 实例的强引用  
StringBuffer str1 = str;也将str1 指向上面的堆空间

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/强引用代码内存结构示例.png)

那么我们将 str = null; 则原来堆中的对象也不会被回收，因为还有其它对象指向该区域

### 软引用（Soft Reference） 内存不足即回收
- 软引用是用来描述一些还有用，但非必需的对象。**只被软引用关联着的对象，在系统将要发生内存溢出异常前，会把这些对象列进回收范围之中进行第二次回收，如果这次回收还没有足够的内存**，才会抛出内存溢出异常。
- 软引用通常用来实现内存敏感的缓存。比如：高速缓存就有用到软引用。如果还有空闲内存，就可以暂时保留缓存，当内存不足时清理掉，这样就保证了使用缓存的同时，不会耗尽内存。
- 垃圾回收器在某个时刻决定回收软可达的对象的时候，会清理软引用，并可选地把引用存放到一个引用队列（Reference Queue）。
- 类似弱引用，只不过 Java 虚拟机会尽量让软引用的存活时间长一些，迫不得已才清理。

> 当内存足够时，不会回收软引用的可达对象；内存不够时，才会回收软引用的可达对象。

```java
// 声明强引用
Object obj = new Object();
// 创建一个软引用
SoftReference<Object> sf = new SoftReference<>(obj);
obj = null; //销毁强引用，这个操作是必须的，不然会存在强引用和软引用
```

```java
/**
 * 软引用的测试：内存不足即回收
 */
public class SoftReferenceTest {

    public static class User {
        public User(int id, String name) {
            this.id = id;
            this.name = name;
        }

        public int id;
        public String name;

        @Override
        public String toString() {
            return "[id=" + id + ", name=" + name + "] ";
        }
    }

    public static void main(String[] args) {
        //创建对象，建立软引用
//        SoftReference<User> userSoftRef = new SoftReference<User>(new User(1, "songhk"));
        //上面的一行代码，等价于如下的三行代码
        User u1 = new User(1, "songhk");
        SoftReference<User> userSoftRef = new SoftReference<User>(u1);
        u1 = null;//取消强引用

        //从软引用中重新获得强引用对象
        System.out.println(userSoftRef.get()); //[id=1, name=songhk]

        System.gc();
        System.out.println("After GC:");
//        //垃圾回收之后获得软引用中的对象
        System.out.println(userSoftRef.get()); //[id=1, name=songhk] 由于堆空间内存足够，所有不会回收软引用的可达对象。
//
        try {
            //让系统认为内存资源不够
//            byte[] b = new byte[1024 * 1024 * 7]; //会报OOM，软引用为null
            //让系统认为内存资源紧张
            byte[] b = new byte[1024 * 7168 - 350 * 1024]; //不会报OOM，软引用为null
        } catch (Throwable e) {
            e.printStackTrace();
        } finally {
            //再次从软引用中获取数据
            System.out.println(userSoftRef.get());//在报OOM之前，垃圾回收器会回收软引用的可达对象。
        }
    }
}

```

### 弱引用（Weak Reference）发现即回收

- 弱引用也是用来描述那些非必需对象，只被弱引用关联的对象只能生存到下一次垃圾收集发生为止。**在系统 GC 时，只要发现弱引用，不管系统堆空间使用是否充足，都会回收掉只被弱引用关联的对象**。
- 由于垃圾回收器的线程通常优先级很低，因此，并不一定能很快地发现持有弱引用的对象。在这种情况下，弱引用对象可以存在较长的时间 
- 弱引用和软引用一样，在构造弱引用时，也可以指定一个引用队列，当弱引用对象被回收时，就会加入指定的引用队列，通过这个队列可以跟踪对象的回收情况。
- 软引用、弱引用都非常适合来保存那些可有可无的缓存数据。如果这么做，当系统内存不足时，这些缓存数据会被回收，不会导致内存溢出。而当内存资源充足时，这些缓存数据又可以存在相当长的时间，从而起到加速系统的作用。

```java
/**
 * 弱引用的测试
 */
public class WeakReferenceTest {
    public static class User {
        public User(int id, String name) {
            this.id = id;
            this.name = name;
        }

        public int id;
        public String name;

        @Override
        public String toString() {
            return "[id=" + id + ", name=" + name + "] ";
        }
    }

    public static void main(String[] args) {
        //构造了弱引用
        WeakReference<User> userWeakRef = new WeakReference<User>(new User(1, "songhk"));
        //从弱引用中重新获取对象
        System.out.println(userWeakRef.get()); // [id=1, name=songhk]

        System.gc();
        // 不管当前内存空间足够与否，都会回收它的内存
        System.out.println("After GC:");
        //重新尝试从弱引用中获取对象
        System.out.println(userWeakRef.get()); //null
    }
}

```

### 虚引用（Phantom Reference ）对象回收跟踪
- 一个对象是否有虚引用的存在，完全不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它和没有引用几乎是一样的，随时都可能被垃圾回收器回收
- 它不能单独使用，也无法通过虚引用来获取被引用的对象。当试图通过虚引用的 get() 方法取得对象时，总是 `null`
- 为一个对象设置虚引用关联的唯一目的在于跟踪垃圾回收过程。比如：能在这个对象被收集器回收时收到一个系统通知
- 虚引用必须和引用队列(ReferenceQueue)一起使用。虚引用在创建时必须提供一个引用队列作为参数。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象后，将这个虚引用加入引用队列，以通知应用程序对象的回收情况。
- 由于虚引用可以跟踪对象的回收时间，因此，也可以将一些资源释放操作放置在虚引用中执行和记录

设置虚引用关联对象的唯一目的，就是在这个对象被收集器回收的时候收到一个系统通知或者后续添加进一步的处理，用来实现比finalize机制更灵活的回收操作  

```java
class Test4{
  // -Xms10M  -Xmx10m
    public static void main(String[] args) {
        Test4 test4 = new Test4();
        ReferenceQueue<Test4> objectReferenceQueue = new ReferenceQueue<>();
        PhantomReference<Test4> test4PhantomReference = new PhantomReference<>(test4, objectReferenceQueue);
        System.out.println(test4PhantomReference.get());//null
        ArrayList<byte[]> objects = new ArrayList<>();

        new Thread(()->{
            while (true){
                objects.add(new byte[1* 1024*1024]);
                try {
                    TimeUnit.MILLISECONDS.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(test4PhantomReference.get()+"//");
            }
        }).start();

    }

    @Override
    protected void finalize() throws Throwable {

        System.out.println("ffff");
    }
}
```

```
null
null//
ffff
null//
null//
Exception in thread "Thread-0" 有虚对象加入了队列
java.lang.OutOfMemoryError: Java heap space
	at com.jvm.Test4.lambda$main$0(TestLock.java:143)
	at com.jvm.Test4$$Lambda$1/1096979270.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)
```

#  七、垃圾回收器

## 分类与性能指标

### GC分类
**按线程数分（垃圾回收线程数），可以分为串行垃圾回收器和并行垃圾回收器**     
- 串行回收指的是在同一时间段内只允许有一个 CPU 用于执行垃圾回收操作，此时工作线程被暂停，直至垃圾收集工作结束。
- 并行回收可以运用多个CPU同时执行垃圾回收，因此提升了应用的吞吐量，**不过并行回收仍然与串行回收一样，采用独占式**，使用了`Stop-The-World`机制。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/线程数区分GC.png)

**按照工作模式分，可以分为并发式垃圾回收器和独占式垃圾回收器**   
- 并发式垃圾回收器与应用程序线程交替工作，以尽可能减少应用程序的停顿时间。
- 独占式垃圾回收器（Stop The World）一旦运行，就停止应用程序中的所有用户线程，直到垃圾回收过程完全结束。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/工作模式区分GC.png)

**按碎片处理方式分，可分为压缩式垃圾回收器和非压缩式垃圾回收器。**

- 压缩式垃圾回收器会在回收完成后，对存活对象进行压缩整理，消除回收后的碎片。
  - 再分配对象空间：使用指针碰撞
- 非压缩式的垃圾回收器不进行这步操作。
  - 再分配对象空间：空闲列表

**按工作的内存区间分，又可分为年轻代垃圾回收器和老年代垃圾回收器。**

### 评估 GC 的性能指标

#### 吞吐量（Throughput）
- 吞吐量就是 CPU 用于运行用户代码的时间与 CPU 总消耗时间的比值，即`吞吐量 = 运行用户代码时间 /（运行用户代码时间 + 垃圾收集时间）`
  - 如果虚拟机完成某个任务，用户代码加上垃圾收集总共耗费了100分钟，其中垃圾收集花掉1分钟，那吞吐量就是99%
- 这种情况下，应用程序能容忍较高的暂停时间，因此，高吞吐量的应用程序有更长的时间基准，快速响应 (暂停时间) 是不必考虑的
- 吞吐量优先，意味着在单位时间内，STW 的时间最短：`0.2 + 0.2 = 0.4`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/吞吐量示意图.png)

#### 暂停时间（pause time）
- `暂停时间`是指一个时间段内应用程序线程暂停，让 GC 线程执行的状态。
  - GC 期间 100 毫秒的暂停时间意味着在这 100 毫秒期间内没有应用程序线程是活动的。
- 暂停时间优先，意味着尽可能让**单次 STW 的时间最短**：`0.1 + 0.1 + 0.1 + 0.1 + 0.1 = 0.5`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/暂停时间示意图.png)

#### 吞吐量 vs 暂停时间

- 如果选择以吞吐量优先，那么必然需要**降低内存回收的执行频率**，但是这样会导致 GC 需要更长的暂停时间来执行内存回收。
- 如果选择以低延迟优先为原则，那么为了降低每次执行内存回收时的暂停时间，也只能**频繁地执行内存回收，但这又引起了年轻代内存的缩减和导致程序吞吐量的下降**

?> **在最大吞吐量优先的情况下，降低停顿时间(基于内存足够大的条件下)**


##  7 款经典的垃圾收集器
- 串行回收器：`Serial`、`Serial Old`
- 并行回收器：`ParNew`、`Parallel Scavenge`、`Parallel Old`
- 并发回收器：`CMS`、`G1`

### 收集器与垃圾分代之间的关系

- 新生代收集器：`Serial`、`ParNew`、`Parallel Scavenge`；
- 老年代收集器：`Serial Old`、`Parallel Old`、`CMS`；
- 整堆收集器：`G1`；

### 垃圾收集器的组合关系
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/垃圾收集器的组合关系.png ':size=60%')

### 垃圾收集器的演进
- 1999 年随 JDK 1.3.1 一起来的是串行方式的 `Serial GC` ，它是第一款 GC。**ParNew 垃圾收集器是 Serial 收集器的多线程版本**。
- 2002 年 2 月 26 日，`Parallel GC` 和 `Concurrent Mark Sweep GC` 跟随 JDK 1.4.2 一起发布。
- `Parallel GC` 在 JDK 6 之后成为 **HotSpot 默认 GC**。
- 2012 年，在 JDK 1.7u4 版本中，`G1` 可用。
- 2017 年，JDK 9 中 `G1` 变成默认的垃圾收集器，以替代 `CMS`。
- 2018 年 3 月，JDK 10 中 `G1` 垃圾回收器的并行完整垃圾回收，实现并行性来改善最坏情况下的延迟。
- 2018 年 9 月，JDK 11 发布。引入 `Epsilon` 垃圾回收器，又被称为 "No-Op（无操作）"回收器。同时，引入 ZGC：可伸缩的低延迟垃圾回收器（`Experimental`）
- 2019 年 3 月，JDK 12 发布。增强`G1`，自动返回未用堆内存给操作系统。同时，引入 `Shenandoah GC`：低停顿时间的 GC（`Experimental`）。
- 2019 年 9 月，JDK 13 发布。增强 `ZGC`，自动返回未用堆内存给操作系统。
- 2020 年 3 月，JDK 14 发布。删除 `CMS` 垃圾回收器。扩展 `ZGC` 在 MacOS 和 Windows 上的应用。

### 查看默认垃圾收集器  
- `XX:+PrintCommandLineFlags`：查看命令行相关参数（包含使用的垃圾收集器）
- 使用命令行指令：`jinfo -flag` 相关垃圾回收器参数、进程 ID


### Serial收集器-串行回收
- Serial收集器是最基础、历史最悠久的收集器，曾经（在JDK 1.3.1之前）是HotSpot虚拟机新生代收集器的唯一选择
- Serial 收集器作为 HotSpot 中`Client 模式`下的默认新生代垃圾收集器
- Serial 收集器采用复制算法、串行回收和"Stop-The-World"机制的方式执行内存回收
- 除了年轻代之外，Serial 收集器还提供用于执行老年代垃圾收集的 `Serial Old` 收集器。`Serial Old` 收集器同样也采用了串行回收和"Stop The World"机制，只不过内存回收算法使用的是`标记-压缩算法`
- Serial Old 是运行在 `Client 模式`下默认的老年代的垃圾回收器
- Serial Old 在 `Server 模式`下主要有两个用途：
  - 与新生代的 Parallel Scavenge 配合使用
  - 作为老年代 CMS 收集器的后备垃圾收集方案

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/serial收集器运行示意图.png)

> 这个收集器是一个单线程的收集器，但它的“单线程”的意义并不仅仅说明它只会使用**一个 CPU 或一条收集线程**去完成垃圾收集工作，更重要的是在它进行垃圾收集时，必须暂停其他所有的工作线程，直到它收集结束`STW`

> 对于**限定单个 CPU 的环境**来说，Serial收集器由于没有线程交互的开销，专心做垃圾收集自然可以获得最高的单线程收集效率

- 在 HotSpot 虚拟机中，使用 `-XX:+UseSerialGC` 参数可以指定年轻代和老年代都使用串行收集器。
  - 等价于新生代用 Serial GC，且老年代用 Serial Old GC

?> 基于目前的硬件多核环境，已经不需要串行垃圾收集器了


### ParNew 回收器-并行回收
- 如果说 Serial GC 是年轻代中的单线程垃圾收集器，那么 `ParNew` 收集器则是`Serial`收集器的**多线程版本**。
  - `Par` 是 `Parallel` 的缩写，**New：只能处理的是新生代**
- `ParNew`收集器除了采用并行回收的方式执行内存回收外，两款垃圾收集器之间几乎没有任何区别。ParNew收集器在年轻代中同样也是采用`复制算法`、`Stop-The-World`机制。
- `ParNew` 是很多 JVM 运行在 Server 模式下新生代的默认垃圾收集器。
- 对于新生代，回收次数频繁，使用并行方式高效。
- 对于老年代，回收次数少，使用串行方式节省资源,CPU 并行需要切换线程，串行可以省去切换线程的资源
- 由于 ParNew 收集器是基于并行回收，那么是否可以断定 ParNew 收集器的回收效率在任何场景下都会比 Serial 收集器更高效？
  - ParNew 收集器运行在**多CPU的环境**下，由于可以充分利用多 CPU、多核心等物理硬件资源优势，可以更快速地完成垃圾收集，提升程序的吞吐量。
  - 但是在`单CPU`的环境下，**ParNew 收集器不比 Serial 收集器更高效**。虽然 Serial 收集器是基于串行回收，但是由于 CPU 不需要频繁得做任务切换，因此可以有效避免多线程交互过程中产生的一些额外开销。
- 除 `Serial Old GC` 外，目前只有 `ParNew GC` 能与 `CMS 收集器`配合工作（JDK 8 中 Serial Old GC 移除对 ParNew GC 的支持，JDK 9 版本中已经明确提示 `UserParNewGC was deprecated`，将在后续版本中被移除，JDK 14 中移除 CMS GC）。
- 在程序中，开发人员可以通过选项`-XX:+UseParNewGC`手动指定使用 ParNew 收集器执行内存回收任务。它**表示年轻代使用并行收集器，不影响老年代**。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/parNew收集器示意图.png)

### Parallel 回收器-吞吐量优先
- HotSpot 的年轻代中除了拥有 ParNew 收集器是基于并行回收的以外，`Parallel Scavenge` 收集器同样也采用了`复制算法`、并行回收和"Stop The World"机制。
- 和 `ParNew` 收集器不同，Parallel Scavenge 收集器的目标则是达到一个可控制的吞吐量（Throughput），它也被称为`吞吐量优先的垃圾收集器`。
- 自适应调节策略也是 `Parallel Scavenge` 与 `ParNew` 一个重要区别。
- 高吞吐量则可以高效率地利用 CPU 时间，尽快完成程序的运算任务，主要适合在后台运算而不需要太多交互的任务。因此，常见在服务器环境中使用。例如，那些执行批量处理、订单处理、工资支付、科学计算的应用程序。
- `Parallel` 收集器在 JDK 1.6 时提供了用于执行老年代垃圾收集的 Parallel Old 收集器，用来代替老年代的 `Serial Old` 收集器。
- `Parallel Old` 收集器采用了`标记-压缩算法`，但同样也是基于并行回收和"Stop-The-World"机制。  
- 在程序吞吐量优先的应用场景中，`Parallel` 收集器和 `Parallel Old` 收集器的组合，在 Server 模式下的内存回收性能很不错。


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/Parallel收集器.png)

?> 在 Java 8 中，默认是此垃圾收集器

**参数配置**    
- `-XX:+UseParallelGC` 手动指定年轻代使用 Parallel 并行收集器执行内存回收任务。
- `-XX:+UseParalleloldGC` 手动指定老年代都是使用并行回收收集器。
  - 分别适用于新生代和老年代。默认 JDK 8 是开启的。
  - **上面两个参数，默认开启一个，另一个也会被开启。互相激活**
- `-XX:ParallelGcrhreads` 设置年轻代并行收集器的线程数。一般地，最好与 CPU 数量相等，以避免过多的线程数影响垃圾收集性能。
  - 在默认情况下，当 CPU 数量小于 等于8 个，ParallelGCThreads 的值等于 CPU 数量。
  - 当 CPU 数量大于 8 个，ParallelGCThreads 的值等于 3 + [5 * CPU_Count] / 8] 10核约为6
- `-XX:MaxGCPauseMillis` 设置垃圾收集器最大停顿时间（即 STW 的时间）。单位是毫秒。
  - 为了尽可能地把停顿时间控制在 MaxGCPauseMills 以内，收集器在工作时会调整 Java 堆大小或者其他一些参数。
  - 对于用户来讲，停顿时间越短体验越好。但是在服务器端，我们注重高并发，整体的吞吐量。所以服务器端适合 Parallel，进行控制。
  - 该参数使用需谨慎
- `-XX:GCTimeRatio` 垃圾收集时间占总时间的比例（= 1 /（N+1））。用于衡量吞吐量的大小。
  - 取值范围（0，100）。默认值 99，也就是垃圾回收时间不超过 1%。
  - 与前一个 `-XX:MaxGCPauseMillis` 参数有一定矛盾性。暂停时间越长，Radio 参数就容易超过设定的比例。
- `-XX:+UseAdaptiveSizePolicy` 设置 `Parallel Scavenge` 收集器具有自适应调节策略。
  - 在这种模式下，年轻代的大小、Eden 和 Survivor 的比例、晋升老年代的对象年龄等参数会被自动调整，已达到在堆大小、吞吐量和停顿时间之间的平衡点。
  - 在手动调优比较困难的场合，可以直接使用这种自适应的方式，仅指定虚拟机的最大堆、目标的吞吐量（GCTimeRatio）和停顿时间（MaxGCPauseMills），让虚拟机自己完成调优工作。

### CMS 回收器- 低延迟
CMS`Concurrent Mark Sweep`收集器是一种以获取最短回收停顿时间为目标的收集器。目前很大一部分的Java应用集中在互联网网站或者基于浏览器的B/S系统的服务端上，这类应用通常都会较为关注服务的响应速度，希望系统停顿时间尽可能短，以给用户带来良好的交互体验  

CMS收集器是基于`标记-清除`算法实现的，它的运作过程相对于前面几种收集器来说要更复杂一些，整个过程分为四个步骤:    
- 初始标记`CMS initial mark`
  - 初始标记仅仅只是标记一下GC Roots能直接关联到的对象，速度很快，标记过程会STW
- 并发标记`CMS concurrent mark`
  - 并发标记阶段就是从GC Roots的直接关联对象开始遍历整个对象图的过程，这个过程耗时较长但是不需要停顿用户线程
- 重新标记`CMS remark`
  - 而重新标记阶段则是为了修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，STW
- 并发清除`CMS concurrent sweep`
  - 清理删除掉标记阶段判断的已经死亡的对象，由于不需要移动存活对象，所以这个阶段也是可以与用户线程同时并发的

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/CMS收集器.png)

> CMS 收集器的垃圾收集算法采用的是标记-清除算法，这意味着每次执行完内存回收后，由于被执行内存回收的无用对象所占用的内存空间极有可能是不连续的一些内存块，不可避免地将会产生一些内存碎片。那么 CMS 在为新对象分配内存空间时，将无法使用`指针碰撞（Bump the Pointer）`技术，而只能够选择`空闲列表（Free List）`执行内存分配


> **CMS 为什么不使用标记整理（压缩）算法？**   
> 并发清除的时候，用`Compact`整理内存的话，原来的用户线程使用的内存还怎么用呢？要保证用户线程能继续执行，前提是它的运行的资源不受影响。`Mark Compact`标记整理更适合`Stop The World`这种场景下使用

**缺陷**  
> CMS会产生内存碎片，导致并发清除后，用户线程可用的空间不足。在无法分配大对象的情况下，不得不提前触发 `Full GC`

> CMS收集器对处理器资源非常敏感,在并发阶段，它虽然不会导致用户线程停顿，但却会因为占用了一部分线程（或者说处理器的计算能力）而导致应用程序变慢，降低总吞吐量。    
> CMS默认启动的回收线程数是（处理器核心数量+3）/4，也就是说，如果处理器核心数在四个或以上，**并发回收时垃圾收集线程只占用不超过25%的处理器运算资源，并且会随着处理器核心数量的增加而下降**。但是**当处理器核心数量不足四个时，CMS对用户程序的影响就可能变得很大**

> CMS 收集器无法处理浮动垃圾。可能出现`Concurrent Mode Failure`失败而导致另一次 Full GC 的产生。在并发标记阶段由于程序的工作线程和垃圾收集线程是同时运行或者交叉运行的，**那么在并发标记阶段如果产生新的垃圾对象，CMS 将无法对这些垃圾对象进行标记**，最终会导致这些新产生的垃圾对象没有被及时回收，从而**只能在下一次执行 GC 时释放这些之前未被回收的内存空间**。   

**参数**  
- `-XX:+UseConcMarkSweepGC` 手动指定使用 CMS 收集器执行内存回收任务。
  - 开启该参数后会自动将 `-XX:+UseParNewGC` 打开。即：`ParNew（Young 区用）+ CMS（Old 区用）+ Serial Old 的组合`。
- `-XX:CMSInitiatingoccupanyFraction` 设置堆内存使用率的阈值，一旦达到该阈值，便开始进行回收。
  - JDK 5 及以前版本的默认值为 68，即当老年代的空间使用率达到 68% 时，会执行一次 CMS 回收。JDK 6 及以上版本默认值为 92%
  - 如果内存增长缓慢，则可以设置一个稍大的值，大的阀值可以有效降低 CMS 的触发频率，减少老年代回收的次数可以较为明显地改善应用程序性能。反之，如果应用程序内存使用率增长很快，则应该降低这个阈值，以避免频繁触发老年代串行收集器。因此通过该选项便可以有效降低 Full GC 的执行次数。
- `-XX:+UseCMSCompactAtFullCollection` 用于指定在执行完 Full GC 后对内存空间进行压缩整理，以此避免内存碎片的产生。不过由于内存压缩整理过程无法并发执行，所带来的问题就是停顿时间变得更长了。
- `-XX:CMSFullGCsBeforecompaction` 设置在执行多少次 Full GC 后对内存空间进行压缩整理。
- `-XX:ParallelcMSThreads` 设置 CMS 的线程数量。CMS 默认启动的线程数是`ParallelGCThreads + 3）/ 4`

?> JDK9中 CMS 被标记为 Deprecate(JEP291)   
JDK 14 中删除 CMS 垃圾回收器(JEP363)  

### 小结  
- 如果你想要最小化地使用内存和并行开销，选 Serial GC；
- 如果你想要最大化应用程序的吞吐量，选 Parallel GC；
- 如果你想要最小化 GC 的中断或停顿时间，选 CMS GC。


### G1 回收器-区域化分代式   
`Garbage First`（简称G1）收集器是垃圾收集器技术发展历史上的里程碑式的成果，它开创了**收集器面向局部收集的设计思路**和**基于Region的内存布局**形式,采用了全新的`分区算法`

详见[分区算法](/Java/JVM虚拟机?id=分区算法 ':target=_self')

G1的设计为了适应现在不断扩大的内存和不断增加的处理器数量，进一步降低暂停时间`pause time`，同时兼顾良好的**吞吐量**。官方给 G1 设定的目标是**在延迟可控的情况下获得尽可能高的吞吐量**，所以才担当起“全功能收集器”的重任与期望

**为什么名字叫 Garbage First（G1）呢？**   
- 因为 G1 是一个并行回收器，它把堆内存分割为很多不相关的区域（Region）（物理上不连续的）。使用不同的 Region 来表示 Eden、幸存者 0 区，幸存者 1 区，老年代等。
- G1 GC 有计划地避免在整个 Java 堆中进行全区域的垃圾收集。G1 跟踪各个 Region 里面的垃圾堆积的价值大小（回收所获得的空间大小以及回收所需时间的经验值），在后台维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的 Region。
- 由于这种方式的侧重点在于回收垃圾最大量的区间（Region），所以我们给 G1 一个名字：**垃圾优先（Garbage First）**


#### 特点
并行和并发  
- 并行性：G1 在回收期间，可以有多个 GC 线程同时工作，有效利用多核计算能力。此时用户线程 STW。
- 并发性：G1 拥有与应用程序交替执行的能力，部分工作可以和应用程序同时执行，因此，一般来说，不会在整个回收阶段发生完全阻塞应用程序的情况

- 从分代上看，G1 依然属于`分代型垃圾回收器`，它会区分年轻代和老年代，年轻代依然有 Eden 区和 Survivor 区。但从堆的结构上看，它不要求整个 Eden 区、年轻代或者老年代都是连续的，也不再坚持固定大小和固定数量。
- 将堆空间分为若干个`区域（Region）`，这些区域中包含了**逻辑上的年轻代和老年代**。
- 和之前的各类回收器不同，它同时兼顾年轻代和老年代。对比其他回收器，或者工作在年轻代，或者工作在老年代；
- G1 将内存划分为一个个的 Region。内存的回收是以 Region 作为基本单位的。**Region 之间是复制算法**，但整体上实际可看作是`标记-压缩（Mark-Compact）`算法，两种算法都可以避免内存碎片
- 这是 G1 相对于 CMS 的另一大优势，G1 除了追求低停顿外，**还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为 M 毫秒的时间片段内，消耗在垃圾收集上的时间不得超过 N 毫秒**。


#### 参数设置
- `-XX:+UseG1GC`：手动指定使用 G1 垃圾收集器执行内存回收任务
- `-XX:G1HeapRegionSize`：设置每个 Region 的大小。值是 2 的幂，范围是 1MB 到 32MB 之间，目标是根据最小的 Java 堆大小划分出约 2048 个区域。默认是堆内存的 1/2000。
- `-XX:MaxGCPauseMillis`：设置期望达到的最大 GC 停顿时间指标（JVM 会尽力实现，但不保证达到），默认值是 200ms。
- `-XX:+ParallelGcThread`：设置 STW 时GC线程数的值，最多设置为 8。
- `-XX:ConcGCThreads`：设置并发标记的线程数。将 n 设置为并行垃圾回收线程数（ParallelGCThreads）的 1/4 左右。
- `-XX:InitiatingHeapOccupancyPercent`：设置触发并发 GC 周期的 Java 堆占用率阈值。超过此值，就触发 GC，默认值是 45。

#### G1 收集器的常见操作步骤

G1 的设计原则就是简化 JVM 性能调优，开发人员只需要简单的三步即可完成调优：   
第一步：开启 G1 垃圾收集器    
第二步：设置堆的最大内存     
第三步：设置最大的停顿时间    

G1 中提供了三种垃圾回收模式：YoungGC、Mixed GC 和 Full GC，在不同的条件下被触发。    

#### G1 收集器的适用场景
- 面向服务端应用，针对具有大内存、多处理器的机器。（在普通大小的堆里表现并不惊喜）
- 最主要的应用是需要低 GC 延迟，并具有大堆的应用程序提供解决方案；
- 例如：在堆大小约 6GB 或更大时，可预测的暂停时间可以低于 0.5 秒；（G1 通过每次只清理一部分而不是全部的 Region 的增量式清理来保证每次 GC 停顿时间不会过长）。
- HotSpot 垃圾收集器里，除了 G1 以外，其他的垃圾收集器使用内置的 JVM 线程执行 GC 的多线程操作，而 G1 GC 可以采用应用线程承担后台运行的 GC 工作，即当 JVM 的 GC 线程处理速度慢时，系统会调用应用程序线程帮助加速垃圾回收过程

#### Region分区
- 使用 G1 收集器时，它将整个 Java 堆划分成约 2048 个大小相同的独立 Region 块，每个 Region 块大小根据堆空间的实际大小而定，整体被控制在 1MB 到 32MB 之间，且为 2 的 N 次幂，即 1MB、2MB、4MB、8MB、16MB、32MB。可以通过 `-XX:G1HeapRegionSize` 设定。**所有的 Region 大小相同，且在 JVM 生命周期内不会被改变**。
- 虽然还保留有新生代和老年代的概念，但新生代和老年代不再是物理隔离的了，**它们都是一部分 Region（不需要连续）的集合，通过 Region 的动态分配方式实现逻辑上的连续**。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/region分区.png)

- 一个 Region 有可能属于 Eden、Survivor 或者 Old/Tenured 内存区域。但是一个 Region 只可能属于一个角色。图中的 E 表示该 Region 属于 Eden 内存区域，S 表示属于 Survivor 内存区域，O 表示属于 Old 内存区域。图中空白的表示未使用的内存空间。
- G1 垃圾收集器还增加了一种新的内存区域，叫做 `Humongous` 内存区域，如图中的 H 块。**主要用于存储大对象，如果超过 1.5 个 Region，就放到 H**。

> 对于堆中的对象，默认直接会被分配到老年代，但是如果它是一个**短期存在的大对象就会对垃圾收集器造成负面影响**。为了解决这个问题，G1 划分了一个 `Humongous` 区，它用来专门存放大对象。**如果一个 H 区装不下一个大对象，那么 G1 会寻找连续的 H 区来存储。为了能找到连续的 H 区，**有时候不得不启动 Full GC。**G1 的大多数行为都把 H 区作为老年代的一部分**来看待

?> 每个 Region 内部都是通过指针碰撞来分配空间，Region 内也支持TLAB

#### G1 垃圾回收器的回收过程
p191-193

#### G1的优化建议
- 年轻代大小
  - 避免使用 -Xmn 或 -XX:NewRatio 等相关选项显式设置年轻代大小
  - 固定年轻代的大小会覆盖暂停时间目标
  - 暂停时间目标暂停时间目标不要太过严苛
- G1 GC 的吞吐量目标是 90% 的应用程序时间和 10% 的垃圾回收时间
  - 评估 G1 GC 的吞吐量时，暂停时间目标不要太严苛。目标太过严苛表示你愿意承受更多的垃圾回收开销，而这些会直接影响到吞吐量


### 总结

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/7个垃圾收集器总结.png)

### 选择垃圾回收器
- 优先调整堆的大小让 JVM 自适应完成。
- 如果内存小于 100M，使用串行收集器。
- 如果是单核、单机程序，并且没有停顿时间的要求，串行收集器。
- 如果是多 CPU、需要高吞吐量、允许停顿时间超过 1 秒，选择并行或者 JVM 自己选择。
- 如果是多 CPU、追求低停顿时间，需快速响应（比如延迟不能超过 1 秒，如互联网应用），使用并发收集器。
- 官方推荐 G1，性能高。现在互联网的项目，基本都是使用 G1。


?> 没有最好的收集器，更没有万能的收集,调优永远是针对特定场景、特定需求，不存在一劳永逸的收集器  

### 面试题
**垃圾收集的算法有哪些？**    
垃圾收集的算法主要分为以下几种：  

1. 标记-清除算法：首先标记所有活动对象，然后清除所有未标记的对象。  
2. 复制算法：将堆空间分为两个区域，每次只使用其中一个区域，当这个区域被填满后，将存活的对象复制到另一个区域中，然后清除原来的区域。  
3. 标记-整理算法：标记所有存活的对象，然后将它们移动到堆的一端，清除堆的另一端的所有对象。  
4. 分代收集算法：将堆空间划分为不同的代，每代使用不同的收集算法，年轻代通常使用复制算法，老年代使用标记-清除或标记-整理算法。  

**如何判断一个对象是否可以回收？**   
一般有以下几种判断方法：

1. 引用计数法：记录每个对象被引用的次数，当引用次数为0时，判定为不可达对象。  
2. 可达性分析法：从一组根对象出发，寻找所有可达的对象，不可达的对象将被判定为垃圾。    
回收算法中的分析：在执行垃圾收集时，需要将所有存活的对象标记出来，剩下的对象将被判定为垃圾。
虚拟机特定的判定方法：某些特定的垃圾收集算法中，需要进行特定的判定方法，例如在标记-整理算法中，需要判定对象是否在被整理时被移动过。

这些方法都有其优缺点，各种垃圾收集器的实现会选择不同的方法来判断对象是否可回收。

**垃圾收集器工作的基本流程。**   
1. 标记：首先，垃圾收集器会从根对象开始，遍历所有可达对象，并将这些对象标记为“存活”。  
2. 清除：然后，垃圾收集器会扫描整个堆，将未标记的对象视为“垃圾”并进行清除。  
3. 压缩：为了减少内存碎片的产生，一些垃圾收集器还会对堆进行压缩。即将存活的对象向一端移动，然后清理掉另一端的所有空间，使空间变成连续的。 

不同的垃圾收集器实现会有不同的具体细节和优化策略，但是大多数垃圾收集器都会遵循上述基本流程。其中，垃圾的判断通常是通过判断对象是否有引用指向它，如果没有引用指向，则被认为是垃圾。Java垃圾收集器的目标是尽可能地回收垃圾对象，同时尽量减少对程序的干扰，以保证程序的性能和稳定性。

# 八、GC日志分析

## 配置项参数
- `-verbose:gc`显示总的 GC 堆的变化
- `-XX:+PrintGC` 输出 GC 日志。
- `-XX:+PrintGCDetails` 输出 GC 的详细日志
- `-XX:+PrintGCTimestamps` 输出 GC 的时间戳（以基准时间的形式）
- `-XX:+PrintGCDatestamps` 输出 GC 的时间戳（以日期的形式，如 2013-05-04T21：53：59.234+0800）
- `-XX:+PrintHeapAtGC` 在进行 GC 的前后打印出堆的信息
- `-Xloggc:../logs/gc.log` 日志文件的输出路径,继而使用GC工具查看GC变化图


### verbose:gc

```
- verbose:gc
```      

这个只会显示总的 GC 堆的变化，如下：

```
[GC (Allocation Failure) 80832K->19298K(227840K),0.0084018 secs]
[GC (Metadata GC Threshold) 109499K->21465K(228352K),0.0184066 secs]
[Full GC (Metadata GC Threshold) 21465K->16716K(201728K),0.0619261 secs]
```

- `Allocation Failure`GC发生的原因（分配失败）
- `Metadata GC Threshold`Metaspace大小达到了GC阈值
- `80832K->19298K`堆在GC前后的大小
- `227840K`目前堆的大小
- `0.0084018 secs`GC持续时间


### PrintGCDetails

```
-verbose:gc -XX:+PrintGCDetails
``` 

```
[GC (Allocation Failure) [PSYoungGen:70640K->10116K(141312K)] 80541K->20017K(227328K),0.0172573 secs] [Times:user=0.03 sys=0.00,real=0.02 secs]
[GC (Metadata GC Threshold) [PSYoungGen:98859K->8154K(142336K)] 108760K->21261K(228352K),0.0151573 secs] [Times:user=0.00 sys=0.01,real=0.02 secs]
[Full GC (Metadata GC Threshold)[PSYoungGen:8154K->0K(142336K)]
[ParOldGen:13107K->16809K(62464K)] 21261K->16809K(204800K),[Metaspace:20599K->20599K(1067008K)],0.0639732 secs]
[Times:user=0.14 sys=0.00,real=0.06 secs]
```

- `PSYoungGen` 使用了`Parallel Scavenge`并行垃圾收集器的新生代GC前后大小变化
- `ParOldGen`使用了`Parallel Old`并行垃圾收集器的老年代GC前后大小变化
- `Metaspace`元空间GC前后变化
- `[Times:user=0.14 sys=0.00,real=0.06 secs]`
  - `user` 垃圾收起花费的所有CPU时间
  - `sys` 花费在等待系统调用或系统事件的时间
  - `real` GC从开始到结束的时间

### Timestamps

```
-verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimestamps -XX:+PrintGCDatestamps
```

GC日志中每次GC的时间都会显示  

```
2019-09-24T22:15:24.518+0800: 3.287: [GC (Allocation Failure) [PSYoungGen:136162K->5113K(136192K)] 141425K->17632K(222208K),0.0248249 secs] [Times:user=0.05 sys=0.00,real=0.03 secs]
2019-09-24T22:15:25.559+0800: 4.329: [GC (Metadata GC Threshold) [PSYoungGen:97578K->10068K(274944K)] 110096K->22658K(360960K),0.0094071 secs] [Times: user=0.00 sys=0.00,real=0.01 secs]
2019-09-24T22:15:25.569+0800: 4.338: [Full GC (Metadata GC Threshold) [PSYoungGen:10068K->0K(274944K)][ParoldGen:12590K->13564K(56320K)] 22658K->13564K(331264K),[Metaspace:20590K->20590K(1067008K)],0.0494875 secs] [Times: user=0.17 sys=0.02,real=0.05 secs]

```

### 日志文件  

```
-Xloggc:/path/to/gc.log
```

将GC信息写入日志文件   

### Minor GC日志

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/MinorGC-YoungGC日志解读.png)

### FullGC 日志
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/FullGC日志解读.png)


## 实例  

JVM参数   
```
-verbose:gc -Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails -XX:SurvivorRatio=8 -XX:+UseSerialGC
```

> `-Xms20M -Xmx20M -Xmn10M`：堆空间初始大小20M，堆空间最大大小20M，新生代大小10M     
> `-XX:SurvivorRatio=8`：新生代中Eden区占8M，两个Survivor区各1M(8:1:1)   
> `-XX:+UseSerialGC`：使用 SerialGC 垃圾回收器    

## 日志分析工具
常用的日志分析工具有：GCViewer、GCEasy、GCHisto、GCLogViewer、Hpjmeter、garbagecat 等

```
/**
 * -Xms60m -Xmx60m -XX:SurvivorRatio=8 -XX:+PrintGCDetails -Xloggc:./logs/gc.log
 */
public class GCLogTest {
    public static void main(String[] args) {
        ArrayList<byte[]> list = new ArrayList<>();

        for (int i = 0; i < 500; i++) {
            byte[] arr = new byte[1024 * 100]; //100KB
            list.add(arr);
            try {
                Thread.sleep(50);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

```

> 上面的参数配置会在项目路径的logs文件夹中生成gc.log文件，再导入日志分析工具进行分析

## 垃圾回收器的新发展

### Open JDK 12 的 Shenandoash GC

### ZGC
《深入理解 Java 虚拟机》一书中这样定义 ZGC：ZGC 收集器是一款基于 Region 内存布局的，（暂时）不设分代的，使用了读屏障、染色指针和内存多重映射等技术来实现可并发的标记-压缩算法的，以低延迟为首要目标的一款垃圾收集器。

未来将在服务端、大内存、低延迟应用的首选的垃圾收集器

JDK 14 之前，ZGC 仅 Linux 才支持   
`-XX:+UnlockExperimentalVMOptions-XX：+UseZGC`

### AliGC


***

?> P204-P265主要是字节码指令的学习 略去

***

# 九、类加载过程和类加载器

## 经典面试
蚂蚁金服：
 
描述一下JVM加载Class文件的原理机制？
 
类加载过程
 
类加载的时机
 
java类加载过程？
 
简述java类加载机制？
 
JVM中类加载机制，类加载过程？
 
JVM类加载机制
 
Java类加载过程
 
描述一下jvm加载class文件的原理机制
 
什么是类的加载？
 
哪些情况会触发类的加载？
 
讲一下JVM加载一个类的过程JVM的类加载机制是什么？

## 一 Loading（加载）阶段

### 加载完成的操作
加载，简而言之就是将Java类的字节码文件加载到机器内存中，并在内存中构建出Java类的原型——类模板对象。  

`类模板对象`，其实就是Java类在]VM内存中的一个快照，JVM将从字节码文件中解析出的常量池、类字段、类方法等信息存储到类模板中，这样JVM在运行期便能通过类模板而获取Java类中的任意信息，能够对Java类的成员变量进行遍历，也能进行Java方法的调用。

> 反射的机制即基于这一基础。如果JVM没有将Java类的声明信息存储起来，则JVM在运行期也无法反射。

加载阶段，简言之，查找并加载类的二进制数据，生成Class的实例。   
在加载类时，Java虚拟机必须完成以下3件事情：
- 通过类的全名，获取类的二进制数据流。 
- 解析类的二进制数据流为方法区内的数据结构（Java类模型） 
- 创建`java.lang.Class`类的实例，表示该类型。作为方法区这个类的各种数据的访问入口 


### 二进制流的获取方式
对于类的二进制数据流，虚拟机可以通过多种途径产生或获得。**只要所读取的字节码符合JVM规范即可**

- 虚拟机可能通过文件系统读入一个class后缀的文件（最常见）
- 读入jar、zip等归档数据包，提取类文件。
- 事先存放在数据库中的类的二进制数据
- 使用类似于HTTP之类的协议通过网络进行加载
- 在运行时生成一段class的二进制信息等
- 在获取到类的二进制信息后，Java虚拟机就会处理这些数据，并最终转为一个java.lang.Class的实例。

如果输入数据不是ClassFile的结构（格式验证失败），则会抛出`ClassFormatError`。

### 类模型与Class实例的位置
> **类模型的位置**   
> 加载的类在JVM中创建相应的类结构，类结构会存储在方法区（JDKl.8之前：永久代；J0Kl.8及之后：元空间）。

> **Class实例的位置**
> 类将.class文件加载至元空间后，会在堆中创建一个Java.lang.Class对象，用来封装类位于方法区内的数据结构，该Class对象是在加载类的过程中创建的，每个类都对应有一个Class类型的对象。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/class类实例位置参考.png)


### 数组类的加载
创建数组类的情况稍微有些特殊，因为数组类本身并不是由类加载器负责创建，而是由JVM在运行时根据需要而直接创建的，但数组的元素类型仍然需要依靠类加载器去创建。创建数组类（下述简称A）的过程：  
- 如果数组的元素类型是引用类型，那么就遵循定义的加载过程递归加载和创建数组A的元素类型；
- JVM使用指定的元素类型和数组维度来创建新的数组类。

如果数组的元素类型是`引用类型`，数组类的可访问性就由元素类型的可访问性决定。否则数组类的可访问性将被缺省定义为public


## 二 Linking（链接）阶段

### 链接阶段之Verification（验证）
当类加载到系统后，就开始链接操作，**验证是链接操作的第一步**。  
它的目的是保证加载的字节码是合法、合理并符合规范的。   
验证的步骤比较复杂，实际要验证的项目也很繁多，大体上Java虚拟机需要做以下检查，如图所示：  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/验证阶段的图示.png)

验证的内容则涵盖了类数据信息的格式验证、语义检查、字节码验证，以及符号引用验证等。

- 其中格式验证会和加载阶段一起执行。验证通过之后，类加载器才会成功将类的二进制数据信息加载到方法区中。
- 格式验证之外的验证操作将会在方法区中进行。

链接阶段的验证虽然拖慢了加载速度，但是它避免了在字节码运行时还需要进行各种检查。


具体说明：
1. `格式验证`：是否以`魔数0XCAFEBABE`开头，主版本和副版本号是否在当前Java虚拟机的支持范围内，数据中每一个项是否都拥有正确的长度等。 
2. `语义检查`：Java虚拟机会进行字节码的语义检查，但凡在语义上不符合规范的，虚拟机也不会给予验证通过。比如： 
  - 是否所有的类都有父类的存在（在Java里，除了object外，其他类都应该有父类）
  - 是否一些被定义为final的方法或者类被重写或继承了
  - 非抽象类是否实现了所有抽象方法或者接口方法
3. `字节码验证`：Java虚拟机还会进行字节码验证，字节码验证也是验证过程中最为复杂的一个过程。它试图通过对字节码流的分析，判断字节码是否可以被正确地执行。比如： 
  - 在字节码的执行过程中，是否会跳转到一条不存在的指令
  - 函数的调用是否传递了正确类型的参数
  - 变量的赋值是不是给了正确的数据类型等


栈映射帧（StackMapTable）就是在这个阶段，用于检测在特定的字节码处，其局部变量表和操作数栈是否有着正确的数据类型。但遗憾的是，100%准确地判断一段字节码是否可以被安全执行是无法实现的，因此，该过程只是尽可能地检查出可以预知的明显的问题。如果在这个阶段无法通过检查，虚拟机也不会正确装载这个类。但是，如果通过了这个阶段的检查，也不能说明这个类是完全没有问题的。

在前面3次检查中，已经排除了文件格式错误、语义错误以及字节码的不正确性。但是依然不能确保类是没有问题的。   

4. `符号引用的验证`：校验器还将进符号引用的验证。Class文件在其常量池会通过字符串记录自己将要使用的其他类或者方法。因此，在验证阶段，虚拟机就会检查这些类或者方法确实是存在的，并且当前类有权限访问这些数据，如果一个需要使用类无法在系统中找到，则会抛出NoClassDefFoundError，如果一个方法无法被找到，则会抛出NoSuchMethodError。此阶段在解析环节才会执行。 

### 链接阶段之Preparation（准备）

准备阶段（Preparation），简言之，为类的静态变分配内存，并将其初始化为默认值。

当一个类验证通过时，虚拟机就会进入准备阶段。在这个阶段，虚拟机就会为这个类分配相应的内存空间，并设置默认初始值。Java虚拟机为各类型变量默认的初始值如表所示

类型 | 默认初始值|
-----| -----|
byte| (byte)0
short |	(short)0
int |	0
long |	0L
float |	0.0f
double |	0.0
char |	\u0000
boolean |	false
reference |	null

?> Java并不支持boolean类型，对于boolean类型，内部实现是int，由于int的默认值是0，故对应的，boolean的默认值就是false。

> 这里不包含基本数据类型的字段用staticfinal修饰的情况，因为final在编译的时候就会分配了，准备阶段会显式赋值

```java
// 一般情况：static final修饰的基本数据类型、字符串类型字面量会在准备阶段赋值
private static final String str = "Hello world";
// 特殊情况：static final修饰的引用类型不会在准备阶段赋值，而是在初始化阶段赋值 在clinit中进行赋值
private static final String str = new String("Hello world");
```

- 注意这里不会为实例变量分配初始化，类变量会分配在方法区中，而实例变量是会随着对象一起分配到Java堆中。 
- **在这个阶段并不会像初始化阶段中那样会有初始化或者代码被执行**

### 链接阶段之Resolution（解析）
在准备阶段完成后，就进入了解析阶段。解析阶段（Resolution），简言之，**将类、接口、字段和方法的符号引用转为直接引用**(做菜教程和实际操作)。

具体描述：   
符号引用就是一些字面量的引用，和虚拟机的内部数据结构和和内存布局无关。比较容易理解的就是在Class类文件中，通过常量池进行了大量的符号引用。但是在程序实际运行时，只有符号引用是不够的，比如当如下`println()`方法被调用时，系统需要明确知道该方法的位置  

以方法为例，Java虚拟机为每个类都准备了一张方法表，将其所有的方法都列在表中，当需要调用一个类的方法的时候，只要知道这个方法在方法表中的偏移量就可以直接调用该方法。通过解析操作，符号引用就可以转变为目标方法在类中方法表中的位置，从而使得方法被成功调用。


## 三 Initialization（初始化）阶段
类的初始化是类装载的最后一个阶段，如果前面的步骤都没有问题，那么表示类可以顺利装载到系统中，此时，类才会开始执行Java字节码

**到了初始化阶段，才真正开始执行类中定义的Java程序代码**


### static与final的搭配问题

使用static+ final修饰的字段的显式赋值的操作，到底是在哪个阶段进行的赋值？  
- 情况1：在链接阶段的准备环节赋值 
- 情况2：在初始化阶段`<clinit>()`中赋值 

在链接阶段的准备环节赋值的情况： 
- 对于`基本数据类型`的字段来说，如果使用`static final`修饰，则显式赋值(直接赋值常量，而非调用方法通常是在链接阶段的准备环节进行) 
- 对于`String`来说，如果使用`字面量的方式赋值`，使用`static final`修饰的话，则显式赋值通常是在链接阶段的准备环节进行 
- 在初始化阶段`<clinit>()`中赋值的情况： 排除上述的在准备环节赋值的情况之外的情况。 

?> **最终结论**：使用`static+final`修饰，且显式赋值中不涉及到方法或构造器调用的基本数据类或String类型的显式赋值，是在链接阶段的准备环节进行。

```java
public static final int INT_CONSTANT = 10;                                // 在链接阶段的准备环节赋值
public static final int NUM1 = new Random().nextInt(10);                  // 在初始化阶段clinit>()中赋值
public static int a = 1;                                                  // 在初始化阶段<clinit>()中赋值

public static final Integer INTEGER_CONSTANT1 = Integer.valueOf(100);     // 在初始化阶段<clinit>()中赋值
public static Integer INTEGER_CONSTANT2 = Integer.valueOf(100);           // 在初始化阶段<clinit>()中概值

public static final String s0 = "helloworld0";                            // 在链接阶段的准备环节赋值
public static final String s1 = new String("helloworld1");                // 在初始化阶段<clinit>()中赋值
public static String s2 = "hellowrold2";                                  // 在初始化阶段<clinit>()中赋值
```


### clinit 的线程安全性
对于`<clinit>()`方法的调用，也就是类的初始化，虚拟机会在内部确保其多线程环境中的安全性。

虚拟机会保证一个类的()方法在多线程环境中被正确地加锁、同步，如果多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的`<clinit>()`方法，其他线程都需要阻塞等待，直到活动线程执行`<clinit>()`方法完毕。

正是因为**函数`<clinit>()`带锁线程安全**的，因此，如果在一个类的`<clinit>()`方法中有耗时很长的操作，就可能造成多个线程阻塞，引发死锁。并且这种死锁是很难发现的，因为看起来它们并没有可用的锁信息。

如果之前的线程成功加载了类，则等在队列中的线程就没有机会再执行`<clinit>()`方法了。那么，当需要使用这个类时，虚拟机会直接返回给它已经准备好的信息。

### 类的初始化情况：主动使用vs被动使用

Java程序对类的使用分为两种：主动使用和被动使用 

**主动使用**
主动使用☞调用`<clinit>`
Class只有在必须要首次使用的时候才会被装载，Java虚拟机不会无条件地装载Class类型。Java虚拟机规定，一个类或接口在初次使用前，必须要进行初始化。这里指的“使用”，是指主动使用，主动使用只有下列几种情况：（即：如果出现如下的情况，则**会对类进行初始化操作。而初始化操作之前的加载、验证、准备已经完成**。

- `实例化`：当创建一个类的实例时，比如使用new关键字，或者通过反射、克隆、反序列化（序列化）。 
- `静态方法`：当**调用**类的静态方法时，即当使用了字节码`invokestatic`指令。 
- `静态字段`：当使用类、接口的静态字段时（final修饰特殊考虑），比如，使用getstatic或者putstatic指令。（对应访问变量、赋值变量操作） 
- `反射`：当使用`java.lang.reflect`包中的方法反射类的方法时。比如：`Class.forName("com.atguigu.java.Test")` 
- `继承`：当初始化子类时，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化

> 在初始化一个类时，并不会先初始化它所实现的接口   
> 在初始化一个接口时，并不会先初始化它的父接口   
> 因此，**一个父接口并不会因为它的子接口或者实现类的初始化而初始化。只有当程序首次使用特定接口的静态字段时，才会导致该接口的初始化**。

- `default方法`：如果一个接口定义了default方法，那么直接实现或者间接实现该接口的类的初始化，该接口要在其之前被初始化。
- `main方法`：当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类。 

> VM启动的时候通过引导类加载器加载一个初始类。这个类在调用`public static void main(String[])`方法之前被链接和初始化。这个方法的执行将依次导致所需的类的加载，链接和初始化。

- `MethodHandle`：当初次调用MethodHandle实例时(`java.lang.invoke`包下)，初始化该MethodHandle指向的方法所在的类。（涉及解析REF getStatic、REF_putStatic、REF invokeStatic方法句柄对应(指向)的类） 

**被动使用**
除了以上的情况属于主动使用，其他的情况均属于`被动使用`。**被动使用不会引起类的初始化**。    
也就是说：并不是在代码中出现的类，就一定会被加载或者初始化。**如果不符合主动使用的条件，类就不会初始化**

- `静态字段`：当通过子类引用父类的静态变量，不会导致子类初始化，只有真正声明这个字段的类才会被初始化。 
- `数组定义`：通过数组定义类引用，不会触发此类的初始化 
- `引用常量`：引用常量不会触发**类或接口的初始化**。因为常量在链接阶段就已经被显式赋值了。 
- `LoadClass方法`：**调用ClassLoader类的loadClass()方法加载一个类，并不是对类的主动使用**，不会导致类的初始化。 
  - `Class clazz = ClassLoader.getSystemClassLoader().loadClass("com.test.java.Person");`


`-XX:+TraceClassLoading`追踪打印类的加载信息

## 类的Using（使用）
开发人员可以在程序中访问和调用它的静态类成员信息（比如：静态字段、静态方法），或者使用new关键字为其创建对象实例。


## 类的Unloading（卸载）
在类加载器的内部实现中，用一个Java集合来存放所加载类的引用。另一方面，一个Class对象总是会引用它的类加载器，调用Class对象的getClassLoader()方法，就能获得它的类加载器。由此可见，代表某个类的Class实例与其类的加载器之间为双向关联关系。

一个类的实例总是引用代表这个类的Class对象。在Object类中定义了getClass()方法，这个方法返回代表对象所属类的Class对象的引用。此外，所有的java类都有一个静态属性class，它引用代表这个类的Class对象

### 类的生命周期

当Sample类被加载、链接和初始化后，它的生命周期就开始了。当代表Sample类的Class对象不再被引用，即不可触及时，Class对象就会结束生命周期，Sample类在方法区内的数据也会被卸载，从而结束Sample类的生命周期。

**一个类何时结束生命周期，取决于代表它的Class对象何时结束生命周期。**

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/类的生命周期Sample实例.png)

loader1变量和obj变量间接应用代表Sample类的Class对象，而objClass变量则直接引用它。

如果程序运行过程中，将上图左侧三个引用变量都置为null，此时Sample对象结束生命周期，MyClassLoader对象结束生命周期，代表Sample类的Class对象也结束生命周期，Sample类在方法区内的二进制数据被卸载。

当再次有需要时，会检查Sample类的Class对象是否存在，如果存在会直接使用，不再重新加载；如果不存在Sample类会被重新加载，在Java虚拟机的堆区会生成一个新的代表Sample类的Class实例（可以通过哈希码查看是否是同一个实例）

### 类的卸载
- 启动类加载器加载的类型在整个运行期间是不可能被卸载的（jvm和jls规范）

- 被系统类加载器和扩展类加载器加载的类型在运行期间不太可能被卸载，因为系统类加载器实例或者扩展类的实例基本上在整个运行期间总能直接或者间接的访问的到，其达到unreachable的可能性极小。

- 被开发者自定义的类加载器实例加载的类型只有在很简单的上下文环境中才能被卸载，而且一般还要借助于强制调用虚拟机的垃圾收集功能才可以做到。可以预想，稍微复杂点的应用场景中（比如：很多时候用户在开发自定义类加载器实例的时候采用缓存的策略以提高系统性能），被加载的类型在运行期间也是几乎不太可能被卸载的（至少卸载的时间是不确定的）。

综合以上三点，**一个已经加载的类型被卸载的几率很小至少被卸载的时间是不确定的**。同时我们可以看的出来，开发者在开发代码时候，不应该对虚拟机的类型卸载做任何假设的前提下，来实现系统中的特定功能。


## 类加载器

类加载器是JVM执行类加载机制的前提。

**ClassLoader的作用：**    
ClassLoader是Java的核心组件，**所有的Class都是由ClassLoader进行加载的**，ClassLoader负责通过各种方式将Class信息的二进制数据流读入JVM内部，转换为一个与目标类对应的`java.lang.Class`对象实例。然后交给Java虚拟机进行链接、初始化等操作。因此，**ClassLoader在整个装载阶段，只能影响到类的加载，而无法通过ClassLoader去改变类的链接和初始化行为**。至于它是否可以运行，则由`Execution Engine`决定。


### 类的加载分类
**显式加载、隐式加载**   
- `显式加载`指的是在代码中通过调用ClassLoader加载class对象，如直接使用Class.forName(name)或this.getClass().getClassLoader().loadClass()加载class对象。
- `隐式加载`则是不直接在代码中调用ClassLoader的方法加载class对象，而是通过虚拟机自动加载到内存中，如在加载某个类的class文件时，该类的class文件中引用了另外一个类的对象，此时额外引用的类将通过JVM自动加载到内存中。

```java
//隐式加载
User user=new User();
//显式加载，并初始化
Class clazz=Class.forName("com.test.java.User");
//显式加载，但不初始化
ClassLoader.getSystemClassLoader().loadClass("com.test.java.Parent");
```


### 类加载器的必要性

一般情况下，Java开发人员并不需要在程序中显式地使用类加载器，但是了解类加载器的加载机制却显得至关重要。从以下几个方面说：

- 避免在开发中遇到`java.lang.ClassNotFoundException`异常或`java.lang.NoClassDefFoundError`异常时，手足无措。只有了解类加载器的 加载机制才能够在出现异常的时候快速地根据错误异常日志定位问题和解决问题
- 需要支持类的动态加载或需要对编译后的字节码文件进行加解密操作时，就需要与类加载器打交道了。
- 开发人员可以在程序中编写自定义类加载器来重新定义类的加载规则，以便实现一些自定义的处理逻辑。


### 类的唯一性
对于任意一个类，都需要由**加载它的类加载器**和这个**类本身**一同确认其在Java虚拟机中的唯一性。   

每一个类加载器，都拥有一个独立的类名称空间：**比较两个类是否相等，只有在这两个类是由同一个类加载器加载的前提下才有意义**。   
否则，**即使这两个类源自同一个Class文件，被同一个虚拟机加载，只要加载他们的类加载器不同，那这两个类就必定不相等**

### 命名空间

- 每个类加载器都有自己的命名空间，命名空间由该加载器及所有的父加载器所加载的类组成 
- 在同一命名空间中，不会出现类的完整名字（包括类的包名）相同的两个类 
- 在不同的命名空间中，有可能会出现类的完整名字（包括类的包名）相同的两个类 

在大型应用中，我们往往借助这一特性，来运行同一个类的不同版本。


```java
//命名空间示例  
public static void main(String[] args){
  String rootDir = "D:\\code\\src";
  try{
    //创建自定义的类的加载器1
    UserClassLoader loader1 = new UserClassLoader(rootDir);
    Class clazz1 = loader1.findClass("com.ric.java.User");

    //创建自定义的类的加载器2
    UserClassLoader loader2 = new UserClassLoader(rootDir);
    Class clazz2 = loader2.findClass("com.ric.java.User");

    System.out.println(clazz1 == clazz2); //false

  }
}
```

### 类加载机制的基本特征

双亲委派模型。但不是所有类加载都遵守这个模型，有的时候，启动类加载器所加载的类型，是可能要加载用户代码的，比如JDK内部的ServiceProvider/ServiceLoader机制，用户可以在标准API框架上，提供自己的实现，JDK也需要提供些默认的参考实现。例如，Java中JNDI、JDBC、文件系统、Cipher等很多方面，都是利用的这种机制，这种情况就不会用双亲委派模型去加载，而是利用所谓的上下文加载器。

- 可见性，子类加载器可以访问父加载器加载的类型，但是反过来是不允许的。不然，因为缺少必要的隔离，我们就没有办法利用类加载器去实现容器的逻辑。
- 单一性，由于父加载器的类型对于子加载器是可见的，所以父加载器中加载过的类型，就不会在子加载器中重复加载。但是注意，类加载器“邻居”间，同一类型仍然可以被加载多次，因为互相并不可见。

### 类的加载器分类  

#### 引导类加载器
**启动类加载器（引导类加载器，Bootstrap ClassLoader）**

- 这个类加载使用C/C++语言实现的，嵌套在JVM内部。 
- 它用来加载Java的核心库（JAVAHOME/jre/lib/rt.jar或sun.boot.class.path路径下的内容）。用于提供JVM自身需要的类。 
- 并不继承自`java.lang.ClassLoader`，没有父加载器。 
- 出于安全考虑，Bootstrap启动类加载器只加载包名为java、javax、sun等开头的类 
- 加载扩展类和应用程序类加载器，并指定为他们的父类加载器

#### 扩展类加载器
扩展类加载器（Extension ClassLoader）

- Java语言编写，由`sun.misc.Launcher$ExtClassLoader`实现。 
- 继承于`ClassLoader`类 
- 父类加载器为启动类加载器 
- 从java.ext.dirs系统属性所指定的目录中加载类库，或从JDK的安装目录的jre/lib/ext子目录下加载类库。如果用户创建的JAR放在此目录下，也会自动由扩展类加载器加载。


#### 系统类加载器
应用程序类加载器（系统类加载器，AppClassLoader）

- java语言编写，由sun.misc.Launcher$AppClassLoader实现
- 继承于ClassLoader类
- 父类加载器为扩展类加载器
- 它负责加载环境变量classpath或系统属性java.class.path 指定路径下的类库
- **应用程序中的类加载器默认是系统类加载器**。
- 它是用户自定义类加载器的默认父加载器
- 通过ClassLoader的getSystemClassLoader()方法可以获取到该类加载器 

?> 一般的我们自定义类加载器是继承于`ClassLoader类`，但是自定义类加载器的父类加载器仍然是AppClassLoader    
**类加载器的父子关系并不依据类的继承关系**

#### 用户自定义类加载器

用户自定义类加载器

- 在Java的日常应用程序开发中，类的加载几乎是由上述3种类加载器相互配合执行的。在必要时，我们还可以自定义类加载器，来定制类的加载方式。
- 体现Java语言强大生命力和巨大魅力的关键因素之一便是，Java开发者可以自定义类加载器来实现类库的动态加载，加载源可以是本地的JAR包，也可以是网络上的远程资源。
- **通过类加载器可以实现非常绝妙的插件机制**，这方面的实际应用案例举不胜举。例如，著名的OSGI组件框架，再如Eclipse的插件机制。类加载器为应用程序提供了一种动态增加新功能的机制，这种机制无须重新打包发布应用程序就能实现。
- 同时，**自定义加载器能够实现应用隔离**，例如Tomcat，Spring等中间件和组件框架都在内部实现了自定义的加载器，并通过自定义加载器隔离不同的组件模块。这种机制比C/C程序要好太多，想不修改C/C程序就能为其新增功能，几乎是不可能的，仅仅一个兼容性便能阻挡住所有美好的设想。
- 自定义类加载器**通常需要继承于ClassLoader**。

### 测试不同的类的加载器
每个Class对象都会包含一个定义它的ClassLoader的一个引用。

获取ClassLoader的途径:

```java
// 获得当前类的ClassLoader
clazz.getClassLoader()
// 获得当前线程上下文的ClassLoader
Thread.currentThread().getContextClassLoader()
// 获得系统的ClassLoader
ClassLoader.getSystemClassLoader()
```

```java
public class ClassLoaderTest1{
    public static void main(String[] args) {
        //获取系统该类加载器
        ClassLoader systemClassLoader=ClassLoader.getSystemCLassLoader();
        System.out.print1n(systemClassLoader);//sun.misc.Launcher$AppCLassLoader@18b4aac2
        //获取扩展类加载器
        ClassLoader extClassLoader =systemClassLoader.getParent();
        System.out.println(extClassLoader);//sun.misc. Launcher$ExtCLassLoader@1540e19d
        //试图获取引导类加载器：失败
        ClassLoader bootstrapClassLoader =extClassLoader.getParent();
        System.out.print1n(bootstrapClassLoader);//null

        //##################################
        try{
            ClassLoader classLoader =Class.forName("java.lang.String").getClassLoader();
            System.out.println(classLoader);
            //自定义的类默认使用系统类加载器
            ClassLoader classLoader1=Class.forName("com.atguigu.java.ClassLoaderTest1").getClassLoader();
            System.out.println(classLoader1);
            
            //关于数组类型的加载：使用的类的加载器与数组元素的类的加载器相同
            String[] arrstr = new String[10];
            System.out.println(arrstr.getClass().getClassLoader());//null：表示使用的是引导类加载器
                
            ClassLoaderTest1[] arr1 =new ClassLoaderTest1[10];
            System.out.println(arr1.getClass().getClassLoader());//sun.misc. Launcher$AppcLassLoader@18b4aac2
            
            int[] arr2 = new int[10];
            System.out.println(arr2.getClass().getClassLoader());//null:基本数据类型不需要加载器
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

- 站在程序的角度看，**引导类加载器与另外两种类加载器（系统类加载器和扩展类加载器）并不是同一个层次意义上的加载器**，引导类加载器是使用C++语言编写而成的，而另外两种类加载器则是使用Java语言编写而成的。由于引导类加载器压根儿就不是一个Java类，因此在Java程序中只能打印出空值。
- 数组类的Class对象，不是由类加载器去创建的，而是在Java运行期JVM根据需要自动创建的。对于数组类的类加载器来说，是通过Class.getClassLoader()返回的，与数组当中元素类型的类加载器是一样的；**如果数组当中的元素类型是基本数据类型，数组类是没有类加载器的**。

### ClassLoader源码解析

**ClassLoader与现有类的关系：**
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/classloader与现有类加载器的关系.png)

除了以上虚拟机自带的加载器外，用户还可以定制自己的类加载器。Java提供了抽象类`java.lang.ClassLoader`，所有用户自定义的类加载器都应该继承ClassLoader类。


### ClassLoader的主要方法

```java
//抽象类ClassLoader的主要方法：（内部没有抽象方法
public final ClassLoader getParent();

//返回该类加载器的超类加载器
public Class<?> loadClass(String name) throws ClassNotFoundException
```

**loadClass解读**

```java
//测试代码
ClassLoader.getSytstemClassLoader().loadClass("com.ric.java.User");

//name 全类名  resolve 是否在加载class的同时进行解析操作
protected Class<?> loadClass(String name, boolean resolve)throws ClassNotFoundException{
        synchronized (getClassLoadingLock(name)) {// 同步操作，保证只加载一次
            // 首先，检查请求的类是否已经被加载过了,缓存中查找
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                  //获取当前类加载器的父类加载器
                    if (parent != null) {
                        //若父类存在就委托给父类进行加载
                        c = parent.loadClass(name, false);
                    } else {
                        //父类加载器是null 父类加载器是引导类加载器
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // 如果父类加载器抛出ClassNotFoundException
                    // 说明父类加载器无法完成加载请求
                }

                if (c == null) {
                    // 在父类加载器无法加载时
                    // 再调用本身的findClass方法来进行类加载
                    long t1 = System.nanoTime();
                    c = findClass(name);

                }
            }
            if (resolve) {
              //是否进行解析
                resolveClass(c);
            }
            return c;
        }
    }

```

加载名称为name的类，返回结果为java.lang.Class类的实例。如果找不到类，则返回 ClassNotFoundException异常。该方法中的逻辑就是双亲委派模式的实现


**findClass 解读**

```java
protected Class<?> findClass(String name) throws ClassNotFoundException {
        throw new ClassNotFoundException(name);
    }
```

作用：查找二进制名称为name的类，返回结果为java.lang.Class类的实例。这是一个受保护的方法，JVM鼓励我们重写此方法，需要自定义加载器遵循双亲委托机制，该方法会在检查完父类加载器之后被loadClass()方法调用。

- 在JDK1.2之前，在自定义类加载时，总会去继承ClassLoader类并重写loadClass方法，从而实现自定义的类加载类。**但是在JDK1.2之后已不再建议用户去覆盖loadClass()方法**，而是建议把自定义的类加载逻辑写在findClass()方法中，从前面的分析可知，findClass()方法是在loadClass()方法中被调用的，**当loadClass()方法中父加载器加载失败后，则会调用自己的findClass()方法来完成类加载，这样就可以保证自定义的类加载器也符合双亲委托模式**。 
- 需要注意的是ClassLoader类中并没有实现findClass()方法的具体代码逻辑，取而代之的是抛出ClassNotFoundException异常，同时应该知道的是findClass方法通常是和defineClass方法一起使用的。

**defineClass 解读**

```java
 private Class<?> defineClass(String name, Resource res) throws IOException {}
```

根据给定的字节数组b转换为Class的实例，off和len参数表示实际Class信息在byte数组中的位置和长度，其中byte数组b是ClassLoader从外部获取的。这是受保护的方法，只有在自定义ClassLoader子类中可以使用。

> defineClass()方法是用来将byte字节流解析成JVM能够识别的Class对象（ClassLoader中已实现该方法逻辑），通过这个方法不仅能够通过class文件实例化class对象，也可以通过其他方式实例化class对象，如通过网络接收一个类的字节码，然后转换为byte字节流创建对应的Class对象。 

?> defineClass()方法通常与findClass()方法一起使用,一般情况下，在自定义类加载器时，会直接覆盖ClassLoader的findClass()方法并编写加载规则，取得要加载类的字节码后转换成流，然后调用defineClass()方法生成类的Class对象


### SecureClassLoader 与 URLClassLoader

`SecureClassLoader`扩展了`ClassLoader`，新增了几个与使用相关的代码源（对代码源的位置及其证书的验证）和权限定义类验证（主要指对class源码的访问权限）的方法，一般我们不会直接跟这个类打交道，更多是与它的子类URLClassLoader有所关联。

`ClassLoader`是一个抽象类，很多方法是空的没有实现，比如`findClass()`、`findResource()`等。而`URLClassLoader`这个实现类为这些方法提供了具体的实现。并新增了`URLClassPath`类协助取得Class字节码流等功能。在编写自定义类加载器时，如果没有太过于复杂的需求，可以直接继承`URLClassLoader`类，这样就可以避免自己去编写`findClass()`方法及其获取字节码流的方式，使自定义类加载器编写更加简洁。  

### ExtClassLoader与AppClassLoade
拓展类加载器`ExtClassLoader`和系统类加载器`AppClassLoader`，这两个类都继承自`URLClassLoader`，是`sun.misc.Launcher`的静态内部类

`ExtClassLoader`并没有重写`loadClass()`方法，这足矣说明其遵循双亲委派模式，而`AppClassLoader`重载了`loadClass()`方法，但最终调用的还是父类`loadClass()`方法，因此依然遵守双亲委派模式。


### Class.forName() 与 ClassLoader.loadClass()

- `Class.forName()`：是一个静态方法，最常用的是`Class.forName(String className)`; 
-  根据传入的类的全限定名返回一个Class对象。该方法在将Class文件加载到内存的同时，**会执行类的初始化**。 

- `ClassLoader.loadClass()`：这是一个实例方法，需要一个`ClassLoader`对象来调用该方法。 
- 该方法将Class文件加载到内存时，**并不会执行类的初始化，直到这个类第一次使用时才进行初始化**。该方法因为需要得到一个ClassLoader对象，所以可以根据需要指定使用哪个类加载器

### 再谈双亲委派模型
类加载器用来把类加载到Java虚拟机中。**从JDK1.2版本开始，类的加载过程采用双亲委派机制，这种机制能更好地保证Java平台的安全**。

> 如果一个类加载器在接到加载类的请求时，它首先不会自己尝试去加载这个类，而是把这个请求任务委托给父类加载器去完成，依次递归，如果父类加载器可以完成类加载任务，就成功返回。只有父类加载器无法完成此加载任务时，就自己去加载。

?> 规定了类加载的顺序是：**引导类加载器先加载，若加载不到，由扩展类加载器加载，若还加载不到，才会由系统类加载器或自定义的类加载器进行加载**。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/双亲委派模型的示例.png)


**优势**   
- 避免类的重复加载，确保一个类的全局唯一性
- Java类随着它的类加载器一起具备了一种带有优先级的层次关系，通过这种层级关可以避免类的重复加载，当父亲已经加载了该类时，就没有必要子ClassLoader再加载一次
- 保护程序安全，防止核心API被随意篡改

代码支持：双亲委派机制在java.lang.ClassLoader.loadClass(String，boolean)接口中体现(上面有)


**思考**   
如果在自定义的类加载器中重写`java.lang.ClassLoader.loadClass(String)`或`java.lang.ClassLoader.loadclass(String，boolean)`方法，抹去其中的双亲委派机制，仅保留上面这4步中的第l步与第4步，那么是不是就能够加载核心类库了呢？

这也不行！因为JDK还为核心类库提供了一层保护机制。不管是自定义的类加载器，还是系统类加载器抑或扩展类加载器，最终都必须调用 `java.lang.ClassLoader.defineclass(String，byte[]，int，int，ProtectionDomain)`方法，而该方法会执行`preDefineClass()`接口，该接口中提供了对JDK核心类库的保护。

**弊端**   
检查类是否加载的委托过程是**单向的**，这个方式虽然从结构上说比较清晰，使各个ClassLoader的职责非常明确，但是同时会带来一个问题，即**顶层的ClassLoader无法访问底层的ClassLoader所加载的类**。

通常情况下，启动类加载器中的类为系统核心类，包括一些重要的系统接口，而在应用类加载器中，为应用类。按照这种模式，应用类访问系统类自然是没有问题，但是**系统类访问应用类就会出现问题**。   
比如在系统类中提供了一个接口，该接口需要在应用类中得以实现，该接口还绑定一个工厂方法，用于创建该接口的实例，而接口和工厂方法都在启动类加载器中。这时，**就会出现该工厂方法无法创建由应用类加载器加载的应用实例的问题**

**结论**  
**由于Java虚拟机规范并没有明确要求类加载器的加载机制一定要使用双亲委派模型，只是建议采用这种方式而已**。    
比如在Tomcat中，类加载器所采用的加载机制就和传统的双亲委派模型有一定区别，当缺省的类加载器接收到一个类的加载任务时，首先会由它自行加载，当它加载失败时，才会将类的加载任务委派给它的超类加载器去执行，这同时也是Serylet规范推荐的一种做法。  

### 破坏双亲委派机制
《深入理解JVM虚拟机第三版》7.4.3　破坏双亲委派模型

> 双亲委派模型并不是一个具有强制性约束的模型，而是Java设计者推荐给开发者们的类加载器实现方式。在Java的世界中大部分的类加载器都遵循这个模型，但也有例外的情况，直到Java模块化出现为止，**双亲委派模型主要出现过3次较大规模`被破坏`的情况**

- JDK1.2之前重写loadClass造成破坏
- 上层类加载器无法调用下层类加载器加载的类，为了解决这个问题引入了线程上下文类加载器`Thread Context ClassLoader`
- OSGi中的类加载器的设计不符合传统的双亲委派的类加载器架构

### 热替换的实现
热替换是指在程序的运行过程中，不停止服务，只通过替换程序文件来修改程序的行为。**热替换的关键需求在于服务不能中断，修改必须立即表现正在运行的系统之中**。   

基本上大部分脚本语言都是天生支持热替换的，比如：PHP，只要替换了PHP源文件，这种改动就会立即生效，而无需重启Web服务器。

但对Java来说，热替换并非天生就支持，如果一个类已经加载到系统中，通过修改类文件，并无法让系统再来加载并重定义这个类。因此，在Java中实现这一功能的一个可行的方法就是灵活运用ClassLoader。

注意：由不同ClassLoader加载的同名类属于不同的类型，不能相互转换和兼容。即两个不同的ClassLoader加载同一个类，在虚拟机内部，会认为这2个类是完全不同的。

根据这个特点，可以用来模拟热替换的实现，基本思路如下图所示：

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/热替换的实现.png)

### 自定义类的加载器

#### 为什么要自定义类加载器？

- **隔离加载类**
  - 在某些框架内进行中间件与应用的模块隔离，把类加载到不同的环境。
  - 比如:阿里内某容器框架通过自定义类加载器确保应用中依赖的jar包不会影响到中间件运行时使用的jar包。
  - 比如:Tomcat这类Web应用服务器，内部自定义了好几种类加载器，用于隔离同一个Web应用服务器上的不同应用程序。 
- **修改类加载的方式**
  - 类的加载模型并非强制，除Bootstrap外，其他的加载并非一定要引入，或者根据实际情况在某个时间点进行按需进行动态加载 
- **扩展加载源**
  - 比如从数据库、网络、甚至是电视机机顶盒进行加载 
- **防止源码泄漏**
  - Java代码容易被编译和篡改，可以进行编译加密。那么类加载也需要自定义，还原加密的字节码。 

**常见的场景**

- 实现类似进程内隔离，类加载器实际上用作不同的命名空间，以提供类似容器、模块化的效果。例如，两个模块依赖于某个类库的不同版本，如果分别被不同的容器加载，就可以互不干扰。这个方面的集大成者是JavaEE和OSGI、JPMS等框架。
- 应用需要从不同的数据源获取类定义信息，例如网络数据源，而不是本地文件系统。或者是需要自己操纵字节码，动态修改或者生成类型。

**注意**  

在一般情况下，使用不同的类加载器去加载不同的功能模块，会提高应用程序的安全性。但是，如果涉及Java类型转换，则加载器反而容易产生不好的事情。    
**在做Java类型转换时，只有两个类型都是由同一个加载器所加载，才能进行类型转换，否则转换时会发生异常**

#### 实现方式
Java提供了抽象类`java.lang.ClassLoader`，所有用户自定义的类加载器都应该继承`ClassLoader`类。    
在自定义ClassLoader的子类时候，我们常见的会有两种做法:
- 方式一:重写`loadClass()`方法
- 方式二:重写`findclass()`方法(建议)

**对比**    
- 这两种方法本质上差不多，毕竟`loadClass()`也会调用`findClass()`，但是从逻辑上讲我们最好不要直接修改`loadClass()`的内部逻辑。
  - 建议的做法是只在findClass()里重写自定义类的加载方法，根据参数指定类的名字，返回对应的Class对象的引用。
- loadclass()这个方法是实现双亲委派模型逻辑的地方，擅自修改这个方法会导致模型被破坏，容易造成问题。因此我们最好是**在双亲委派模型框架内进行小范围的改动**，不破坏原有的稳定结构。同时，也避免了自己重写loadClass()方法的过程中必须写双亲委托的重复代码，从代码的复用性来看，不直接修改这个方法始终是比较好的选择。
- 当编写好自定义类加载器后，便可以在程序中调用loadClass()方法来实现类加载操作。

说明

- 其父类加载器是系统类加载器(应用程序类加载器AppClassLoader)
- JVM中的所有类加载都会使用java.lang.ClassLoader.loadClass(String)接口(自定义类加载器并重写java.lang.ClassLoader.loadClass(String)接口的除外)，连JDK的核心类库也不能例外。


```java
public class MyClassLoader extends ClassLoader {
    //class文件路径
    private String byteCodePath;

    public MyClassLoader(String byteCodePath) {
        this.byteCodePath = byteCodePath;
    }

    public MyClassLoader(ClassLoader parent, String byteCodePath) {
        super(parent);
        this.byteCodePath = byteCodePath;
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        String fileName = byteCodePath + "xxx.class";//class地址
        BufferedInputStream bis = null;

        ByteArrayOutputStream bos = null;
        try {
            //获取输入流
            bis = new BufferedInputStream(new FileInputStream(fileName));
            //输出流
            bos = new ByteArrayOutputStream();

            int len;
            byte[] data = new byte[1024];
            while ((len = bis.read(data)) != -1) {
                bos.write(data, 0, len);
            }
            //获取内存中的字节数组数据
            byte[] bytes = bos.toByteArray();
            //调用 defineclass
            Class<?> aClass = defineClass(null, bytes, 0, bytes.length);
            return aClass;
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                bis.close();
                bos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return null;
    }
}
```  

### JDK9中的模块化和类加载器
为了保证兼容性，JDK9没有从根本上改变三层类加载器架构和双亲委派模型，但为了模块化系统的顺利运行，仍然发生了一些值得被注意的变动。

1. 扩展机制被移除，扩展类加载器由于向后兼容性的原因被保留，不过被重命名为平台类加载器(platform class loader)。可以通过classLoader的新方法getPlatformClassLoader()来获取。
2. **JDK9时基于模块化进行构建**(原来的rt.jar和tools.jar被拆分成数十个JMOD文件)，其中的Java类库就已天然地满足了可扩展的需求，那自然无须再保留`<JAVA_HOME>\lib\ext`目录，此前使用这个目录或者java.ext.dirs系统变量来扩展JDK功能的机制已经没有继续存在的价值了。 
3. 平台类加载器和应用程序类加载器都不再继承自`java.net.URLClassLoader`。现在启动类加载器、平台类加载器、应用程序类加载器全都继承于`jdk.internal.loader.BuiltinClassLoader`。 
4. 在Java9中，类加载器有了名称。该名称在构造方法中指定，可以通过getName()方法来获取。平台类加载器的名称是platform，应用类加载器的名称是app。类加载器的名称在调试与类加载器相关的问题时会非常有用。
5. 启动类加载器现在是在jvm内部和java类库共同协作实现的类加载器（以前是C++实现），但为了与之前代码兼容，在获取启动类加载器的场景中仍然会返回null，而不会得到BootClassLoader实例。
6. 类加载的委派关系也发生了变动。当平台及应用程序类加载器收到类加载请求，在委派给父加载器加载前，要先判断该类是否能够归属到某一个系统模块中，如果可以找到这样的归属关系，就要优先委派给负责那个模块的加载器完成加载。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/JDK9类加载机制.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/JDK9类加载机制和之前对比.png)

## 沙箱安全机制

沙箱安全机制

- 保证程序安全
- 保护Java原生的JDK代码

Java安全模型的核心就是Java沙箱sandbox。什么是沙箱？**沙箱是一个限制程序运行的环境**。

沙箱机制就是将Java代码 限定在虚拟机（JVM）特定的运行范围中，并且严格限制代码对本地系统资源访问。通过这样的措施来保证对代码的有限隔离，防止对本地系统造成破坏。   
沙箱主要限制系统资源访问，那系统资源包括什么？CPU、内存、文件系统、网络。不同级别的沙箱对这些资源访问的限制也可以不一样。   
所有的Java程序运行都可以指定沙箱，可以定制安全策略   

### JDK1.0时期
在Java中将执行程序分成`本地代码`和`远程代码`两种，本地代码默认视为可信任的，而远程代码则被看作是不受信的。对于授信的本地代码，可以访问一切本地资源。而对于非授信的远程代码在早期的Java实现中，安全依赖于沙箱（Sandbox）机制。如下图所示JDK1.0安全模型

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/Java沙箱安全.png)

### JDK1.1时期
JDK1.0中如此严格的安全机制也给程序的功能扩展带来障碍，比如当用户希望远程代码访问本地系统的文件时候，就无法实现。    
因此在后续的Java1.1版本中，针对安全机制做了改进，增加了安全策略。允许用户指定代码对本地资源的访问权限。   
如下图所示JDK1.1安全模型  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/Java沙箱安全1.png)

### JDK1.2时期
在Java1.2版本中，再次改进了安全机制，增加了代码签名。不论本地代码或是远程代码，都会按照用户的安全策略设定，由类加载器加载到虚拟机中权限不同的运行空间，来实现差异化的代码执行权限控制。如下图所示JDK1.2安全模型：

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/Java沙箱安全2.png)

### JDK1.6时期
当前最新的安全机制实现，则引入了域（Domain）的概念。

虚拟机会把所有代码加载到不同的系统域和应用域。系统域部分专门负责与关键资源进行交互，而各个应用域部分则通过系统域的部分代理来对各种需要的资源进行访问。虚拟机中不同的受保护域（Protected Domain），对应不一样的权限（Permission）。存在于不同域中的类文件就具有了当前域的全部权限，如下图所示，最新的安全模型（jdk1.6）

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/Java沙箱安全3.png)
























# 参考资料
> - [尚硅谷JVM](https://www.bilibili.com/video/BV1PJ411n7xZ)
> - [Java 虚拟机底层原理知识总结](https://doocs.gitee.io/jvm/)
> - [《深入理解JVM虚拟机第三版》]()
> - [《深入理解JVM虚拟机第三版》的勘误仓库](https://github.com/fenixsoft/jvm_book)