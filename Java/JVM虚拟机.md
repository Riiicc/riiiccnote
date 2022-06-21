# 一、JVM整体结构，执行流程，生命周期
## 整体结构
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jvm3.png)

## java代码执行流程
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jvm2.png)

### 两种指令集架构
- 基于栈式架构
    - 大部分零地址指令，**指令集小，指令多，跨平台**，性能比寄存器架构略低
- 基于PC寄存器
    - 对硬件依赖大，**可移植性差，耦合高，性能高**，
    - 基于寄存器架构的指令集往往都是_一地址指令，二地址指令，三地址指令_为主

## JVM生命周期
- 启动，通过引导类加载器创建初始类来完成
- 执行
- 结束
    - 执行完成
    - 异常，错误
    - 操作系统错误
    - 调用`Runtime/System.exit()/halt()` 
  
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
- ClassLoader只负责class文件的加载，是否可以运行由`ExecutionEngine`决定 
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

> **疑问：** 上面说到常量在编辑阶段会存入常量池，引用定义常量的类不会触发类初始化，
> 而接口中的常量默认为public static final 为什么会导致父类接口初始化呢，那么如果直接调用子接口中的常量 会导致子接口初始化吗

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
- **数组类型**不通过类加载器创建，由java虚拟机直接在内存中动态构造出来 

#### 数组类型加载详解 

- 如果数组的组件类型是引用类型，那就递归采用通用加载过程去加载这个组件类型，数组将会被标识在加载该组件类型的类加载器的类名称空间上
- 如果数组的组件类型不是引用类型（如 `int[]`），java虚拟机将会把数组标记为于引导类加载器关联

#### 其他
> **加载阶段**与**连接阶段**的部分动作（如 字节码格式验证动作）是交叉进行的，加载阶段尚未完成，连接阶段可能已经开始，但这些加载阶段的动作仍然是连接阶段的一部分，这两个阶段的**开始时间**仍然保持着固定的先后顺序


### 验证
> **验证是连接阶段的第一步**，这一阶段的目的是确保Class文件的字节流中包含的信息符合《Java虚拟机规范》的全部约束要求，保证这些信息被当作代码运行后不会危害虚拟机自身的安全。

1. 文件格式验证
   - 是否以魔数`0xCAFEBABE`开头
   - 主次版本号是否在当前Java虚拟机接受范围之内
   - 常量池的常量中是否有不被支持的常量类型
   - 指向常量的各种索引值中是否有指向不存在的常量或者不符合类型的常量
   - 。。。
2. 元数据验证
> 对字节码描述的信息进行**语义分析**
- 类是否有父类
- 类的父类是否继承了不允许被继承的类
- 如果这个类不是抽象类，是否实现了其父类或者接口中要求实现的所有方法
- 类的字段、方法是否与父类产生矛盾（例如覆盖了父类的final字段，或者出现不符合规则的方法重载，例如方法参数都一致，但返回值类型却不同等）
3. 字节码验证
> 第三阶段是整个验证过程中最复杂的一个阶段，主要目的是通过数据流分析和控制流分析，确定程序语义是合法的、符合逻辑的。

4. 符号引用验证
> 符号引用验证可以看作是对类自身以外（常量池中的各种符号引用）的各类信息进行匹配性校验，通俗来说就是，该类是否缺少或者被禁止访问它依赖的某些外部类、方法、字段等资源

> 如果无法通过符号引用验证，Java虚拟机将会抛出一个java.lang.IncompatibleClassChangeError的子类异常，典型的如：java.lang.IllegalAccessError、java.lang.NoSuchFieldError、java.lang.NoSuchMethodError等。


### 准备
> 准备阶段是正式为类中定义的变量（即静态变量，被static修饰的变量）分配内存并设置类变量初始值的阶段

> 这时候进行内存分配的仅包括**类变量**，而不包括**实例变量**，实例变量将会在对象实例化时随着对象一起分配在Java堆中。
> 其次是这里所说的初始值“通常情况”下是数据类型的**零值**，假设一个类变量的定义为：
```java
    public static int value = 123;
```
> 变量value在准备阶段过后的**初始值为0而不是123**，因为这时尚未开始执行任何Java方法，而把value赋值为123的putstatic指令是程序被编译后，存放于类构造器<clinit>()方法之中，所以把value赋值为123的动作要到类的初始化阶段才会被执行。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jvm5.png) 

> **关于初始值的特殊情况：**如果类字段的字段属性表中存在`ConstantValue`属性，那在准备阶段变量值就会被初始化为`ConstantValue`属性所指定的初始值，下面类变量value的初始值为123
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

### 解析
解析阶段是Java虚拟机将常量池内的符号引用替换为直接引用的过程  
- 符号引用（Symbolic References）：符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可。
- 直接引用（Direct References）：直接引用是可以直接指向目标的指针、相对偏移量或者是一个能间接定位到目标的句柄。
> 解析动作主要针对**类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符**这7类符号引用进行，
> 分别对应于常量池的`CONSTANT_Class_info`、`CON-STANT_Fieldref_info`、`CONSTANT_Methodref_info`、`CONSTANT_InterfaceMethodref_info`、`CONSTANT_MethodType_info`、`CONSTANT_MethodHandle_info`、`CONSTANT_Dyna-mic_info`和`CONSTANT_InvokeDynamic_info` 8种常量类型 



### 初始化
> 初始化阶段就是执行类构造器`<clinit>()`方法的过程  

> `<clinit>()`方法是由编译器自动收集类中的所有类变量的赋值动作和静态语句块（`static{}`块）中的语句合并产生的，
> 编译器收集的顺序是由语句在源文件中出现的顺序决定的，静态语句块中只能访问到定义在静态语句块之前的变量，定义在它之后的变量，在前面的静态语句块可以赋值，但是不能访问
```java
public class Test{
    static{
        i = 0; // 给变量复制可以正常编译通过
        System.out.print(i); // 这句编译器会提示“非法向前引用”
    }
    static int i = 1;
}
```
> `<clinit>()`方法与类的构造函数（即在虚拟机视角中的实例构造器`<init>()`方法）不同，它不需要显式地调用父类构造器，
> Java虚拟机会保证在子类的`<clinit>()`方法执行前，父类的`<clinit>()`方法已经执行完毕。因此在Java虚拟机中第一个被执行的`<clinit>()`方法的类型肯定是`java.lang.Object` 

> 由于父类的`<clinit>()`方法先执行，也就意味着父类中定义的静态语句块要优先于子类的变量赋值操作

> `<clinit>()`方法对于类或接口来说并不是必需的，如果一个类中没有静态语句块，也没有对变量的赋值操作，那么编译器可以不为这个类生成`<clinit>()`方法

> 接口中不能使用静态代码块，但接口也需要通过 `<clinit>()` 方法为接口中定义的静态成员变量显式初始化。但接口与类不同，接口的 `<clinit>()` 方法不需要先执行父类的 `<clinit>()` 方法，只有当父接口中定义的变量使用时，父接口才会初始化

> 虚拟机会保证一个类的 `<clinit>()` 方法在多线程环境中被正确加锁、同步。如果多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的 `<clinit>()` 方法。


## 类加载器 Class Loader 

> 对于任意一个类，都必须由**加载它的类加载器**和**类本身**一起确立其在java虚拟机中的唯一性，比较两个类是否相同，只有在这两个类是由同一个类加载器加载的前提下才有意义，两个类来源于同一个Class文件，被同一个Java虚拟机加载，**只要加载它们的类加载器不同，那这两个类就必定不相等**

> 这里所指的“相等”，包括代表类的Class对象的equals()方法、isAssignableFrom()方法、isInstance()方法的返回结果，也包括了使用instanceof关键字做对象所属关系判定等各种情况

JVM支持两种类加载器，一种是启动类加载器 (BootstrapClassLoader),另一种是剩余的所有类加载器

### 启动（引导）类加载器 Bootstrap Class Loader
- 使用C/C++语言实现，嵌套在JVM内部
- 负责加载存放在`<JAVA_HOME>\lib`目录，或者被`-Xbootclasspath`参数所指定的路径中存放的，而且是Java虚拟机能够识别的（按照文件名识别，如rt.jar、tools.jar，名字不符合的类库即使放在lib目录中也不会被加载）类库加载到虚拟机的内存中  

### 扩展类加载器Extension Class Loader
这个类加载器是在类`sun.misc.Launcher$ExtClassLoader`中以Java代码的形式实现的。它负责加载`<JAVA_HOME>\lib\ext`目录中，或者被`java.ext.dirs`系统变量所指定的路径中所有的类库。

### 应用程序类加载器 Application Class Loader
由于这个类加载器是 ClassLoader 中的 getSystemClassLoader() 方法的返回值，所以一般也称它为“系统类加载器”。它负责加载用户类路径（classpath）上所指定的类库，开发者可以直接使用这个类加载器，如果应用程序没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jvm7.png)

> 上图中展示的各种类加载器之间的层次关系被称为类加载器的“双亲委派模型（Parents DelegationModel）”。双亲委派模型要求除了顶层的启动类加载器外，**其余的类加载器**都应有自己的父类加载器。

### 双亲委派模型
类加载器的双亲委派模型在JDK1.2时期被引入

> 如果一个类加载器收到了类加载请求，它首先不会自己去尝试加载这个类，二十八这个请求委派给父类加载器去完成，父类再去委派父类，直到顶层的启动类加载器，只有当父类加载器反馈自己无法完成这个加载请求（范围内找不到对应的类） 子类加载器才会尝试自己去加载。

用处：  
- 避免类的重复加载
- 保护程序安全，防止核心API被篡改（如：自定义类`java.lang.String`）  


> 像 java.lang.Object 这些存放在 rt.jar 中的类，无论使用哪个类加载器加载，最终都会委派给最顶端的启动类加载器加载，从而使得不同加载器加载的 Object 类都是同一个。

>相反，如果没有使用双亲委派模型，由各个类加载器自行去加载的话，如果用户自己编写了一个称为 java.lang.Object 的类，并放在 classpath 下，那么系统将会出现多个不同的 Object 类，Java 类型体系中最基础的行为也就无法保证。


### 自定义类加载器
- 继承抽象类`java.lang.ClassLoader`，重写`loadClass()`方法或者`findClass()`方法
- 继承URLClassLoader类 




# 三、运行时数据区
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jvm1.png) 
## 程序计数器（PC寄存器）
`程序计数器（Program Counter Register）`是一块较小的内存空间，是当前线程正在执行的那条字节码指令的地址，若当前线程正在执行的是一个本地方法(Native Method)，那么此时程序计数器为`Undefined`  

Java虚拟机的多线程是通过线程轮流切换、分配处理器执行时间的方式来实现的，在任何一个确定的时刻，一个处理器（对于多核处理器来说是一个内核）都只会执**行一条线程中的指令**。因此，为了线程切换后能恢复到正确的执行位置，**每条线程都需要有一个独立的程序计数器，各条线程之间计数器互不影响，独立存储**，我们称这类内存区域为“线程私有”的内存。

### 作用
- 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制。
- 在多线程情况下，程序计数器记录的是当**前线程执行的位置**，从而当线程切换回来时，就知道上次线程执行到哪了。

### 特点
- 是一块较小的内存空间。
- 线程私有，每条线程都有自己的程序计数器。
- 生命周期：随着线程的创建而创建，随着线程的结束而销毁。
- 是唯一个不会出现`OutOfMemoryError`的内存区域


## Java虚拟机栈
虚拟机栈描述的是Java方法执行的线程内存模型：每个方法被执行的时候，Java虚拟机都会同步创建一个栈帧 `（Stack Frame）`用于**存储局部变量表、操作数栈、动态连接、方法出口等信息**。每一个方法被调用直至执行完毕的过程，就对应着一个**栈帧在虚拟机栈中从入栈到出栈**的过程。

- 每个线程都一个虚拟机栈，线程私有，，随着线程创建而创建，随着线程的结束而销，不存在数据一致问题和同步锁问题  
- 运行速度特别快,仅仅次于 PC 寄存器。

### 虚拟机栈中的异常
- 两类异常状况：
  - 如果线程请求的栈深度大于虚拟机所允许的深度（请求分配的栈容量超过虚拟机栈允许的最大容量），将抛出StackOverflowError异常；
  - **如果**Java虚拟机栈容量可以动态扩展，当栈扩展时无法申请到足够的内存会抛出`OutOfMemoryError`异常

> 可以使用参数 `-Xss` 来设置线程的最大栈空间，栈的大小直接决定了函数调用的最大可用深度
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

> **HotSpot虚拟机的栈容量是不可以动态扩展的**，以前的Classic虚拟机倒是可以。所以在HotSpot虚拟机上是不会由于虚拟机栈无法扩展而导致`OutOfMemoryError`异常——只要线程申请栈空间成功了就不会有OOM，但是如果申请时就失败，仍然是会出现OOM异常的

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jvm8.png)

**2022-1-26写到这里 下面的有待优化**
### 局部变量表 Local Variables Table
局部变量表（Local Variables Table）是一组变量值的存储空间，用于存放方法参数和方法内部定义的局部变量。

局部变量表存放了编译期可知的各种Java虚拟机基本数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference类型，它并不等同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或者其他与此对象相关的位置）和returnAddress类型（指向了一条字节码指令的地址）。  

这些数据类型在局部变量表中的存储空间以局部变量槽（Slot）来表示，其中64位长度的long和double类型的数据会占用两个变量槽，其余的数据类型只占用一个。局部变量表所需的内存空间在编译期间完成分配，当进入一个方法时，这个方法需要在栈帧中分配多大的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表的大小（变量槽的数量）。 

### 操作数栈

### 动态连接

### 方法出口（方法返回地址）  


## 本地方法栈

## Java堆 Heap

## 方法区

## 运行时常量池

## 直接内存  

## 参考资料
> - [《深入理解JVM虚拟机第三版》]()
> - [Java 虚拟机底层原理知识总结](https://doocs.gitee.io/jvm/)
