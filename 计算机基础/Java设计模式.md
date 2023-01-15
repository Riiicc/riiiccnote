
# 待办
- P64 spring mvc 中的适配器模式 
- 美团技术团队2022年货中关于设计模式的理解补充

# 设计模式
设计模式可以让代码具有更好的  
- 代码重用性
- 可读性
- 可扩展性
- 可靠性
- 使程序呈现高内聚，低耦合的特性

# 设计模式七大原则   
## 一句话概括
- `开闭原则`：对扩展开放，对修改关闭。扩展功能使用扩展类，不要修改源代码
- `单一职责原则`：一个类只有一种职责，只有一个引起它变化的原因
- `接口隔离原则`：细化分解大接口，客户端尽量不依赖不需要的接口
- `依赖倒转原则`：尽量面向接口编程，要依赖抽象类，而不是具体类
- `里氏替换原则`：替换就是子类可以替换父类，定义类型时尽量使用父类
- `迪米特法则`：最少知道原则，两个类不必直接通信就避免直接调用
- `合成复用原则`：尽量使用组合或聚合，避免使用继承

## 开闭原则
`Open Closed Principle` 开闭原则，是编程中最基础、最重要的设计原则    
一个软件实体，如类，模块和函数应该是对**扩展开放(提供方)，**对**修改关闭（使用方）**，也就是类中的函数应该是可扩展的，但是扩展后不能对使用其函数的方法造成影响，所以要用抽象构建框架，用实现扩展细节  

当软件需要变化时，**尽量通过扩展软件实体的行为来实现变化**，而不是通过修改已有的代码来实现变化,**扩展**而不是修改  

**对扩展开放，对修改关闭**，扩展类不能修改原有代码，通过接口和抽象类来实现


?> 编程中遵循其他原则，以及使用设计模式的目的就是遵循开闭原则   

## 单一职责原则
对类来说，一个类只应该负责一项职责，若类A负责两个不同职责，职责1、职责2，若因职责1需求变更而修改A时，可能会影响职责2的实现执行，所以要将类A的粒度分解为A1和A2   

> 类承担的职责越多，被复用的可能越小，职责过多就是过度耦合

1. 创建一个交通工具类,创建方法`run`输出"在路上跑"
2. 并不是所有交通工具都在路上，于是创建 水上交通工具，空中交通工具，陆地交通工具三个类，`run` 方法不同,输出结果不同`类的粒度分解`
3. 再次改进，仍然只有一个交通工具类，创建三个不同方法 水上方法，空中方法，陆地方法，同样可以实现不同工具执行不同结果`方法的粒度分解`

- 主要作用是降低类的复杂度，一个类只负责一项职责
- 提高类的可读性，可维护性
- 降低改变代码引起的风险
- 通常情况下应当遵守单一职责原则，即**使用类进行分解**；只有类中方法足够少，逻辑简单，才能在方法上进行粒度分解


## 接口隔离原则 
`Interface Segregation Principle` 客户端不应该依赖它不需要的接口，即一个类对另一类的依赖应该建立在最小的接口上  

1. 一个接口有1，2，3，4，5个方法，B类和D类实现了接口
2. A类通过B类调用接口的1，2，3方法
3. C类通过D类调用接口的1，4，5方法
4. 此时依据接口隔离原则，需要将接口拆分成几个独立接口
5. 接口1 包含方法1，接口2包含方法2，3,接口3 包含方法4，5 
6. B类 实现 接口1 和 接口2，D类实现 接口1 和 接口3 ，这样就实现了接口隔离，

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/接口隔离优化.drawio.png)

## 依赖倒转原则
`Dependence INversion Principle` 依赖倒转原则  

- 高层模块不应该依赖底层模块，二者都应该依赖其抽象
- 抽象不应该依赖细节，细节应该依赖抽象
- 依赖倒转的中心思想是**面向接口编程**
- 使用接口或抽象类就是为了制定好规范，具体实现细节交由实现类去完成

```java
public class Dependence {
    public static void main(String[] args) {
        Person person = new Person();
        person.receive(new Email() );
    }
}
class Email{
    public String getInfo(){
        return "email";
    }
}
//完成Person 接受消息的功能
class Person {
    public void receive(Email email){
        System.out.println(email.getInfo());
    }
}

interface IReceiver{
    String getInfo();
}

class Email implements IReceiver{
    public String getInfo(){
        return "email";
    }
}
//完成Person 接受消息的功能
class Person {
    public void receive(IReceiver email){
        System.out.println(email.getInfo());
    }
}
```

**依赖关系传递的三种方式**    
- 接口传递(上面的案例) 
- 构造方法传递，通过构造方法
- setter方式传递,通过set方法

**注意事项**   
- 底层模块尽量都有抽象类或接口
- **变量的声明类型尽量是抽象类或者接口**,这样我们引用变量和实际对象之间就存在一个缓冲，利于程序优化和扩展
- 继承时遵循里氏替换原则   

**面型接口编程，依赖的类应该是抽象类或者接口**

## 里氏替换原则
继承中，若父类已经实现好的方法，实际上是在设定规范和契约，虽然它不强制要求所有子类必须遵守契约，但是若子类对这些已经实现的方法进行修改，就会对整个继承体系造成破坏，**继承会给程序带来侵入性，可移植性降低，耦合性增加**

想要在编程中正确使用继承，就需要遵循 **里氏替换原则**

> 如果对每个类型为T1的对象O1，都有类型为T2的对象O2,使得T1定义的所有程序P在所有对象O1都代换成O2时，程序P的行为没有发生变化，那么类型T2就是T1的子类型，**所有引用基类的方法必须能够透明的使用其子类型的对象**

**使用继承时，遵循里氏替换原则，在子类中尽量不要重写父类的方法**   
**继承实际上让两个类耦合性增强了，可以通过`聚合、组合、依赖`来解决问题**


## 迪米特法则
迪米特法则`Law of Demoeter ,LoD`也称为最少知识原则  
- 不要和陌生人说话，只与直接朋友通信  

其中的**朋友**我们定义为： 
- 当前对象本身
- 以参数形式掺入到当前对象方法中的对象
- 当前对象的成员对象
- 如果当前对象是一个集合，那么集合中的元素也都是朋友
- 当前对象所创建的对象

迪米特发自主要用于控制信息的过载，运用迪米特法则主要注意以下几点：  
- 在类的划分上，应当尽量创建松耦合的类，类之间耦合度越低，就越有利于复用
- 类结构设计上，应尽量减低其成员变量和函数的访问权限
- 尽量将类设计为不变类
- 在对其他类的引用上，一个对象对其他对象的引用应当降到最低  


## 合成复用原则
尽量使用合成/聚合的方式，而不是使用继承


# UML图
`UML` `Unified modeling language` 统一建模语言 
UML 本身是一套符号的规定，就像数学符号和化学符号，这些符号用于描述软件模型中的各个元素和他们之间的关系，如类、接口、实现、泛化、依赖、组合、聚合等   

> UML2.0中，提供了13种图 `用例图` `类图` `对象图` `包图` `组合结构图` `状态图` `活动图` `顺序图` `通信图` `定时图` `交互概览图` `组件图` `部署图`
 
**当前重点是类图，其他略过**

## UML类图
用于描述系统中的类(对象)本身的组成和类(对象)之间的各种竞态关系
类之间的关系： 依赖、泛化(继承)、实现、关联、聚合、组合   

简单示例   

```java
public class Person {
	private Integer id;
	private String name;
	
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
}
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/personUml示例.png)  

## 说明  
UML类图中,类一般由三部分组成: 类名,属性,类的操作  

### 类名
每个类都必须有一个名字,类名是一个字符串,按照Java命名规范  

### 属性  
类的成员变量,属性表示方式为`可见性 名称:类型 [= 默认值]` 

可见性,表示属性对类外元素是否可见,包括公有`public`,私有`private`,受保护`protected`,在类图中分别用符号`+` `-` `#`表示   
Java中新增了包内可见性`package`使用`*`表示    

可见性，对应方法和属性前面的符号:  
- `private(-)`：在类的外部无法被访问（即使是它的子类、或者通过 类.私有进行访问 ）
- `protected(#)`：在子类中可以访问；
- `public(+)`：任何地方都可以访问它们
- `package(*)`包内访问

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/uml类图属性说明示意图.png)

### 类的操作 
操作是类的任意一个实例对象都可以使用的行为,操作是类的成员方法  
`可见性 名称 ([参数列表])[:返回类型]`   
可见性定义和属性的相同,名称即方法名称,参数列表多个参数之间用`,`隔开,返回类型是可选项,表示方法的返回值类型,依赖于具体语言,也可以是用户自定义类型,还可以是空`void` 

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/uml类图操作说明示意图.png)




## 类的关系 

线条说明：  
- `Generalization` 泛化关系-继承关系(实线三角)
- `Realization/Implementation` 实现关系(虚线三角)
- `Association` 关联关系(实线箭头)
- `Dependency` 依赖关系(虚线箭头)
- `Aggregation`聚合关系(实现空心菱形)
- `Composite` 组合关系(实现实心菱形)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/类图关系说明大图.png)

### 泛化关系
泛化关系实际上就是**继承关系**，也是依赖关系的特例，子类指向父类   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/uml泛化关系实例.png)

### 实现关系
实现关系实际上就是A类实现B接口，也是是依赖关系的特例，实现类指向被实现类  
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/uml实现关系实例.png)

### 依赖关系
依赖关系`Dependency`，是一种使用关系,只要在类中用到了对方，那么他们就存在依赖关系，A用到B，就成为A依赖B   
大部分情况下,依赖关系体现在某个类的方法使用另一个类的对象作为参数 

```java
class Driver{
    public void drive(Car car){
        car.move();
    }
}
class Car{
    public void move(){

    }
}
```
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/uml依赖关系实例.png)


### 关联关系
关联关系`Association`是类与类之间的关系，通常将一个类的对象作为另一个类的属性,关联具有导航性，即双向关联关系或者单向关联关系  

1. 双向关联,默认情况下关联是双向的,例如:人和身份证,顾客和商品  

```java
class Person{
    private IDCard card;
}
class IDCard{
    private Person person;
}
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/双向一对一关联关系.png)

```java
class Customer{
    private Product[] products;
}

class Product{
    private Customer cusomer;
}
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/uml双向关联关系顾客商品示例.png)  

2. 单项关联关系,单向关联关系用带箭头的实线表示,例:人和身份证,人和地址     

```java
//单向一对一 person用到idcard 但是idcard 没有 用到person
class Person{
    private IDCard card;
}
class IDCard{}
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/单向一对一关联关系.png)

```java
class Customer{
    private Address address;
}

class Address{
}
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/uml单向关联关系顾客地址示例.png)  

3. 自关联,类的属性是类本身 

```java
class Node{
    private Node subNode;
}
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/uml自关联示意图.png)

4. 多重关联,表示一个类的对象和另一个类的对象连接的个数 

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/uml多重关联示意图.png) 

```java
public class Form{
    private Button[] buttons;
}

public class Button{

}
```  
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/uml多重关联实例图.png) 


### 聚合关系
聚合关系`Aggregation` 表示整体和部分的关系，整体和部分可以分开，是关联关系的一种特例   
例如：一台电脑由键盘、显示器、鼠标等组成，这些配件是可以从电脑上分离的，所以他们就是聚合关系     
再比如： 羊群和羊的关系、学校和老师 就是聚合关系

```java
class Computer{
    private Mouse mouse;
    private Monitor monitor;

    public void setMouse(Mouse mouse) {
        this.mouse = mouse;
    }

    public void setMonitor(Monitor monitor) {
        this.monitor = monitor;
    }
}
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/聚合关系示例.png)

### 组合关系 
组合关系`Compostion`接上，若我们认为鼠标，显示器和电脑是不可分离的，那么他们就是组合关系   
再举例，人和身份证的关系就是聚合关系，和脑袋的关系就是组合关系    

**组合关系可以认为是更强烈的聚合关系**

```java
class Computer{
    private Mouse mouse = new Mouse();
    private Monitor monitor = new Monitor();

    public void setMouse(Mouse mouse) {
        this.mouse = mouse;
    }

    public void setMonitor(Monitor monitor) {
        this.monitor = monitor;
    }
}
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/组合关系示例.png)


## 其他
除此之外，我们还需要对它们之间的数量关系进行约束，也称为多重性数字约束。例如：一个访客大厅类只能有一个售票处，但是厕所要好几个。将约束关系写为`m..n`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/多对多uml关系说明.png)





# 设计模式类型 🔶
设计模式分为三种类型，共23种，`GOF 23`    
- **创建型**模式：单例模式、抽象工厂模式、原型模式、建造者模式、工厂模式
- **结构型**模式：适配器模式、桥接模式、装饰模式、组合模式、外观模式、享元模式、代理模式
- **行为型**模式：模板方法模式、命令模式、访问者模式、迭代器模式、观察者模式、中介者模式、备忘录模式、解释器模式、状态模式、策略模式、责任链模式  

?> 创建型模式关注对象的创建过程，结构型模式关注对象与类的组织，行为型模式关注对象之间的交互  

# 创建型模式🔵
软件系统在运行时，类将实例化成对象，并由这些对象来写作完成各项业务功能。**创建型模式**对类的实例化过程进行了抽象，能够将软件模块中对象的创建和对象的使用分离。  

为了使软件的结构更加清晰，外界对于这些对象只需要知道它们共同的接口，而不用清除其具体的实现细节，使整个系统的设计更加符合单一职责原则。  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/创建者模式一览.png)

# ❗单例模式🔵
单例设计模式，就是采取一定的方法**保证在整个软件系统中，对某个类只能存在一个对象实例**，并且该类只提供一个获得其对象实例的方法(静态方法)

例如Hibernate 的SessionFactory   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/单例模式UML.png)

## 八种方式
1. 饿汉式(静态常量)
2. 饿汉式(静态代码块)
3. 懒汉式(线程不安全)
4. 懒汉式(线程安全，同步方法)
5. 懒汉式(线程安全，同步代码块)
6. 双重检查
7. 静态内部类
8. 枚举

### 饿汉式
静态常量

```java
class Singleton {
    //构造器私有化，外部不能通过new 创建
    private Singleton() {}
    //本类内部创建对象实例
    private final static Singleton instance = new Singleton();

    //提供一个公有的竞态方法，返回实例对象
    public static Singleton getInstance() {
        return instance;
    }
}
```

静态代码块

```java
class Singleton {
    //构造器私有化，外部不能通过new 创建
    private Singleton() {}
    //本类内部创建对象实例
    private static Singleton instance;
    static {
        //在静态代码块种创建单例对象
        instance = new Singleton();
    }
    //提供一个公有的静态方法，返回实例对象
    public static Singleton getInstance() {
        return instance;
    }
}
```

> 上述两种均是利用类加载完成实例化，通过`<clinit>()`方法由编译器自动收集类中的所有类变量的赋值动作和静态语句块,在类加载(初始化)阶段完成赋值，也是饿汉名称由来，提前准备食物(类实例)，随时可以吃(使用)    

?> 优点： 写法简单，在类加载时就完成实例化，避免线程同步问题  

!> 缺点：在类加载的时候就完成实例化，没有达到懒加载效果，若从未使用过该实例，就会造成内存浪费


### 懒汉式
懒汉式，饿了(需要用时)才去准备食物(创建类实例)   

**线程不安全** 

```java
class Singleton{
    private static Singleton instance;
    private Singleton(){}
    //懒汉式
    //提供静态方法，在调用方法时创建对象
    public static Singleton getInstance(){
        if (instance==null){
            instance = new Singleton();
        }
        return instance;
    }
}
```  

优点： 起到了懒加载效果，但是只能在单线程下使用    
缺点： 多线程情况下可能创建多个，无法保证单例，实际开发种**不能**使用   

**同步方法保证线程安全**

```java
class Singleton{
    private static Singleton instance;
    private Singleton(){}
    //懒汉式
    //提供静态同步方法，在调用方法时创建对象
    public static synchronized Singleton getInstance(){
        if (instance==null){
            instance = new Singleton();
        }
        return instance;
    }
}
```

缺点： 效率低下，每次getInstance都会执行同步方法，实际开发不推荐

**同步代码块**

```java
class Singleton{
    private static Singleton instance;
    private Singleton(){}
    //懒汉式
    //提供静态方法，在调用方法时创建对象
    public static Singleton getInstance(){

        if (instance==null){
            //同步代码块
            synchroized{
                instance = new Singleton();
            }
        }
        return instance;
    }
}
```

这种方法无法解决多线程带来的问题，毫无作用

**双重检查**

```java
class Singleton {
    //使用volatile
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

**静态内部类**   
> 主类`Singleton`加载时，内部静态类不会自动初始化，只有调用静态内部类的方法，静态域，或者构造方法的时候才会加载静态内部类,而java类加载过程是线程安全的   

```java
class Singleton {
    private Singleton() {}
    //静态内部类
    private static class SingletonInstance{
        private static final Singleton INSTANCE = new Singleton();
    }
    //调用方法是 才会加载静态内部类，且此时是线程安全的（类加载机制）
    public static Singleton getInstance() {
        return SingletonInstance.INSTANCE;
    }
}
```

**枚举 单例模式**

```java
enum Singleton{
    INSTANCE;
    public void sayOK(){
        System.out.println("OK");
    }
}
```

调用`Singleton instance = Singleton.INSTANCE;` 返回的就是Singleton类，可以把`INSTANCE`理解为Singleton类的子类   
可以使用方法获取INSTANCE类 `Singleton instance2 = Enum.valueOf(Singleton.class, "INSTANCE");`

> 借助了JDK1.5 中的枚举类来实现单例模式，可以避免多线程问题，还能方式反序列化重新创建新的对象  
> **推荐使用**

**下面的案例更好理解**   

```java
public class SingletonTest {
    public static void main(String[] args) {
        Singleton instance = Singleton.INSTANCE;
        Singleton instance1 = Singleton.INSTANCE;
        Singleton instance2 = Enum.valueOf(Singleton.class, "INSTANCE");

        instance.setXx(new Person("11", 1));
        System.out.println(instance.getXx().getName());//11
        System.out.println(instance2.getXx().getName());//11

        instance2.setXx(new Person("22", 2));
        System.out.println(instance.getXx().getName());//22

        System.out.println(instance.getXx() == instance2.getXx());//true
        System.out.println(instance1.getXx().getName());//22
        System.out.println(instance2.getXx().getName());//22

    }
}

enum Singleton {
    INSTANCE;
    private Person xx;

    public Person getXx() {
        return xx;
    }
    public void setXx(Person xx) {
        this.xx = xx;
    }

    public void sayOK() {
        System.out.println("OK");
    }
}

class Person {
    private String name;
    private Integer id;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public Person(String name, Integer id) {
        this.name = name;
        this.id = id;
    }
}
```

## 实际应用

`java.lang.Runtime`类就使用了饿汉式进行单例加载   

```java
public class Runtime {
    private static Runtime currentRuntime = new Runtime();

    /**
     * Returns the runtime object associated with the current Java application.
     * Most of the methods of class <code>Runtime</code> are instance
     * methods and must be invoked with respect to the current runtime object.
     *
     * @return  the <code>Runtime</code> object associated with the current
     *          Java application.
     */
    public static Runtime getRuntime() {
        return currentRuntime;
    }

    /** Don't let anyone else instantiate this class */
    private Runtime() {}
    
    ...
}
```

## 注意事项
1. 单例模式保证了系统内存中该类只有一个对象，节省了系统资源，对于一些需要频繁创建销毁的对象，使用单例模式可以提高系统性能  
2. 当需要实例化一个单例类的时候，必须要记住使用响应的获取对象的方法，而不是使用new
3. 单例模式的使用场景：需要频繁进行创建和销毁的对象，创建对象耗时过多或耗费资源过多(重量级的对象)，工具类对象(经常使用的对象)，频繁访问数据库或者文件的对象(数据源，session工厂)

## 单例模式的反射攻击
偶然看到的博客，单例模式可以使用反射，创建不同的实例对象,实现打破单例规则      

```java
public class SingletonTest01 {
    public static void main(String[] args) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
        Singleton instance = Singleton.getInstance();
        Singleton instance1 = Singleton.getInstance();
        System.out.println(instance == instance1);//true

        Class<Singleton> singletonClass = Singleton.class;
        //获取构造器
        Constructor<Singleton> declaredConstructor = singletonClass.getDeclaredConstructor();
        //设置权限访问私有域
        declaredConstructor.setAccessible(true);
        //newInstance调用默认无参构造器
        Singleton singleton = declaredConstructor.newInstance();
        System.out.println(instance == singleton);//false
    }
}

class Singleton {
    //构造器私有化，外部不能通过new 创建
    private Singleton() {}
    //本类内部创建对象实例
    private final static Singleton instance = new Singleton();

    //提供一个公有的静态方法，返回实例对象
    public static Singleton getInstance() {
        return instance;
    }
}

//对Singleton进行改造
class Singleton {
    //构造器私有化，外部不能通过new 创建
    private Singleton() {
        if(instance!=null){
            throw new RuntimeException("禁止反射构造");
        }
    }
    //本类内部创建对象实例
    private final static Singleton instance = new Singleton();

    //提供一个公有的竞态方法，返回实例对象
    public static Singleton getInstance() {
        return instance;
    }
}

```

# ❗工厂模式🔵

## 简单工厂模式 
> 定义一个工厂类，可以根据参数不同返回不同类的实例，被创建的实例通常都具有共同的父类   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/简单工厂模式示意图.png)

实例： 开发一套图表库，可以为系统提供多种图表，柱状图，饼状图，折线图，使用简单工厂模式设计如图   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/简单工厂模式示例.png)

**再次简化**   
可以将，抽象父类和工厂类合并，将静态工厂方法移至抽象产品类中，

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/简单工厂模式再简化.png)

客户端可以直接通过调用抽象父类的静态工厂方法，根据参数不同创建不同的产品子类对象   

```java
//静态工厂案例  
public class ChartFactory{
    public static Chart getChart(String type){
        Chart chart = null;
        if(type.equals("histogram")){
            //设置柱状图
            chart = new HistogramChart();
            
        }else if(type.equals("pie")){
            //设置饼状图
            chart = new PieChart();
            
        }else if(type.equals("line")){
            //设置折线图
            chart = new LineChart();
            
        }
    }
}
```

## 工厂方法模式

工厂方法模式：定义一个用于创建对象的接口，但是让子类决定将哪一个类实例化，工厂方法模式让一个类的实例化延迟到其子类

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/工厂方法模式.png)

- `Product`(抽象产品) 它是定义产品的接口，是工厂方法模式所创建对象的超类型，也就是**产品对象**的公共父类
- `ConcreteProduct`(具体产品) 它实现了抽象产品接口，某种类型的具体产品由专门的具体工厂创建，**具体工厂和具体产品之间一一对应**  
- `Factory`(抽象工厂) 在抽象工厂类中声明了工厂方法`Factory Method`，用于返回一个产品，抽象工厂是工厂方法模式的核心，所有创建对象的工厂类都必须实现该接口 
- `ConcreateFactory`(具体工厂) 它是抽象工厂类的子类，实现了在抽象工厂中声明的工厂方法，并可由客户端调用，返回一个**具体产品实例**  

### 模式实现

1. 抽象工厂`Factory`，可以是接口也可以是抽象类，本例用接口,抽象工厂中声明工厂方法，但并未实现

```java
public interface Factory{
    public Product factroyMethod();
}
```

2. 具体工厂`ConcreateFactory` 实现工厂方法，不同具体工厂创建不同的具体产品`ConcreteProduct`  

```java
public class ConcreteFactory implements Factory{
    public Product factoryMethod(){
        return new ConcreteProduct();
    }
}
```

3. `Product`(抽象产品)和`ConcreteProduct`(具体产品)

```java

public interface Product{
    public void printProduct();
}

public class ConcreteProduct implements Product{
     
    @Override
    public void printProduct(){
        System.out.println("打印具体产品");
    };
}

```

4. 客户端调用方法

```java
public class Test{
    public static void main(String[] args){
        Factory factory;
        factory = new ConcreteFactory();
        Product product;
        product = factory.factoryMethod();
        product.printProduct();//调用实际方法
    }
}
```

### 适用场景

- **一个类不知道他所需要的对象的类：**客户端不需要知道具体产品类名，只需知道对应的工厂即可，通过具体工厂构建具体产品对象
- **一个类通过其子类来指定创建那个对象：**对于抽象工厂类只需提供一个创建产品的接口，由其子类来去欸的那个具体药创建的对象，利用面向对象的多态性和里氏替换原则，子类覆盖父类对象，使得系统容易扩展  
- 将创建对象的任务委托给多个工厂子类，客户端在使用时无需关心那一个工厂子类创建产品子类，需要是再动态指定


### 最终说明
每增加一个**具体产品**对应一个**具体工厂**，我们都要增加一个继承于工厂接口`Factory`的具体工厂类 `ConcreteFactory`   
所以，当产品数量上涨，系统中的类会**成倍**增加   

工厂方法模式和简单工厂模式虽然都是通过工厂来创建对象，他们之间最大的不同是——工厂方法模式在设计上完全完全符合`开闭原则`。  

### 使用场景
- 日志记录器，用户可以选择记录日志位置
- 数据库访问，不同的数据库连接池返回不同的连接  

```java
    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druid(){
        return  new DruidDataSource();
    }
```

- 设计一个连接服务器的框架，需要三个协议，"POP3"、"IMAP"、"HTTP"，可以把这三个作为产品类，共同实现一个接口

## 抽象工厂模式
抽象工厂：提供一个创建一系列相关或相互依赖对象的接口，而无需指定他们具体的类  

- `AbstractFactory` 抽象工厂，用于声明生成抽象产品的方法
- `ConcreteFactory` 具体工厂，实现抽象工厂定义的方法，具体实现一系列产品对象的创建
- `AbstractProduct` 抽象产品，定义一类产品对象的接口
- `ConcreteProduct` 具体产品，产品具体实现

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/抽象工厂模式示意图.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/抽象工厂模式结构图.png)

?> **抽象工厂模式和工厂模式的主要区别就是，抽象工厂有多个抽象产品类**  

> 抽象工厂模式和工厂方法模式一样，都符合`开闭原则`。但是不同的是，工厂方法模式在增加一个具体产品的时候，都要增加对应的工厂。但是抽象工厂模式只有在新增一个类型的具体产品时才需要新增工厂。也就是说，**工厂方法模式的一个工厂只能创建一个具体产品**。而**抽象工厂模式的一个工厂可以创建属于一类类型的多种具体产品**。**工厂创建产品的个数介于简单工厂模式和工厂方法模式之间**。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/抽象工厂模式uml.jpg)



## 三种工厂的主要区别
类别 | 抽象工厂 | 具体工厂| 抽象产品 | 具体产品 
---------|----------|---------|---------|---------
 简单工厂 | 1 | 0| 1| 1
 工厂模式 | 1 | N| 1| N
 抽象工厂 | 1 | N| N| N


## 常见的工厂模式  
- Spring的 Bean 工厂，IOC 通过`BeanFactory`对Bean 进行管理
- `Calendar.getInstance()`调用也采用了简单工厂模式  

```java
...
        Calendar cal = null;

        if (aLocale.hasExtensions()) {
            String caltype = aLocale.getUnicodeLocaleType("ca");
            if (caltype != null) {
                switch (caltype) {
                case "buddhist":
                cal = new BuddhistCalendar(zone, aLocale);
                    break;
                case "japanese":
                    cal = new JapaneseImperialCalendar(zone, aLocale);
                    break;
                case "gregory":
                    cal = new GregorianCalendar(zone, aLocale);
                    break;
                }
            }
        }
        ...
        return cal;
```

# ❗原型模式🔵
**原型模式**: 使用原型实例指定带创建的对象的类型，并且通过复制这个原型来创建新的对象    

> 案例参考，`Java`中`Object`类是所有类的基类，`Object`提供了一个`clone()`方法，该方法可以将一个`Java`对象进行复制，使用该方法需要实现接口`Cloneable`

原型模式是一种`创建型`设计模式，允许一个对象再创建另外一个可定制的对象，无需知道如何创建的细节   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/原型模式UML类图.png)

- `Prototype`抽象原型类，声明克隆方法的接口，可以是接口、抽象类
- `ConcretePrototype`具体原型类，实现`Prototype`中的克隆方法，在方法中返回自己的克隆对象
- `Clinet`客户类，实例化具体原型类后，通过其克隆方法得到多个相同对象   

## 应用 
原型模式用作快速创建大量的相同或者相似对象，简化创建结构，通过封装在原型中类的克隆方法来实现创建重复对象   

### Spring Bean 的 prototype 和 singleton  
> 若bean的scope 是 prototype 每次getbean 都会得到一个新对象，若是singleton使用getbean得到的就是同一个对象


## 浅克隆(拷贝)  
原型对象的成员变量是基本数据类型(`int` `byte` `double`...)，就会复制一份给克隆对象，若是引用对象，就会引用相同的内存地址(本质上是同一个对象)    
**基本数据和对象本身复制，引用对象仍是同一个**  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/浅克隆示意图.png)

## 深克隆(拷贝)  
无论对象的成员变量是值类型还是引用类型，都将复制一份给克隆对象，

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/深克隆示意图.png)

可以通过引用类型实现`Cloneable`接口 或者 **先序列化再反序列化**这两种方式实现深拷贝    

```java
public class DeepPrototype implements Serializable, Cloneable {

    private static final long serialVersionUID = -6552383604366032330L;
    public String name;
    public Sheep sheep;

    public DeepPrototype() {
        super();
    }

    /**
     * 重写克隆方法返回自己的DeepPrototype对象
     */
    @Override
    protected Object clone() throws CloneNotSupportedException {
        Object deep = null;
        deep = super.clone();
        DeepPrototype deepPrototype = (DeepPrototype) deep;
        deepPrototype.sheep = (Sheep) sheep.clone();
        return deep;
    }

    //通过对象的序列化实现深拷贝
    public Object deepClone() throws IOException {
        //创建流对象
        ByteArrayOutputStream bos = null;
        ObjectOutputStream oos = null;
        ByteArrayInputStream bis = null;
        ObjectInputStream ois = null;
        try {
            //序列化
            bos = new ByteArrayOutputStream();
            oos = new ObjectOutputStream(bos);
            oos.writeObject(this);

            //反序列化
            bis = new ByteArrayInputStream(bos.toByteArray());
            ois = new ObjectInputStream(bis);
            DeepPrototype o = (DeepPrototype) ois.readObject();
            return o;

        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
            return null;
        } finally {
            bos.close();
            oos.close();
            bis.close();
            ois.close();
        }
    }
}
```

# 建造者模式 🔵 
将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示  

建造者模式是一种对象创建型模式，它将客户端与包含多个不见的复杂对象的创建过程分离，客户端无需知道复杂对象的内部组成部分与装配方式，只需要知道所需建造者的类型即可。建造者模式关注如何一步一步的创建一个复杂对象，不同的建造者定义了不同的创建过程。

## 结构
- `Builder`抽象建造者，为创建一个产品的各个部件指定抽象接口，一般声明两类方法，一种是`buildPartX()`构建部件，另一种是返回整体的对象
- `ConcreteBuilder`具体建造者，实现了`Builder`接口
- `Product`产品，被构建的复杂对象，包含多个组成部件，通过具体建造者进行构建
- `Director`指挥者，负责安排复杂对象的建造次序即`buildPartX()`的调用顺序

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/建造者模式示意图.png)

## 构建实现

1. 构建一个复杂对象类，包含多个属性(部件)

```java
public class Product {
    private String partA;
    private String partB;
    private String partC;
    //省略set get
    ...
}
```
2. 抽象建造者类，定义建造者模板，定义抽象构建方法和返回对象方法 

```java
public abstract class Builder {
    //产品对象
    protected Product product = new Product();
    //构建部分
    public abstract void buildPartA();

    public abstract void buildPartB();

    public abstract void buildPartC();
    
    public Product getProduct() {
        return product;
    }

}
```
3. 具体建造者类，真正负责建造功能的具体实现，实际应用中可以有多个

```java
public class ConcreteBuilder extends Builder {

    @Override
    public void buildPartA() {
        System.out.println("构建A");
        product.setPartA("AA");
    }

    @Override
    public void buildPartB() {
        System.out.println("构建B");
        product.setPartB("BB");
    }

    @Override
    public void buildPartC() {
        System.out.println("构建C");
        product.setPartC("CC");
    }
}
```
4. 指挥者类Director ，控制产品对象的创建过程，决定调用buildpart次序,接收不同的builder来对其进行构建  

```java
public class Director {
    private Builder builder;
    //构造器传入
    public Director(Builder builder) {
        this.builder = builder;
    }
    //构造方法传入
    public void setBuilder(Builder builder) {
        this.builder = builder;
    }

    public Product construct() {
        builder.buildPartA();
        builder.buildPartB();
        builder.buildPartC();
        return builder.getProduct();
    }
}
```
5. 实际调用，无需关心具体建造者的类型，无需关心产品对象的具体组装过程,更换`ConcreteBuilder`无需改变代码

```java
    ...
    ConcreteBuilder concreteBuilder = new ConcreteBuilder();
    Director director = new Director(concreteBuilder);
    director.construct();

    ConcreteBuilder2 concreteBuilder2 = new ConcreteBuilder2();
    Director director2 = new Director(concreteBuilder2);
    director2.construct();
    ...
```

## 优化讨论 
> 指挥者类Director是建造者模式的重要组成部分，负责指导如何构建产品，控制调用先后次序，下面是几种Director的变种形式   

### 省略Director 
为了简化系统，可以   
1. 将`Director` 和抽象建造者Builder进行合并
2. 将`construct`方法中的参数去掉，直接使用this调用  

```java
public abstract class Builder2 {
    //产品对象
    protected Product product = new Product();
    //构建部分
    public abstract void buildPartA();
    public abstract void buildPartB();
    public abstract void buildPartC();
    public Product getProduct() {
        return product;
    }
    //第一种简化
    public static Product construct(Builder2 builder) {
        builder.buildPartA();
        builder.buildPartB();
        builder.buildPartC();
        return builder.getProduct();
    }
    //第二种简化
    public Product construct2() {
        this.buildPartA();
        this.buildPartB();
        this.buildPartC();
        return this.getProduct();
    }

}
```

```java
    ...
    //第一种调用
    ConcreteBuilder2 concreteBuilder2 = new ConcreteBuilder2();
    ConcreteBuilder2.construct(concreteBuilder2);
    ...

    //第二种调用
    ConcreteBuilder2 concreteBuilder2 = new ConcreteBuilder2();
    concreteBuilder2.construct2();

```

### 钩子方法控制
通过钩子方法控制是否构建某个PartX    
1. 抽象建造者构建钩子方法 

```java
public abstract class Builder3 {
    //产品对象
    protected Product product = new Product();
    //构建部分
    public abstract void buildPartA();

    public abstract void buildPartB();

    public abstract void buildPartC();

    public Product getProduct() {
        return product;
    }
    //控制是否构建 part A
    public boolean isBuildA(){
        return false;
    }
}
```
2. 具体建造者 根据需求实现方法`isBuildA`,重写方法并返回true就能在Director中判断构建A

```java
public class ConcreteBuilder3 extends Builder3 {

    @Override
    public void buildPartA() {
    }
    @Override
    public void buildPartB() {
    }
    @Override
    public void buildPartC() {
    }

    //重写这个方法就能构建A 不重写就不能构建A
    @Override
    public boolean isBuildA() {
        return true;
    }
}
```

3. Director ，在调用director时 传入不同的`ConcreteBuilder` 就能实现A的构建控制

```java
public class Director2 {
    private Builder3 builder;
    //构造器传入
    public Director2(Builder3 builder) {
        this.builder = builder;
    }
    //构造方法传入
    public void setBuilder(Builder3 builder) {
        this.builder = builder;
    }

    public Product construct() {
        //实际控制
        if (builder.isBuildA()){
            builder.buildPartA();
        }
        builder.buildPartB();
        builder.buildPartC();
        return builder.getProduct();
    }
}

```

## 适用场景  

- 使用建造者模式可以避免重叠构造函数的出现 
- 使用代码创建不同形式的产品
- 使用生成器构造组合树或其他复杂对象 

# 结构型模式🔴

结构型模式关注如何将现有类或对象组织在一起形成更强大的结构。不同的机构型模式从不同的角度来组合类或对象   

结构型模式可以描述两种不同的东西，**类与类的实例(对象)**，根据这一点，结构型模式可以分为`类结构型模式`和`对象结构型模式`  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/结构型模式一览.png)

# 适配器模式🔴
适配器模式`Adapter Pattern`：将一个类的接口转换成客户希望的另一个接口，适配器模式让那些**接口不兼容的类**可以一起工作，别名包装器`Wrapper`模式

可以类比为电源转接头、手机充电器...  

## 结构
适配器模式包括**类适配器**和**对象适配器**，在对象适配器模式中，适配器和适配者之间的关系是关联关系；在类适配器模式中，适配器与适配者之间是继承(或实现)的关系     
- `Target` 目标抽象类，定义客户所需接口，可以是抽象类、接口、具体类，`类适配器`中，由于java不支持多继承，所以它只能是接口
- `Adapter` 适配器类，可以调用另一个接口，作为一个转换器，对`Adaptee`和`Target`进行适配，通过实现`Target`接口并继承`Adaptee`类来使二者产生联系  
- `Adaptee`适配者类(被适配)，定义了一个已经存在的接口，这个接口需要被适配   


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/类适配器模式结构图.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/对象适配器模式结构图.png)

## 类适配器
以手机充电接头为例  
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/适配器模式手机案例图.png)

```java
public class Voltage220V {
    public int output220V(){
        int src = 220;
        System.out.println("输出"+src+"V");
        return src;
    }
}

public interface Voltage5V {
    int output5V();
}

public class VoltageAdapter extends Voltage220V implements Voltage5V{
    @Override
    public int output5V() {
        //获取220V
        int i = output220V();
        //电压转换
        int dst = i/44;
        return dst;
    }
}

public class Phone {
    public void charging(Voltage5V i){
        if (i.output5V()==5){
            System.out.println("可以充电");
        }else{
            System.out.println("电压不合格，无法充电");
        }

    }

    public static void main(String[] args) {
        //调用实例
        Phone phone = new Phone();
        phone.charging(new VoltageAdapter());
    }
}
```

## 对象适配器 
对象适配器和类适配器思路基本相同，但是由于类适配器中存在**继承关系**   
根据`合成复用原则` 尽量使用`合成/聚合`关系来替代继承关系，来进行修改   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/对象适配器模式手机案例图.png)

```java
public class VoltageAdapter1  implements Voltage5V{
    private Voltage220V voltage220V;

    public VoltageAdapter1(Voltage220V voltage220V) {
        this.voltage220V = voltage220V;
    }

    @Override
    public int output5V() {
        int dst = 0;
        if(voltage220V!=null){
            //获取220V
            int i = voltage220V.output220V();
            //电压转换
            dst = i/44;
        }

        return dst;
    }
}

```

调用修改为：  

```java
...
Phone phone = new Phone();
phone.charging(new VoltageAdapter1(new Voltage220V()));
...

```

## 缺省适配器(单接口适配器)模式
当不需要实现一个接口提供的所有方法时，可以先设计一个抽象类实现该接口，并为接口中的每个方法提供一个默认实现（空方法）,那么该抽象类的子类可以选择性的覆盖父类的某些方法来实现需求     
- `ServiceInterface`适配者接口
- `AbstractServiceClass`缺省适配器类 它是缺省适配器模式的核心类，使用空方法的形式实现了 `ServiceInterface`中声明的方法  
- `ConcreteServiceClass`具体业务类，缺省适配器类的子类，根据需要选择覆盖父类中的方法   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/缺省适配器模式结构图.png)

## 优缺点和细节

### 优点
- 将目标类和适配者类解耦，通过引入一个适配器类来重用现有的适配者类，无需修改原有结构
- 增加了类的透明性和复用性
- 灵活性和扩展性都很好

## 适配器模式在SpringMVC中的应用
P64


# 桥接模式🔴
将抽象部分与它的实现部分解耦，使得两者都能够独立变化。  

> 在桥接模式中将两个独立变化的维度设计为两个独立的继承等级结构，而不是将二者耦合在一起形成多层继承结构，桥接模式在抽象层建立起一个抽象关联，该关联关系类似一条连接两个独立继承结构的敲，故名桥接模式  

?> 桥接模式是一种对象结构型模式，又被称为柄体模式或接口模式   
桥接模式用来处理多继承问题，用抽象关系取代了传统的多层继承，并**将类之间的静态继承关系转换为动态的对象组合关系**，使得系统更加灵活并易于扩展，有效控制了系统中**类的个数**

> 简单类比：需要用笔画出彩色，原来用蜡笔画出彩色，需要不同颜色的蜡笔(颜色和笔耦合),改用毛笔+彩色墨水(解耦)同样也能画出彩色

## 类比
对不同造型、不同品牌的手机进行打电话操作，传统思路如下

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/桥接模式手机案例1.png)

可以看到类很多，多重继承关系，如果新增一个品牌或者造型的手机，需要新增多个类    
使用桥接模式优化后得到    

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/桥接模式手机案例2.png)

```java
public interface Brand {
    void open();
    void call();
    void close();
}
public class Huawei implements Brand {

    @Override
    public void open() {
        System.out.println("华为开");
    }

    @Override
    public void call() {
        System.out.println("华为打");
    }

    @Override
    public void close() {
        System.out.println("华为关");
    }
}
public class XiaoMi implements Brand {

    @Override
    public void open() {
        System.out.println("小米开");
    }

    @Override
    public void call() {
        System.out.println("小米打");
    }

    @Override
    public void close() {
        System.out.println("小米关");
    }
}
//桥
public abstract class Phone {
    private Brand brand;

    public Phone(Brand brand) {
        this.brand = brand;
    }
    void open(){
        this.brand.open();
    }
    void call(){
        this.brand.call();
    }
    void close(){
        this.brand.close();
    }
}

public class FoldedPhone extends Phone {
    public FoldedPhone(Brand brand) {
        super(brand);
    }

    public void open() {
        super.open();
        System.out.println("folded open");
    }

    public void call() {
        super.call();
        System.out.println("folded call");
    }

    public void close() {
        super.close();
        System.out.println("folded close");
    }
}

//调用
public class Client {
    public static void main(String[] args) {
        Phone phone = new FoldedPhone(new Huawei());
        phone.open();
        phone.call();

    }
}
```

## 实际应用

### JDBC中的桥接模式
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jdbc的桥接模式.png)


## 优点缺点
优点：  
- 分离抽象接口和其实现部分
- 桥接模式可以取代多层继承方式
- 提高了系统扩展性 

缺点：  
- 增加了系统设计和理解难度


# 装饰模式🔴
装饰模式： 动态地给一个对象增加一些额外的职责(功能)，就扩展功能而言，装饰模式提供了一种比**使用子类**更加灵活的替代方案    

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/装饰着模式示意图.png)

- `Component`**抽象构件类** 它是具体构建和抽象装饰类的**共同父类**  
- `ConcreateComponent`**具体构件**，它是抽象构件类的子类，用于定义具体的构件对象，实现了在抽象构件中的声明放发，装饰类可以给他增加额外的方法   
- `Decorator`**抽象装饰类**，他也是`抽象构建类`的子类，用于给具体构建增加职责(方法)，但是具体方法在其子类中实现;同时维护一个指向`抽象构件`对象的引用  
- `ConcreteDecorator`**具体装饰类**，它是抽象装饰类的子类，负责向构件添加新的职责

## 示例  
以咖啡举例，咖啡分意式浓缩、美式 ，可以加配料牛奶，巧克力，糖... 
根据上面的示例，咖啡定义为`抽象构建类`，某种咖啡定义为`具体构件` ，配料定义为`抽象装饰类`，某种配料定义为`具体抽象类`，为了方便理解我们将咖啡和配料组合理解为`饮品`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/装饰着模式实例.png)

```java
//调用实例
public class CoffeeBar {
    public static void main(String[] args) {
        //2份糖 +牛奶+longblack
        Drink order = new LongBlack();
        order = new Milk(order);
        order = new Sugar(order);
        order = new Sugar(order);
        System.out.println(order.cost());
        System.out.println(order.getDes());
    }
}
```

Milk包含了LongBlack   
一份sugar包含了(Milk+LongBlack)  
一份sugar包含了(Chocolate+Milk+LongBlack)   
这样不管是什么形式的单品咖啡+调料组合，通过递归方式可以方便的组合和维护    

**最终关系如下图**    

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/装饰着模式实例关系.png)   

即使我们新增配料`Sor`和咖啡`ShortBlack`也很容易  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/装饰着模式实例类图.png)  


## 优缺点  

### 优点 
- 装饰模式比继承更加灵活，不会导致类的个数急剧增加
- 通过动态扩展对象功能，通过配置文件可以在运行时选择不同的具体装饰类，实现不同行为
- 可以对一个对象进行多次装饰，通过不同装饰的排列组合得到不同的的对象组合
- 可以根据需求增加具体构建类 和具体装饰类，原有代码无需改变

### 缺点 
- 使用装饰模式过程中会产生很多小对象，这些对象内部的具体装饰类套接方式不同
- 比继承灵活机动，但是对于多次装饰的对象排查问题比较困难   


# 组合模式🔴
**组合模式** `Composite Pattern` :组合多个对象形成树结构以表示具有部分-整体关系的层次接哦古，组合模式让客户端可以统一对待单个对象和组合对象    

> 组合模式又称 部分-整体 模式，属于对象结构型模式，他将对象组织到树结构中，可以用来描述整体与部分的关系   

类比Windows系统中的文件夹就是一种组合模式    
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/组合模式Windows示例.png ':size=70%')

## 结构  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/组合模式结构图.png)

- `Component`抽象构件:可以是接口或抽象类，为叶子结构和容器构件对象声明接口，在该角色中可以包含所有**子类共有行为的声明和实现**。
  - 在其中定义了访问及管理它的子构件的方法，如增加、删除、获取子构件等
- `Leaf`叶子构件：在组合结构中白哦是叶子节点对象，叶子节点没有子节点，实现了在抽象构件中定义的行为  
- `Composite`容器构件(非叶子节点) 容器节点对象，包含子节点(叶子节点或容器节点)

> 组合模式适用： 要处理的对象可以生成一颗树结构，而我们在对树上的节点和叶子进行操作时，它能够提供一致的方式，而不用考虑是节点还是叶子   

## 实例  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/组合模式学校案例.png)  


```java
//抽象类 抽象构件
public abstract class OrganizationComponent {
    private String name;
    private String des;

    protected void add(OrganizationComponent component){
        throw new UnsupportedOperationException();
    }
    protected void remove(OrganizationComponent component){
        throw new UnsupportedOperationException();
    }
    protected abstract void print();

    public OrganizationComponent(String name, String des) {
        this.name = name;
        this.des = des;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDes() {
        return des;
    }

    public void setDes(String des) {
        this.des = des;
    }
}
//大学 容器构件
public class University extends OrganizationComponent{

    List<OrganizationComponent> organizationComponents = new ArrayList<>();

    public University(String name, String des) {
        super(name, des);
    }

    @Override
    protected void add(OrganizationComponent component) {
        organizationComponents.add(component);
    }

    @Override
    protected void remove(OrganizationComponent component) {
        organizationComponents.remove(component);
    }

    @Override
    protected void print() {
        System.out.println("-----"+getName()+"-----");
        for (OrganizationComponent component : organizationComponents) {
            component.print();
        }
    }

    @Override
    public String getName() {
        return super.getName();
    }

    @Override
    public String getDes() {
        return super.getDes();
    }
}
//学院 容器构件
public class College extends OrganizationComponent{

    List<OrganizationComponent> organizationComponents = new ArrayList<>();

    public College(String name, String des) {
        super(name, des);
    }

    @Override
    protected void add(OrganizationComponent component) {
        organizationComponents.add(component);
    }

    @Override
    protected void remove(OrganizationComponent component) {
        organizationComponents.remove(component);
    }

    @Override
    protected void print() {
        System.out.println("-----"+getName()+"-----");
        for (OrganizationComponent component : organizationComponents) {
            component.print();
        }
    }

    @Override
    public String getName() {
        return super.getName();
    }

    @Override
    public String getDes() {
        return super.getDes();
    }
}
//专业 叶子构件
public class Department extends OrganizationComponent{
    public Department(String name, String des) {
        super(name, des);
    }

    @Override
    protected void print() {
        System.out.println(getName());
    }

    @Override
    public String getName() {
        return super.getName();
    }

    @Override
    public String getDes() {
        return super.getDes();
    }
}

//调用
public class Client {
    public static void main(String[] args) {
        OrganizationComponent university = new University("清华", "顶级大学");

        OrganizationComponent college1 = new College("计算机学院", "描述1");
        OrganizationComponent college2 = new College("建筑学院", "描述2");

        college1.add(new Department("软件工程","miao1"));
        college1.add(new Department("网络工程","miao2"));
        college1.add(new Department("计算机科学","miao3"));

        college2.add(new Department("土木科学","miao1"));
        college2.add(new Department("地质构造","miao2"));

        university.add(college1);
        university.add(college2);

        university.print();

    }
}
-----清华-----
-----计算机学院-----
软件工程
网络工程
计算机科学
-----建筑学院-----
土木科学
地质构造
```

## 注意事项和细节 
### 优点
- 可以清楚的定义分层次的复杂对象，表示对象的全部或部分层次，忽略层次差异，方便对整个层次结构进行控制
- 可以一致的使用一个组合结构或其中单个对象
- 新增容器构建或叶子构件都很容易，无需对现有类库进行任何修改，符合开闭原则

### 适用环境 
- 在具有整体和部分层次结构中，希望通过一种方式忽略整体与部分的差异，客户端可以一致的对待他们
- 在一个使用面向对象语言的系统中需要处理一个**树形结构**
- 在一个系统中能分离出叶子对象和容器对象，而且他们的类型不固定，需要增加一些新类型  


# 外观模式🔴  
外观模式`Facade Pattern` 为子系统中的一组接口提供一个统一的入口，外观模式定义了一个高层接口，这个接口使得这一子系统更加容易使用   

> 在外观模式中，一个子系统的外部与其内部的通信通过一个统一的外观类进行，弯管类将客户类与子系统的内部复杂性分隔开，使得客户类只需要与外观角色打交道，而不需要与子系统内部的很多对象打交道   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/外观模式示意图.png)

## 结构
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/外观模式结构图.png)

- `Client`客户端  
- `Facade`外观角色：在客户端调用它的方法，它将所有从客户端发来的请求委派到响应的子系统，传递给相应的子系统对象处理   
- `SubSystem`子系统角色，在软件系统中可以有一个或多个，处理外观角色的请求，外观角色对子系统而言只是一个客户端   

## 实现 

```java
public class SubSystemA{
    public void methodA(){
        //业务
    }
}
public class SubSystemB{
    public void methodB(){
        //业务
    }
}
public class SubSystemC{
    public void methodC(){
        //业务
    }
}
//外观类
public class Facade{
    private SubSystemA obj1 = new SubSystemA();
    private SubSystemB obj2 = new SubSystemB();
    private SubSystemC obj3 = new SubSystemC();

    public void method(){
        obj1.methodA();
        obj1.methodB();
        obj1.methodC();
    }

}

//实际调用
public class Test{
    public static void main(String args[]){
        Facade facade = new Facade();
        facade.method();
    }
}
```

## 适用环境和优缺点
### 适用环境
BS系统中的导航页面就可以看做一个外观角色，CS系统的菜单或者工具栏也可以看作外观角色    
- 为一系列子系统提供一个简单入口
- 客户端程序与多个子系统之间存在很大依赖，引入外观类降低子系统和客户端的耦合
- 层次化结构中可以使用外观模式定义系统中每一层的入口，层与层之间不直接产生联系，而通过外观类建立联系，降低耦合

### 优点 
- 对客户端屏蔽了子系统组件，减少了客户端所处理的对象数目
- 实现了子系统和客户端之间的松耦合关系，子系统的变化不会影响到客户端，只需调整外观类

### 缺点
- 不能很好的限制客户端直接使用子系统类
- 若设计不当，增加新的子系统可能需要修改外观类的源码，违背开闭原则

# 享元模式🔴  
若一个软件系统正在运行时所创建的**相同或者相似的对象数量太多**，导致运行代价过高，带来系统资源的浪费、性能下降等问题。为了避免系统中出现大量重复对象，就需要使用**享元模式**通过共享技术实现相同或相似对象的复用。    
在享元模式中存储这些共享实例对象的地方称为`享元池`，java中的`String常量池`、`数据库连接池`等都是享元模式的实现方式      

## 定义  
享元模式(Flyweight Patern)：运用共享技术有效的支持大量细粒度对象的复用  

> 享元模式要求能够被共享的对象必须是细粒度对象，又称为轻量模式或蝇量模式，享元模式能做到共享的关键是区分了内部状态`Intrinsic State`和外部状态`Extrinsic State`   
> **内部状态**是存储在享元对象内部并且不会随环境改变而改变状态，内部状态可以共享。例如：字符内容不会随外部环境的变化而变化，无论任何环境下字符`a`始终是`a`，不会变成`b`     
> **外部状态**是随环境改变而改变的，不可以共享的状态。享元对象的外部状态通常由客户端保存，并在享元对象被创建之后需要使用的时候再传入到享元对象内部。一个外部状态与另一个外部状态之间是相互独立的。    


## 结构

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/享元模式原理图.png)

- `Flyweight`抽象享元类，抽象享元类通常是一个接口或者抽象类，同时定义出对象的内部状态和外部状态
- `ConcreteFlyweight`具体享元类,实现了抽象享元类，为内部状态提供存储空间，通常结合单例模式来设计具体享元类
- `UnsharedConcreteFlyweight`非共享具体享元类，不能共享的享元类，可以通过此类创建不可共享的具体享元类
- `FlyweightFactory`享元工厂类，用于创建并管理享元对象，针对抽象享元类编程，将各类型的具体享元对象存储在一个享元池中，同时提供从池中获取对象的方法  


## 实现  

```java
//抽象享元类
public abstract class Flyweight{
    public abstract void operation(String extrinsicState);
}

//具体享元类
public class ConcreteFlyweight extends Flyweight{
    //内部状态intrinsicState 作为成员变量，同一个享元对象的内部状态是一致的
    private String intrinsicState;


    public ConcreteFlyweight(String intrinsicState){
        this.intrinsicState = intrinsicState;
    }

    //外部状态 extrinsicState 在使用时由外部设置，不保存在享元对象中，即使是同一个对象

    public void operation(String extrinsicState){

    }
}

// 非共享具体享元类
public class UnsharedConcreteFlyweight extends Flyweight{
    public void operation(String extrinsicState){

    }
}

//享元工厂类
public class FlyweightFactory{
    //共享享元池
    private HashMap flyweights = new HashMap();

    public Flyweight getFlyweight(String key){
        if(flyweight.containsKey(key)){
            return (Flyweight)flyweight.get(key);
        }else{
            //若对象不存在，先创建一个新的对象添加到享元池中
            Flyweight fw = new ConcreteFlyweight();
            flyweight.put(key,fw);
            return fw;
        }
    }

}

```

## 实际应用
java中的String常量池和Integer常量池  

## 适用环境和优缺点

### 优点 
- 可以减少系统中重复对象的数量，提高性能
- 外部状态的独立，保证了享元对象可以在不同环境中被共享

### 缺点
- 使得程序负载话，需要分离享元对象的内部状态和外部状态 


### 适用环境
- 系统中存在大量相同或相似的对象，造成内存浪费
- 对象大部分状态都可以外部化，可以将这些外部状态传入对象中
- 维护享元池也需要资源，所以要在多次重复使用对象时再使用享元模式  


# 代理模式🔴
代理模式`Proxy Pattern`：给某一个对象提供一个代理，并由代理对象控制对原对象的引用；通过代理对象访问目标对象。  

> 代理模式是一种对象结构型模式，代理对象在客户端对象和目标对象之间起到中介的作用，可以去掉客户不能访问的内容或者增加额外的内容（对原有对象功能进行扩展）   

代理模式主要分三种形式，静态代理，动态代理(JDK代理，接口代理)，Cglib代理

## 静态代理
静态代理在使用时，需要定义接口或父类，被代理对象(即目标对象)与代理对象一起实现相同的接口或者继承相同的父类  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/静态代理模式结构图.png)


```java
//抽象父类或接口
public abstract class Subject{
    public abstract void request();
} 


//真实对象
public class RealSubject extends Subject{
    public void request(){
        //业务逻辑
        System.out.println("真实方法调用");
    }
}

//代理对象类
public Proxy extends Subject{
    //真实对象的引用
    //也可以通过构造器传入进行构造
    private RealSubject realSubject = new RealSubject();

    public void preRequest(){
        //前置方法
    }

    public void request(){
        preRequest();
        //调用真实对象的方法
        realSubject.request();
        postRequest();
    }

     public void postRequest(){
        //后置方法
    }

}

//测试
public class Client{
    public static void main(String[] args){
        Proxy pr = new Proxy();
        pr.request();

    }
}

```

> **优点：** 在不修改目标对象的功能前提下，能通过代理对象对目标功能进行扩展和限制    
> **缺点：** 可能会有多个代理类，一旦接口或父类的方法增加，所有的目标类和代理类都需要改变   

## 动态代理
由于**静态代理**会产生很多静态类，所以我们要想办法可以通过一个代理类完成全部的代理功能，这就是动态代理   

- 代理对象不需要实现接口，但是目标对象要实现接口，否则不能用动态代理
- 代理对象的生成，是利用JDK的API(反射)，**动态的在内存中构建代理对象**
- 动态代理也叫做；JDK代理，接口代理   

在Java中想要实现动态代理机制，需要`java.lang.reflect.InvocationHandler`接口和`java.lang.reflect.Proxy`类的支持

```java
// 接口或抽象类
public interface ITeacherDao {
    void teach();
}

//被代理对象
public class TeacherDao implements ITeacherDao {
    @Override
    public void teach() {
        System.out.println("教学任务");
    }
}

//通过反射的动态代理实现
public class ProxyFactory {
    private Object target;

    public ProxyFactory(Object target) {
        this.target = target;
    }
    // Proxy.newProxyInstance
    //  public static Object newProxyInstance(ClassLoader loader, 目标对象使用的类加载器
    //                                    Class<?>[] interfaces, 目标对象实现的接口类型
    //                                      InvocationHandler h) 事件处理方法
    /**
        被代理对象target通过参数传递进来，
        通过target.getClass().getClassLoader()获取ClassLoader对象，
        然后通过target.getClass().getInterfaces()获取它实现的所有接口，
        再将target包装到实现了InvocationHandler接口的对象中。
        通过newProxyInstance函数我们就获得了一个动态代理对象。
    
     */
    public Object getProxyInstance() {
        return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                Object invoke = method.invoke(target, args);

                return invoke;
            }
        });
    }
}
```

## Cglib代理  
静态代理和JDK代理模式都要求`目标对象实现一个接口`，但是有时候目标对象只是一个单独的对象，并没有实现任何接口，这个时候可以使用目标对象子类来实现代理，这就是**Cglib代理**   

Cglib代理也叫做子类代理，它是在内存中构建一个子类对象从而实现对目标对象功能扩展；Cglib是一个强大的高性能的代码生成包，它可以在运行期间扩展java类与实现java接口,底层通过字节码处理框架`ASM`来转换字节码并生成新的类   

在AOP编程中若:  
- 目标对象需要实现接口，使用`JDK`代理
- 目标对象不需要实现接口，使用`Cglib`代理  

### 构建Cglib
- 引入jar包或依赖
- 代理的类不能为final类，否则会报错
- 目标方法不能为final 或static，否则无法生效，类加载机制

1. 引入jar包或依赖  

```xml
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.3.0</version>
</dependency>
```

2. 目标对象，无需实现接口或继承

```java
public class TeacherDao{
    public void teach(){
        System.out.println("进入教学");
    }
}
```

3. `MethodInterceptor`类似于JDK动态代理的`InvocationHandler`，在方法调用前后进行增强  

```java
public class ProxyFactory implements MethodInterceptor {

    private Object target;

    public ProxyFactory(Object target){
        this.target = target;
    }

//重写 intercept 方法，实现对被代理对象（目标对象）方法调用   
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("cglib 代理开始，可以添加逻辑");
        Object obj = method.invoke(target,objects);
        System.out.println("cglib 代理结束");
        return obj;
    }

    //通过这个方法给目标对象创建一个代理对象
    public Object getProxyInstance(){
        //工具类，类似于JDK动态代理的Proxy类
        Enhancer enhancer = new Enhancer();
        //设置父类
        enhancer.setSuperclass(target.getClass());
        //设置回调函数
        enhancer.setCallback(this);
        //创建子类对象，即代理对象
        return enhancer.create();
    }
}
```

4. 客户端调用

```java
public class Client {

    public static void main(String[] args) {

        //目标对象
        TeacherDao target = new TeacherDao();
        //获取代理对象,并且将目标对象传递给代理对象
        TeacherDao teacher = (TeacherDao) new ProxyFactory(target).getProxyInstance();
        teacher.teach();
    }
}
```

## 应用场景
- **延迟初始化**，可能会使用的重量级服务对象
- **访问控制**，特定客户端才能访问服务对象
- **本地执行远程服务**，本地执行网络客户端的请求，负责处理网络相关复杂细节
- **记录日志**

# 行为型模式🟩  
行为型模式关注系统中对象之间的交互，研究系统在运行时对象之间的相互通信与写作，进一步明确对象的职责。行为型模式不仅仅关注类和对象本身，还重点关注他们之间的相互作用和职责划分。  

行为型模式珠岙分为`类行为模式`和`对象行为型模式`   
- **类行为型模式**使用继承关系在几个类中分配行为，通过多态来分配父类和子类的职责
- **对象行为型模式**使用对象的关联关系来分配行为，通过对象关联分配职责

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/行为型模式一览1.png)   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/行为型模式一览1.png)


# 模板方法模式🟩  
模板方法`Template Method Pattern`：超类中定义一个算法框架，而将一些步骤延迟到子类中，模板方法使得子类不改变一个算法的结构即可重定义该算法的某些步骤   

> 模板方法是一种阶段那的行为型设计模式，在其结构中**只存在父类和子类之间的继承关系**。在抽象父类中提供一个称为模板方法的方法来定义这些基本方法的执行次序，而通过其子类来覆盖某些步骤，从而使得**相同的算法框架可以有不同的执行结果**。  

## 结构 
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/模板方法模式结构图.png)

- `AbstractClass` 抽象类，定义一系列基本操作，和一个模板方法来规定算法骨架(方法调用顺序等)
- `ConcreteClass` 具体子类，抽象类的子类，实现在父类中声明的抽象方法来完成算法中子类的特定操作  




## 实现  
使用一个例子来说明，咖啡制作，咖啡都是由`萃取浓缩咖啡->加水->加配料->加辅料->倒入杯中`的流程制作,配料不同的咖啡有不同名称，加奶就是拿铁`Latte`，加水就是美式`LongBlack`，加奶泡就是卡布奇诺`Cappuccino`,同时可以选择添加辅料：糖，冰... ;通过定义模板方法中的调用顺序，来实现算法流程，**模板方法要用final修饰，防止子类重写**  

把抽象类规定为咖啡，下属子类为拿铁，美式...等，制作一杯咖啡有固定步骤萃取，加水，倒出(基本操作，抽象类实现)，不同的咖啡配料不同(抽象方法子类实现)，客户也可以自定义加糖、冰(钩子方法)...

```java
//抽象类
public abstract class Coffee {

    void addEspresso() {
        System.out.println("萃取咖啡");
    }
    void addWater() {
        System.out.println("加水冲泡");
    }
    void addIce() {
        System.out.println("加冰");
    }
    void addSugar() {
        System.out.println("加糖");
    }
    void pourInCup() {
        System.out.println("倒入杯子");
    }
    //是否选择加糖 默认不加
    boolean choseAddSugar() {
        return false;
    }

    //选择加冰
    boolean choseAddIce() {
        return false;
    }
    //抽象方法，需要子类进行重写
    abstract void addType();

    public final void makingDrinks() {
        //基本浓缩咖啡
        addEspresso();
        //冲泡
        addWater();
        //加料(自定义配料)
        addType();
        
        //选择辅料
        //钩子方法
        if(choseAddSugar()){
            addSugar();
        }
        if (choseAddIce()){
            addIce();
        }

        //倒进杯子
        pourInCup();
    }
}

//美式咖啡
public class LongBlack extends Coffee {
    @Override
    void addType() {
        System.out.println("美式咖啡再加水");
    }
    //重写方法 进行加糖加冰
    @Override
    boolean choseAddSugar() {
        return true;
    }

    @Override
    boolean choseAddIce() {
        return true;
    }
}

//拿铁
public class Latte extends Coffee {
    @Override
    void addType() {
        System.out.println("拿铁加奶");
    }

    @Override
    boolean choseAddSugar() {
        return true;
    }
}

//卡布奇诺
public class Cappuccino extends Coffee {
    @Override
    void addType() {
        System.out.println("卡布奇诺加入奶泡");
    }
}
```

```java
public class Client {
    public static void main(String[] args) {

        LongBlack latte = new LongBlack();
        latte.makingDrinks();  
        //输出 萃取咖啡 加水冲泡 美式咖啡再加水 加糖 加冰 倒入杯子

        Latte latte1 = new Latte();
        latte1.makingDrinks();
        //萃取咖啡 加水冲泡 拿铁加奶 加糖 倒入杯子
    }
}

```

## Spring 中的模板方法
IOC 容器初始化时的模板方法，不管是 XML 还是注解的方式，对于核心容器启动流程都是一致的

`AbstractApplicationContext` 的 `refresh` 方法实现了 IOC 容器启动的主要逻辑。

一个 `refresh()` 方法包含了好多其他步骤方法，像不像我们说的 **模板方法**，`getBeanFactory()` 、`refreshBeanFactory()` 是子类必须实现的抽象方法，`postProcessBeanFactory()` 是钩子方法。

```java
public abstract class AbstractApplicationContext extends DefaultResourceLoader
      implements ConfigurableApplicationContext {
	@Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			prepareRefresh();
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
			prepareBeanFactory(beanFactory);
            postProcessBeanFactory(beanFactory);
            invokeBeanFactoryPostProcessors(beanFactory);
            registerBeanPostProcessors(beanFactory);
            initMessageSource();
            initApplicationEventMulticaster();
            onRefresh();
            registerListeners();
            finishBeanFactoryInitialization(beanFactory);
            finishRefresh();
		}
	}
    // 两个抽象方法
    @Override
	public abstract ConfigurableListableBeanFactory getBeanFactory() throws 		IllegalStateException;	
    
    protected abstract void refreshBeanFactory() throws BeansException, IllegalStateException;
    
    //钩子方法
    protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
	}
 }
```

打开你的 IDEA，我们会发现常用的 `ClassPathXmlApplicationContext` 和 `AnnotationConfigApplicationContext` 启动入口，都是它的实现类（子类的子类的子类的...）。

`AbstractApplicationContext` 的一个子类 `AbstractRefreshableWebApplicationContext` 中有钩子方法 `onRefresh() ` 的实现：

```java
public abstract class AbstractRefreshableWebApplicationContext extends …… {
    /**
	 * Initialize the theme capability.
	 */
	@Override
	protected void onRefresh() {
		this.themeSource = UiApplicationContextUtils.initThemeSource(this);
	}
}
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/模板方法模式Springioc结构图.png)

## 模板方法注意事项和细节

- 算法只存在于父类中，容易进行修改，同时由子类实现具体细节，方便扩展
- 模板方法通过钩子可以实现反向控制
- 更换和增加新的子类非常方便，符合单一职责和开闭原则



# 命令模式🟩  
命令模式`Command Pattern` 将一个请求封装为一个对象，从而可用不同的请求对客户进行参数化， 对请求排队或者记录请求日志，以及支持**可撤销**的操作     
命令模式是一种对象行为型模式，也称作动作模式或事务模式

## 结构和实现 
命令模式的核心在于引入了抽象命令类和具体命令类，**通过命令类来降低发送者和接收者的耦合度**，请求发送者秩序指定一个命令对象，再通过命令对象来调用请求接收者的处理方法   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/命令模式结构图.png)

- `Command`抽象命令类，一般是一个抽象类或接口，通过其中的方法调用请求接收者的相关操作  
- `ConcreteCommand`具体命令类，抽象命令类的子类，实现了抽象命令类中声明的方法
- `Invoker`调用者，调用者即请求发送者，通过命令对象来执行请求
- `Reveiver`接收者，接收者执行与请求相关的操作，具体实现对请求的业务处理 

> 通俗理解就是：将军发布命令，士兵去执行命令，`Invoker`就是将军，`Reveiver`就是士兵，`Command` 和 `ConcreteCommand`就是命令类，将军负责发布命令，具体命令由谁执行，由命令类重写的方法决定，譬如是由工兵还是侦察兵来执行，需要命令类来指定     

```java
//命令接口
public interface Command {
    //执行
    void execute();
    //撤销
    void undo();
}

//一个命令单独实现Command 实际可能有多个
public class ConcreteCommand implements Command {

    private Receiver receiver;

    public ConcreteCommand(Receiver receiver) {
        this.receiver = receiver;
    }

    @Override
    public void execute() {
        receiver.action();
    }

    @Override
    public void undo() {
        receiver.undo();
    }
}

//执行者 真正的执行动作
public class Receiver {
    public void action(){
        System.out.println("执行动作");
    }
    public void undo(){
        System.out.println("执行取消动作");
    }
}

public class Invoker {
    private Command command;

    //构造注入
    public Invoker(Command command) {
        this.command = command;
    }

    //设值注入
    public void setCommand(Command command) {
        this.command = command;
    }
    //调用命令的方法
    public void call(){
        command.execute();
    }
    public void unCall(){
        command.undo();
    }
}

//测试
public class Client {
    public static void main(String[] args) {
        Command concreteCommand = new ConcreteCommand(new Receiver());
        Invoker invoker = new Invoker(concreteCommand);
        invoker.call();//执行动作
        invoker.unCall();//执行取消动作
    }
}
```

命令模式中可以存在多个`ConcreteCommand`和`Receiver`


## 优缺点和细节

### 优点 
- 降低系统的耦合度
- 提高系统扩展性，新的命令可以很容易的加入到系统中
- 可以比较容易的设计一个命令队列或宏命令
- 为请求的撤销和恢复操作提供了一种设计和实现方案 


### 缺点 
命令模式可能会导致系统中有过多的具体命令类  

### 适用环境
- 系统需要将请求调用者和请求接收者解耦，使得调用者和接收者不直接交互
- 系统需要在不同的时间指定请求、将请求排队和执行请求 
- 系统需要支持命令的撤销和恢复操作  


# 访问者模式🟩 
访问者模式主要包含访问者和被访问者，被访问的元素具有不同的类型，而且不同的访问者可以对其施加不同的访问操作。访问者模式可以使得用户在不修改现有系统的情况下扩展系统功能，为不同类型的元素添加新的操作    

## 结构 
以医生开具药方为例，`医生`开具`药方`，`会计`根据`药方`缴费，`药房`根据`药方`拿药,药方在这里看作多种药品的集合，可能需要提供针对药的多种处理方式  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/访问者模式结构.png ':size=80%')  

- `Visitor`抽象访问者，为对象结构中每一个具体元素声明一个访问操作，具体访问者需要实现这些操作方法，定义具体访问操作
- `ConcreteVisitor`具体访问者，实现了每个由抽象访问者声明的操作，每个操作用于访问对象结构中一种类型的元素  
- `Element`抽象元素，一般是一个抽象类或接口，声明一个`accept()`方法，用于接受访问者的访问操作，方法通常以一个抽象访问者作为参数
- `ConcreteElement`具体元素，实现了`accept()`方法，在方法中调用访问者的访问方法以便完成对一个元素的操作   
- `ObjectStructure`对象结构，元素的集合，用于存放元素对象，提供遍历其内部元素(集合)的方法

案例代码：  
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/访问者模式示例.png)

1. 药品超类   
```java
public abstract class Med {
    public abstract void accept(Visitor v);
}
```

2.中药和西药类   
```java
public class MedZhong extends Med {
    @Override
    public void accept(Visitor visitor) {
        visitor.getZhong(this);
    }
}

public class MedXi extends Med {
    @Override
    public void accept(Visitor visitor) {
        visitor.getXi(this);
    }
}
```

3. 访问者超类(医院员工类)     
```java
public abstract class Visitor {
    public abstract void getZhong(MedZhong z);
    public abstract void getXi(MedXi x);
}
```

4. 医生类和会计类     
```java
public class Doctor extends Visitor {
    @Override
    public void getZhong(MedZhong z) {
        System.out.println("医生获取-中药");
    }

    @Override
    public void getXi(MedXi x) {
        System.out.println("医生获取-西药");
    }
}
public class Kuaiji extends Visitor {
    @Override
    public void getZhong(MedZhong z) {
        System.out.println("会计获取-中药");
    }

    @Override
    public void getXi(MedXi x) {
        System.out.println("会计获取-西药");
    }
}
``` 

5. 药方类，多种药的集合     
```java
public class MedListObj {
    private List<Med> meds = new LinkedList<>();

    public void add(Med m) {
        meds.add(m);

    }

    public void remove(Med p) {
        meds.remove(p);
    }

    public void display(Visitor v) {
        for (Med m : meds) {
            m.accept(v);
        }
    }
}
```

6. 测试类

```java
public class Client {
    public static void main(String[] args) {
        MedListObj medListObj = new MedListObj();
        medListObj.add(new MedXi());
        medListObj.add(new MedZhong());
        medListObj.display(new Doctor()); //医生获取-西药 医生获取-中药 医生获取-西药
    }
}
```

## 注意事项和细节  

### 优点 
- 访问者模式符合单一职责原则，让程序有扩展性和灵活性，增加新的访问操作很方便  
- 访问者模式适用于数据结构相对稳定的系统  

### 缺点 
- 在访问者模式中增加新的元素类ConcreteElement很困难，每增加一个新元素都需要在抽象访问者种添加新的抽象方法  
- 访问者模式破坏了对象的封装性。

### 适用环境  
一个对象结构包含多个类型的对象，希望对这些对象实施一些依赖其具体类型的操作。    
对象结构中对象对应的类很少改变，但经常需要在此对象结构上定义新的操作   

# 迭代器模式🟩 
软件开发中存在大量的可以存储多个成员对象的聚合类，对应的对象称为聚合对象，为了更加方便的操作聚合对象，引入迭代器无需了解聚合对象的内部结构即可实现对聚合对象中成员的遍历  

**迭代器模式：**提供一种方法顺序访问一个聚合对象中的各个元素，而又不用暴露该对象的内部表示(内部结构)   

## 结构 

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/迭代器模式示意图.png)

- `Iterator`抽象迭代器 定义了访问和遍历元素的接口，声明了用于遍历数据元素的方法,一般用Java中的`java.util.Iterator`   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/Iterator类图.png)

- `ConcreteIterator`具体迭代器，实现了抽象迭代器接口，完成对聚合对象的遍历，同时在其中通过游标来记录在聚合对象中所处的当前位置
- `Aggregate`抽象聚合类，用于存储和管理元素对象，声明一个`createIterator()`方法用于创建一个迭代器对象，充当抽象迭代器工厂角色
- `ConcreteAggregate`具体聚合类，抽象聚合类的子类，实现了`createIterator()`方法，返回一个具体聚合类对应的具体迭代器`ConcreteIterator`实例


## 实现
案例定义 学校的学院和部门的关系，一个学院中有多个部门，遍历学校学院和其内的部门    
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/Iterator案例图.png)

1. 抽象聚合类 和具体聚合类    

```java
public interface College {
    String getName();
    void addDepartment(String name,String desc);
    Iterator createIterator();
}

public class ComputerCollege implements College {
    //数组方式
    Department[] departments;
    int numOfDepartment = 0;

    public ComputerCollege() {
        departments = new Department[3];
        addDepartment("Java","java desc");
        addDepartment("PHP","PHP desc");
        addDepartment("Python","python desc");
    }

    @Override
    public String getName() {
        return "计算机学院";
    }

    @Override
    public void addDepartment(String name, String desc) {
        Department department = new Department(name, desc);
        departments[numOfDepartment] = department;
        numOfDepartment++;

    }

    @Override
    public Iterator createIterator() {
        return new ComputerCollegeIterator(departments);
    }
}

public class InfoCollege implements College {
    List<Department> departmentList;

    public InfoCollege() {
        departmentList = new ArrayList<>();

        addDepartment("信息专业1","专业描述1");
        addDepartment("信息专业2","专业描述2");
        addDepartment("信息专业3","专业描述3");
    }

    @Override
    public String getName() {
        return "信息工程学院";
    }

    @Override
    public void addDepartment(String name, String desc) {
        Department department = new Department(name, desc);
        departmentList.add(department);

    }

    @Override
    public Iterator createIterator() {
        return new InfoCollegeIterator(departmentList);
    }
}
```

2. 具体迭代器     

```java
//学院迭代器
public class ComputerCollegeIterator implements Iterator {
    //数组方式
    Department[] departments;
    //遍历的位置
    int position = 0;

    public ComputerCollegeIterator(Department[] departments) {
        this.departments = departments;
    }

    @Override
    public boolean hasNext() {
        if (position >= departments.length || departments[position] == null) {
            return false;
        } else {
            return true;
        }
    }

    @Override
    public Object next() {
        Department department = departments[position];
        position += 1;

        return department;
    }

    @Override
    public void remove() {

    }
}

public class InfoCollegeIterator implements Iterator {
    List<Department> departmentList;
    //索引
    int index = -1;

    public InfoCollegeIterator(List<Department> departmentList) {
        this.departmentList = departmentList;
    }

    @Override
    public boolean hasNext() {
        if (index >= departmentList.size() - 1) {
            return false;
        } else {
            index += 1;
            return true;
        }
    }

    @Override
    public Object next() {
        //此处index 是在调用 hasNext基础上的 index已经+1了
        return departmentList.get(index);
    }

    @Override
    public void remove() {

    }
}
```

3. 基本部门类和测试方法

```java
//部门
public class Department {
    private String name;
    private String desc;

    public Department(String name, String desc) {
        this.name = name;
        this.desc = desc;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDesc() {
        return desc;
    }

    public void setDesc(String desc) {
        this.desc = desc;
    }
}

//打印
public class OutputImpl {
    //学院集合
    List<College> collegeList;

    public OutputImpl(List<College> collegeList) {
        this.collegeList = collegeList;
    }
    //遍历所有学院
    public void printCollege(){
        Iterator<College> iterator = collegeList.iterator();
        while (iterator.hasNext()){
            College next = iterator.next();
            System.out.println("==="+next.getName()+"===");
            printDepartment(next.createIterator());
        }
    }

    public  void printDepartment(Iterator iterator){
        while (iterator.hasNext()){
            Department next = (Department) iterator.next();
            System.out.println(next.getName());
        }
    }

}
//测试类
public class Client {
    public static void main(String[] args) {
        ArrayList<College> colleges = new ArrayList<>();
        colleges.add(new ComputerCollege());
        colleges.add(new InfoCollege());
        OutputImpl output = new OutputImpl(colleges);
        output.printCollege();
    }
}
```

## Java中的迭代器
Java的集合框架`List` `Set` 等聚合类都继承或实现了`java.util.Collection`接口，`Collection`接口除了基本操作方法外，还提供了一个`iterator()`方法，用于返回一个Iterator类型的迭代器对象  

Java中的`Collection`接口和`Iterator`接口充当了迭代器模式的抽象层，分别对应于**抽象聚合类**和抽**象迭代器**

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/java中的迭代器模式示例图.png)


## 使用细节和优缺点
### 优点
- 迭代器模式支持以不同方式遍历一个聚合对象，在同一个聚合对象上可以定义多种遍历方式 
- 迭代器模式简化了聚合类 
- 将迭代器分开，就是把管理对象集合和遍历对象集合的责任分开  

### 缺点
- 每个聚合类都需要一个迭代器，增加新的聚合类时需要对应增加新的迭代器类  
- 设计难度较大，需要充分考虑系统扩展

### 适用环境 
- 访问一个聚合对象的内容而无需暴露它的内部表示，迭代器和聚合对象分离
- 需要为一个聚合对象提供多种遍历方式
- 为遍历不同聚合结构提供一个统一的接口，方便操作


# 观察者模式🟩 
观察者模式`Observer Patern`定义对象之间的一种一对多的依赖关系，使得每当一个对象状态发生改变时其相关依赖对象皆得到通知并被自动更新   

> 观察者模式是适用频率较高的模式之一，观察者模式中发生改变的对象称为观察目标，被通知的对象称为观察者，一个观察目标可以对应多个观察者，而且这些观察者之间可以没有任何互相联系，可以根据需要增加和删除观察者，使得系统易于扩展

?> 观察者模式又称为 `发布-订阅`模式、`模型-视图`模式、`源-监听器`模式、`从属者`模式  

## 结构  
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/观察者模式结构图.png)

- `Subject`目标，被观察对象，在其中定义一个观察者集合，提供方法来增加和删除观察者对象，同时定义通知方法`notify()`
- `ConcreteSubject`具体目标，目标类的子类，状态发生改变时将向各个观察者发出通知
- `Observer`抽象观察者，接口声明了更新数据方法`update()` 
- `Concreteobserver`具体观察者

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/观察者模式示例图.png)

```java
public abstract class Subject {
    //定义一个观察者集合用于存储所有观察者对象
    protected List<Observer> observers = new ArrayList<>();

    //添加观察者
    public void attach(Observer observer){
        observers.add(observer);
    }
    //移除观察者
    public void detach(Observer observer){
        observers.remove(observer);
    }
    //通知方法，子类重写
    public abstract void notifyX();
}

public class ConcreteSubject extends Subject {
    @Override
    public void notifyX() {
        //遍历所有观察者 执行其响应方法
        for (Observer observer : observers) {
            observer.update();
        }
    }
}

public interface Observer {
    void update();
}


public class ConcreteObserver implements Observer {
    @Override
    public void update() {
        //具体观察者的响应方法
    }
   
}

//调用示例
public static void main(String[] args) {
        ConcreteObserver concreteObserver = new ConcreteObserver();
        ConcreteSubject subject = new ConcreteSubject();
        subject.attach(concreteObserver);
        subject.notifyX();
}
```

## 应用  

### JDK中的观察者模式
`java.util.Observable`类充当了观察目标类，在其中定义了一个`Vector` 来存储观察者对象`Observer`   
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/java中的观察者模式.png)

 Java9后被弃用  


### Spring中的观察者模式
1. **事件：ApplicationEvent** 是所有事件对象的父类。ApplicationEvent 继承自 jdk 的 EventObject，所有的事件都需要继承 ApplicationEvent，并且通过 source 得到事件源。

   Spring 也为我们提供了很多内置事件，`ContextRefreshedEvent`、`ContextStartedEvent`、`ContextStoppedEvent`、`ContextClosedEvent`、`RequestHandledEvent`。

2. **事件监听：ApplicationListener**，也就是观察者，继承自 jdk 的 EventListener，该类中只有一个方法 onApplicationEvent。当监听的事件发生后该方法会被执行。

3. **事件源：ApplicationContext**，`ApplicationContext` 是 Spring 中的核心容器，在事件监听中 ApplicationContext 可以作为事件的发布者，也就是事件源。因为 ApplicationContext 继承自 ApplicationEventPublisher。在 `ApplicationEventPublisher` 中定义了事件发布的方法：`publishEvent(Object event)`

4. **事件管理：ApplicationEventMulticaster**，用于事件监听器的注册和事件的广播。监听器的注册就是通过它来实现的，它的作用是把 Applicationcontext 发布的 Event 广播给它的监听器列表。


## 优缺点和适用环境
### 优点
- 观察目标和观察者之间建立了一个抽象耦合，观察目标只许维持一个抽象观察者集合，无需了解具体观察者
- 支持广播通信，观察目标会向所有已经注册的观察者对象发送通知、
- 新增观察者无需修改源代码，符合开闭原则  

### 缺点
- 若一个观察目标对象有很多直接观察者，通知所有人花费代价很大
- 观察者和观察目标之间有依赖循环，就会导致循环调用

### 适用环境 
- 链式触发机制
- 仅仅监听改变而不关心改变的内容和对象   


# 中介者模式
一个系统中对象之间的联系呈现为网状结构，对象之间存在大量的多对多关系，导致系统非常的负载，这些对象即会影响其他对象，也会被其他对象影响，这些对象称为同事对象，他们之间通过彼此相互作用实现系统的行为，这将导致一个过度耦合的系统     
中介者模式可以使对象之间的关系数量急剧减少，通过引入中介者对象可以将系统的网状结构变为以中介者为中心的星型结构，所有对象都通过中介者发生相互作用

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/中介者模式的网状结构和星型结构.png)

?> 例如：多个微信好友之间进行一对一聊天和群聊(中介者就是群)的关系

> 如果在一个系统中对象之间存在多对多的相互关系，可以将对象之间的一些交互行为从各个对象中分离出来，集中封装在一个中介者对象中，并由该中介者进行统一协调   

中介者模式`Mediator Pattern`:定义一个对象来封装一系列对象的交互，中介者模式使各对象之间不需要显式地相互调用，从而使其耦合松散，而且用户可以独立地改变它们之间的交互   

## 结构

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/中介者模式的结构图.png)

- `Mediator`抽象中介者，定义一个接口或抽象类，用于同事类之间的通信
- `Concrete Mediator`具体中介者，抽象终结者的子类，通过协调各个同事对象来实现协作行为，维持了对各个同事对象的引用 
- `Colleague`抽象同事类，定义各个同事类公有的方法，并声明一些抽象方法供子类实现，同时维持了一个对抽象中介者类的引用，子类通过该引用和中介者通信
- `Concrete Colleague`具体同事类，抽象同事类的子类，同事类之间通过中介者完成通信  

## 应用

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/中介者模式案例图.png)

抽象中介者类/具体中介者类    
```java
public abstract class Mediator {
    protected List<Colleague> colleagues = new ArrayList<>();
    //将同事对象加入列表方便相互调用
    public void register(Colleague colleague) {
        colleagues.add(colleague);
    }

    public abstract void operation(Colleague colleague, int i);

}

public class ConcreteMediator extends Mediator {

    @Override
    public void operation(Colleague colleague,int i) {
        //根据不同的值调用另一个同事类的自定义方法
        //实现同事类之间的通信
        if (i==1){
            colleagues.get(1).method1();
        }else{
            colleagues.get(0).method1();
        }
    }
}
```

抽象同事类和2个具体同事类  

```java
public abstract class Colleague {
    //抽象中介者的引用
    protected Mediator mediator;
    //构造方法
    public Colleague(Mediator mediator) {
        this.mediator = mediator;
    }
    //声明自定义方法
    public abstract void method1();
    //与中介者的通信
    public void method2(int i){
        mediator.operation(this,i);
    }
}

public class ConcreteColleague extends Colleague {

    public ConcreteColleague(Mediator mediator) {
        super(mediator);
    }

    @Override
    public void method1() {
        System.out.println("同事类1的自定义方法");
    }
}

public class ConcreteColleague2 extends Colleague {

    public ConcreteColleague2(Mediator mediator) {
        super(mediator);
    }

    @Override
    public void method1() {
        System.out.println("同事类2的自定义方法");
    }
}
```

客户端  

```java
public class Client {
    public static void main(String[] args) {
        Mediator concreteMediator = new ConcreteMediator();

        Colleague concreteColleague = new ConcreteColleague(concreteMediator);
        Colleague concreteColleague2 = new ConcreteColleague2(concreteMediator);
        concreteMediator.register(concreteColleague);
        concreteMediator.register(concreteColleague2);

        concreteColleague.method1();//同事类1的自定义方法
        concreteColleague.method2(1);//同事类2的自定义方法

        concreteColleague2.method1();//同事类2的自定义方法
        concreteColleague2.method2(0);//同事类1的自定义方法
    }
}
```

## 优缺点

**优点**：中介者模式简化了对象交互，解耦同事类对象，减少子类生成

**缺点**：中介者角色包含大量业务逻辑，交互细节，难以维护

# 备忘录模式🟩 
备忘录模式是软件系统中的“月光宝盒”它提供了一种对象状体的撤销实现机制，当系统中的某个对象需要恢复到某一历史状态时可以使用备忘录模式进行设计

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/备忘录模式示意图.png)


**备忘录模式**`Memento Pattern`在不破坏封装的前提下捕获一个对象的内部状态，并在该对象之外保存这个状态，这样可以在以后将对象恢复到原先保存的状态   

?> 备忘录模式是一种对象行为型模式，别名为 标记`Token`模式

## 结构

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/备忘录模式结构图.png)

- `Originator`原发器。通过创建一个备忘录来存储当前内部状态，也可以使用备忘录来恢复其内部状态
- `Memento` 备忘录，用于存储原发器的内部状态，根据原发器来决定保存那些内部状态
- `Caretaker`负责人，负责保存/提取备忘录，但是不能对备忘录的内容进行操作或检查  

```java
public class Originator {
    private String state;

    public String getState() {
        return state;
    }

    public void setState(String state) {
        this.state = state;
    }

    //编写一个方法可以保存状态对象，
    public Memento save(){
        return new Memento(state);
    }

    public void getStateFromMemento(Memento memento){
        state = memento.getState();
    }
}

public class Memento {
    //保存的state
    private String state;

    public Memento(String state) {
        this.state = state;
    }

    public String getState() {
        return state;
    }
}

public class Caretaker {
    private List<Memento> list = new ArrayList<>();

    public void add(Memento memento) {
        list.add(memento);
    }

    public Memento get(int index) {
        return list.get(index);
    }
}

//测试类
public class Client {
    public static void main(String[] args) {
        Originator originator = new Originator();
        Caretaker caretaker = new Caretaker();
        originator.setState("状态1");
        caretaker.add(originator.save());

        originator.setState("状态2");
        caretaker.add(originator.save());

        originator.getStateFromMemento(caretaker.get(0));
        System.out.println(originator.getState());
        originator.getStateFromMemento(caretaker.get(1));
        System.out.println(originator.getState());

    }
}
```

## 优缺点和注意
提供了一种状态恢复机制，可以恢复到特定的状态，但是资源消耗过大，成员变量对象太多  

适用于保存一个对象在某一时刻的全部状态或部分状态，防止外界对象破环一个对象的历史状态   

# 解释器模式🟩 
解释器模式用于描述和构成一个简单的语言解释器，主要应用于使用面向对象语言开发的解释器的设计。当需要开发一个新的语言时可以考虑使用解释器模式  

**解释器模式**`Interpreter Pattern`:给定一个语言，定义它的文法的一种表示，并定义一个解释器，这个解释器使用该表示来解释语言中的句子

## 结构 
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/解释器模式结构图.png)

- `AbstractExpression`抽象表达式。声明了抽象的解释操作，是所有`终结符表达式`和`非终结符表达式`的公共父类
- `TerminalExpression`终结符表达式,实现了文法中的终结符相关联的解释操作,在句子中的每一个终结符都是该类的一个实例
- `NonterminalExpression`非终结符表达式,实现了文法中非终结符的解释操作.解释操作一般通过递归方式来实现
- `Contex`环境类,用于存储解释器之外的一些全局信息,通常临时存储了要解释的语句
- `Client`客户类,在客户类中构造了表示该文法定义的语言中一个特定句子的抽象语法树,该抽象语法树由非终结符表达式和终结符表达式实例组合而言,在客户类中还将调用解释操作,实现对句子的解释


**解释器模式略过,有需要再详细理解**     
p133-135 略过

# 状态模式🟩 
状态模式用于解决系统中复杂队形的状态转换以及不同专改下行为的封装问题,当系统中某个对象存在多个状态,这些状态之间可以进行转换,而且对象在不同状态下行为不相同时可以使用状态模式    

**状态模式**`State Pattern`:允许一个对象在其内部状态改变时改变它的行为,对象开起来似乎修改了它的类 

## 结构 

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/状态模式结构图.png)

- `Context`环境类,又称为上下文类,拥有多种状态的对象
- `State`抽象状态类，定义一个接口封装与`Context`的一个特定状态相关的行为，在抽象状态类中声明了各种不同状态的应对方法  
- `ConcreteState`具体状态类，抽象状态类的子类，每个子类实现一个与`Context`的状态相关的行为，每个具体状态类对应环境的一个具体状态  

## 实现

```java
public class Context {
    private State state;

    public Context(){
        state = null;
    }

    public void setState(State state){
        this.state = state;
    }

    public State getState(){
        return state;
    }
}


public interface State {
    public void doAction(Context context);
}

//开始状态
public class StartState implements State {

    public void doAction(Context context) {
        System.out.println("Player is in start state");
        context.setState(this);
    }

    public String toString(){
        return "Start State";
    }
}

//结束状态
public class StopState implements State {
 
   public void doAction(Context context) {
      System.out.println("Player is in stop state");
      context.setState(this); 
   }
 
   public String toString(){
      return "Stop State";
   }
}

//测试类
public class Test {
    public static void main(String[] args) {
        Context context = new Context();

        StartState startState = new StartState();
        startState.doAction(context);//Player is in start state
        System.out.println(context.getState().toString());//Start State

        StopState stopState = new StopState();//Player is in stop state
        stopState.doAction(context);

        System.out.println(context.getState().toString());
    }
}
```


## 优缺点和适用环境

### 优点
- 状态模式封装了状态的转换规则，在状态模式中可以将状态的转换代码封装在环境类或者是具体状态类中，可以对状态转换代码进行集中管理，而不是分散在一个个业务类中  
- 状态模式将所有对于某个状态有关的行为放到一个类中，只需诸如一个不同的状态对象即可使环境对象拥有不同的行为
- 状态模式允许状态转换逻辑与状态对象合成一体，而不是提供一个巨大的条件语句块 
- 状态模式可以让多个环境对象共享一个状态对象，从而减少系统中对象的个数

### 缺点
- 状态模式会增加系统中类和对象的个数，导致系统运行开销增大
- 状态模式的结构与实现都较为复杂，如果适用不当将增加系统设计难度 
- 状态模式对开闭原则的支持并不太好，增加新的状态类需要修改那些负责状态转换的源代码，修改某个状态类的行为，也需要修改对应类的源代码

### 适用环境
- 对象的行为依赖于它的状态，状态的改变将导致行为的变化
- 代码总包含大量的与对象状态有关的条件语句，这些条件语句会导致代码的可维护性和灵活性变差，不能方便的增加和删除状态


# 策略模式🟩 
策略模式用于算法的自由切换和扩展，策略模式对应于解决某一问题的一个算法族，允许用户从该算法族中任选一个算法解决某一问题，同可以方便的更换算法或者增加新的算法。

例如：我们选择出行方式，飞机快但是贵，火车慢但是便宜，这时就需要根据实际情况灵活的选择出行方式，就可以使用策略模式  

**策略模式**`Strategy Pattern`:定义一系列算法，将每个算法封装起来，并让它们可以相互替换。策略模式让算法可以独立于使用它的客户而变化   


## 结构 
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/策略模式结构图.png)  

- `Context`环境类，环境类是使用算法的角色，在解决某个问题时可以采用多种策略，类中维持一个抽象策略类的引用实例
- `Strategy`抽象策略类，为所支持的算法声明了抽象方法，是所有策略类的父类，环境类通过抽象方法调用具体策略类中实现的算法
- `ConcreteStrtegy`具体策略类，实现了具体策略类中声明的算法

## 实现

```java
public abstract class AbstractStrategy {
    public abstract void algorithm(); //声明抽象算法
}
```

```java
//具体算法A
public class ConcreteStrategyA extends AbstractStrategy {
    @Override
    public void algorithm() {
        System.out.println("ConcreteStrategyA");
    }
}

//具体算法B
public class ConcreteStrategyB extends AbstractStrategy {
    @Override
    public void algorithm() {
        System.out.println("ConcreteStrategyB");
    }
}
```

```java
public class Context {
    //维持一个对抽象策略类的引用
    private AbstractStrategy strategy;

    public void setStrategy(AbstractStrategy strategy) {
        this.strategy = strategy;
    }

    //调用策略类中的算法
    public void algorithm(){
        strategy.algorithm();
    }

}
```

```java
//示例调用
public class Client {
    public static void main(String[] args) {
        Context context = new Context();
        AbstractStrategy strategyA = new ConcreteStrategyA();
        AbstractStrategy strategyB = new ConcreteStrategyB();

        context.setStrategy(strategyA);
        context.algorithm();// System.out.println("ConcreteStrategyA");

        context.setStrategy(strategyB);
        context.algorithm();// System.out.println("ConcreteStrategyB");

    }
}
```

## 优缺点和注意事项 

### 适用场景
- 使用对象中各种不同的算法变体，并希望能在运行时切换算法，可以使用策略模式
- 有许多仅在执行某些行为时略有不同的相似类时，可以使用策略模式
- 类中使用了负载条件运算符以在统一算法的不同变体中切换时

### 优点
- 可以在运行时切换对象内的算法
- 可以将算法的实现和使用算法的代码隔离开来
- 使用组合来替代继承
- 开闭原则，无需对上下文进行修改就能够引入新的策略

### 缺点 
- 针对算法极少发生改变的情况，没有必要引入策略模式
- 客户端必须知道所有的策略类，并自行巨顶使用哪一个策略类
- 策略模式将造成系统产生很多具体策略类，任何细小变化都会导致系统要增加新的策略类
- 无法同时在客户端使用多个策略类
- 通过匿名函数等方式可以避免创建策略类而实现不同算法 

# *责任链模式🟩 
大学中的奖学金审批流程，学生提交申请后，分别要通过辅导员、系主任、院长、校长的层层审批，他们构成一个处理申请表的链式结构，申请表沿着这条链进行传递，这条链就成为职责链（责任链）

**职责链模式**：避免将一个请求的发送者与接收者耦合在一起，让多个对象都有机会处理请求，将接受请求的对象连接成一条链，并且沿着这条链传递请求，直到有一个对象能够处理它为止   

## 结构
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/职责链模式结构图.png)  

- `Handler`抽象处理者，定义一个处理请求的接口，一般为抽象类，由于不同的具体处理者处理请求的方式不同，因此在其中定义了抽象请求处理方法  
- `ConcreteHandler`具体处理者，抽象处理者的子类，可以处理用户请求，在具体处理者类中实现了抽象处理者中定义的抽象处理方法，处理前进行判断，可以处理就处理，否则转发给后继者   

?> 职责链模式通常每个接收者都包含对另一个接收者的引用，如果一个对象不能处理该请求，就会把这个请求转给下一个接收者   


## 实现  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/责任链模式实例图.png)

抽象处理者 

```java
public abstract class Approver {
    Approver approver;//后继者
    String name;//名称

    public Approver(String name) {
        this.name = name;
    }

    //设置下一个处理者
    public void setApprover(Approver approver) {
        this.approver = approver;
    }

    //处理审批请求的方法 交由子类完成
    public  abstract void processRequest(PurchaseRequest request);
}
```

三个具体处理者，系主任，院长，校长

```java
public class DepartApprover extends Approver {

    public DepartApprover(String name) {
        super(name);
    }

    @Override
    public void processRequest(PurchaseRequest request) {
        if (request.getPrice()<=5000){
            System.out.println("请求id"+request.getId()+"被"+this.name+"处理");
        }else{
            //无法处理交给下一个
            approver.processRequest(request);
        }
    }
}

public class CollegeApprover extends Approver {

    public CollegeApprover(String name) {
        super(name);
    }

    @Override
    public void processRequest(PurchaseRequest request) {
        if (request.getPrice()>5000 && request.getPrice()<=10000){
            System.out.println("请求id"+request.getId()+"被"+this.name+"处理");
        }else{
            //无法处理交给下一个
            approver.processRequest(request);
        }
    }
}

public class SchoolApprover extends Approver {

    public SchoolApprover(String name) {
        super(name);
    }

    @Override
    public void processRequest(PurchaseRequest request) {
        if (request.getPrice()>10000 ){
            System.out.println("请求id"+request.getId()+"被"+this.name+"处理");
        }else{
            //无法处理交给下一个
            approver.processRequest(request);
        }
    }
}
```

奖学金类  

```java
public class PurchaseRequest {
    private int type = 0;
    private float price = 0.0f;
    private int id = 0;

    public PurchaseRequest(int type, float price, int id) {
        this.type = type;
        this.price = price;
        this.id = id;
    }

    public int getType() {
        return type;
    }

    public float getPrice() {
        return price;
    }

    public int getId() {
        return id;
    }
}
```

测试方法

```java
public class Test {
    public static void main(String[] args) {
        PurchaseRequest purchaseRequest = new PurchaseRequest(1, 20000, 1);
        Approver a1 = new DepartApprover("系主任");
        Approver a2 = new CollegeApprover("院长");
        a1.setApprover(a2);
        Approver a3 = new SchoolApprover("学校");
        a2.setApprover(a3);
        a3.setApprover(a1);//形成调用环

        a1.processRequest(purchaseRequest);

    }
}
```

## Spring MVC中的应用
JAVA 中的异常处理机制、JAVA WEB 中 Apache Tomcat 对 Encoding 的处理，Struts2 的拦截器，JSP、Servlet 的 Filter 均是责任链的典型应用

- Spring MVC请求流程中，执行了拦截器相关方法`interceptor.preHandler`
- 在处理Spring MVC请求时，使用到职责链模式和适配器模式
- `HandlerExecutionChain`主要负责请求拦截的执行和请求处理，但是他本身不处理请求，将请求分配给链上注册处理执行
- `HandlerExecutionChain`维护了`HandlerInterceptor`的集合，可以向其中注册相应的拦截器 


## 总结
**职责链就是灵活版的if...else...**,就是将这些判定语句放到了各个处理类中，这样做的优点是比较灵活，但是要注意设置处理类的前后关系时，一定要特别仔细，不要出现循环调用问题

### 优点
- 降低耦合度
- 简化对象的相互连接
- 增强给对象指派职责的灵活性
- 增加新的请求处理类很方便 

### 缺点
- 针对比较长的职责链，请求可能涉及到多个处理对象，系统性能会受到影响，链式调用顺序不当可能会导致死循环，一般需要设置最大节点数避免过长的职责链




# 参考资料
> - [尚硅谷视频](https://www.bilibili.com/video/BV1G4411c7N4)
> - [黑马视频](https://www.bilibili.com/video/BV1Np4y1z7BU)
> - [Javakepper](https://javakeeper.starfish.ink/)
> - [Java设计模式(刘伟)](https://book.douban.com/subject/30173863/)
> - [图说设计模式](https://design-patterns.readthedocs.io/zh_CN/latest/index.html)
> - [设计模式博客](https://refactoringguru.cn/design-patterns)