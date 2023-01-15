# 说明
💣 待完善学习细化  
⭐ 重点关注
# 基础

## JVM JDK JRE
JRE 是 Java 运行时环境它不能用于创建新程序
JDK 是 Java Development Kit 缩写，它是功能齐全的 Java SDK。它拥有 JRE 所拥有的一切，还有编译器（javac）和工具（如 javadoc 和 jdb）。它能够创建和编译程序。

## 字节码
JVM可以理解的代码叫做字节码`.class` 文件，通过字节码文件可以无需编译在不同系统运行java程序 
通过运行时编译 JIT编译器,完成编译后,会将其字节码对应的机器码保存,下次直接使用, **java是编译与解释共存的语言**

## 为什么java是编译与解释共存
java程序要先编译后解释,先编译生成.class文件,通过java解释器解释执行

## 字符型常量,字符串
字符常量 是单引号引起的一个字符,只占2字节

## 可变参数 
```java
public static void method(String... args){

}
```
- 可变参数只能作为函数最后一个参数
- 方法重载优先匹配固定参数方法

## break continue return 
- break 跳出当前整个循环体(只跳出一层,嵌套循环只跳出所在循环体,外层循环继续执行),直接跳出外层看下面代码
- continue 跳过当次循环,执行下次循环
- return 跳出方法

```java
public class LinshiTest {
    public static void main(String[] args) {
        out:
        for (int i = 0; i < 10; i++) {
            System.out.println("iii"+i);
            for (int j = 0; j < 5; j++) {
                if(j==3){
                    break out;
                }
                System.out.println("jjj"+j);
            }
        }
    }
}
```

## 方法

### 静态方法为什么不能调用非静态成员
- 静态方法属于类,类加载就会分配内存,可以通过类名直接访问,
- 非静态成员必须在对象实例化后才能访问
- 在类的非静态成员不存在的时候静态成员就已经存在了,此时调用非法

### 方法重载\重写

重载:
- 发生在同一个类中（或者父类和子类之间）
- 方法名必须相同,参数类型,个数,顺序,返回值,修饰符 可以不同
- Java 允许重载任何方法， 而不只是构造器方法

重写:
- 子类对父类的允许访问的方法的实现过程进行重新编写
- 方法名、参数列表必须相同，子类方法返回值类型应比父类方法返回值类型更小或相等，抛出的异常范围小于等于父类，访问修饰符范围大于等于父类
- 如果父类方法访问修饰符为 `private/final/static` 则子类就不能重写该方法，但是被 static 修饰的方法能够被再次声明
- 构造方法无法重写

> 重写就是子类对父类方法的重新改造,外部样子不能变,内部逻辑可以改变  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/guide1.png)  


## == equals() 
`==` 基本数据类型 比较值,引用数据类型比较内存地址
`equals()` 方法 是Object类方法,默认使用 `==` 判断,

## hashCode() equals()

> 两个相等的对象`hashCode()` 必须相等,如果equals方法判断为相等的对象,他们的hashcode值也要相等
> 如果重写equals方法没有重写hashcode方法,可能会导致equals方法判断相等,而hashcode却不相等

## 基本数据类型
- 四种基本整型 `byte` `short` `int` `long` 两种浮点 `float` `double` 一种字符型 `char` 一种布尔型 `boolean` 
- 注意他们对应的包装类型 如:`Character` 默认值为null 
- 基本数据类型占内存少 在java虚拟机栈 方法栈帧 中的局部变量表 

## 基本数据类型的二元运算参考
> `double` > `float` > `long` > `int`


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/gudie2.png)



## & && | ||
- `exp1 && exp2` 若 `exp1` false `exp2` 不会进行计算,`短路`运算  
- `exp1 || exp2` 若 `exp1` true `exp2` 不会进行计算,,`短路`运算  
- `&` 同为1时为1，否则为0,无论如何都会计算两边的表达式,非`短路`运算  
- `|` 同时0时为0,否则为1 ,无论如何都会计算两边的表达式,非`短路`运算  

## 右结合运算符
- `a+=b+=c` => `a += (b += c)` 
- `=` `+=` `-=`


## 数组创建
```java
// 数据类型[]  变量名 = new 数据类型[存储元素的个数]  
int[] x; //声明一个int[]类型的变量  
x=new int[100]; //和上面组成数组的定义  可作为匿名数组  
int[] arr = new int[4];  
arr[0] = 1;  
// 数组类型[] 变量名 = new 数组类型[]{元素1,元素2 ...}  
int[] arr = new int[]{ 1, 2, 3, 4, 5 };  
// 数组类型[] 变量名 = {元素1,元素2 ...}  
int[] arr = { 1, 2, 3, 4, 5 };  
```



 ## 包装类型的常量池 
> `Byte,Short,Integer,Long` 这 4 种包装类默认创建了数值 `[-128，127]` 的相应类型的缓存数据，
> `Character` 创建了数值在 `[0,127]` 范围的缓存数据，`Boolean` 直接返回 `True or False`

```java
Integer i1 = 33;
Integer i2 = 33;
System.out.println(i1 == i2);// 输出 true
```
## 自动装箱拆箱

```java
Integer i = 10;  //装箱
int n = i;   //拆箱
```
装箱其实就是调用了 包装类的`valueOf()`方法，拆箱其实就是调用了 `xxxValue()`方法


## 成员变量 局部变量
- 成员变量属于类,可以被 `public...` `static` 等修饰,可以被`final`修饰,属于类或者实例,随对象创建和消失,自动赋值默认值
- 局部变量属于方法,不能被控制修饰符及 `static` 修饰,可以被`final`修饰,属于方法,存在于栈内存,随方法调用而消失,不会自动赋值 


## 内部类
- 成员内部类
  - 内部类可以无限制的访问外围类的所有 成员属性和方法
  - 外围类访问内部类要通过内部类实例访问
  - 成员内部类不能存在任何static变量和方法
  - 成员内部类依附于外部类
- 局部内部类
  - 方法内部的类,不能用public或者private修饰
  - 属性只能在方法内部生效
- 匿名内部类(新建接口对象/类对象)
- 静态内部类
  - 创建不需要依赖外部类对象
  - 不能使用任何外部类的非static 成员变量和方法




## 静态域 静态常量 final 域
```java
//没有final静态域
private static int nextId = 1;
//静态常量
private static final int APD = 1;
//final 域
private final int xxid =123;
```

> final域要有初始值或者在构造方法中赋值,在构造对象后就不能修改值  

## 静态方法
- 不可访问对象状态,所需参数都通过显式提供
- 只能访问类的静态域  

## final 类和方法
- final类不能继承,类中方法默认为final方法
- 子类不能覆盖final方法
- 使用final方法主要目的就是确保它们不会再子类中改变语义


## 构造方法
构造方法主要作用是完成对类对象的初始化工作

构造方法特点:
- 名字与类名相同
- 没有返回值
- 生成类的对象自动执行无需调用
- 不能重写,可以重载,一个类中可以存在多个构造函数

## 深拷贝 浅拷贝
浅拷贝: 对象内部的引用对象会直接复制内存地址,拷贝后的对象内部的引用对象和拷贝前的是同一个对象  
深拷贝: 对象内部的引用对象重写clone 方法 ,并且在对象内部clone方法调用引用对象clone方法赋值,拷贝后对象内部的引用对象和拷贝前不是同一个对象  

实现`Cloneable` 接口 重写 `clone()` 方法

```java
@Override
public Person clone() {
    try {
        Person person = (Person) super.clone();
        person.setAddress(person.getAddress().clone());
        return person;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}

```

## String StringBuffer StringBuilder  

- `String` 使用final修饰 不可变,不可继承
- `StringBuffer` 和 `StringBuilder` 都继承自 `AbstractStringBuilder` 
- `StringBuffer` 线程安全 性能略低
- `StringBuilder` 线程不安全

### 字符拼接
> Java 语言本身并不支持运算符重载，“+”和“+=”是专门为 String 类重载过的运算符，也是 Java 中仅有的两个重载过的元素符  
> 对象引用和“+”的字符串拼接方式，实际上是通过 StringBuilder 调用 append() 方法实现的，拼接完成之后调用 toString() 得到一个 String 对象  


### 字符串常量池
> 字符串常量池 是 JVM 为了提升性能和减少内存消耗针对字符串（String 类）专门开辟的一块区域，主要目的是为了避免字符串的重复创建
> 1.7 之前在方法区  之后再堆中


## 抽象类
- 包含一个或多个抽象方法的类本身必须被声明为抽象
- 抽象类充当占位角色，他们的具体实现在子类中
- 抽象类也可以包含具体方法和属性
- 扩展抽象类
  - 子类实现部分抽象方法或者不实现抽象方法，子类必须为抽象类
  - 子类实现全部抽象方法，子类不为抽象类
- 抽象类不能被实例化

## 接口
- 接口不是类 
- 接口不能包含实例域或静态方法,但可以包含常量
- java8 后接口可以包含静态方法,但是静态方法只能通过接口直接调用,实现类无法调用
- java8 后可以为接口提供一个默认实现,必须用`default` 修饰这个方法,默认实现可以通过接口和实现类对象调用




## 泛型 
> JDK5 引入 包括泛型类、泛型接口、泛型方法  
> 一般情况下 E 表示集合元素, KV表示map类型,T U S 表示任意类型

```java
//泛型类
class Pair<R>{
    private R first;
}

class Pair<R,T>{
    private R first;
    private T second;
}

//泛型接口
public interface Generator<T> {
    public T method();
}

//泛型方法
public <E> void printarr(E[] ss ){
}

//泛型类型限定 类和方法都可以使用
class Pair<T extends Comparable>{
    private T second;
}

```

### 类型擦除
> 无论何时定义一个泛型类型,都自动提供了一个相应的原始类型,原始类型的名字就是删去类型参数后的泛型类型名.
> 擦除类型变量,并替换为限定类型,无限定类型的变量用 `Object`
> ` Pair < T >` 原始类型为 `Pair` 因为T是无限定的变量,所以直接用Object 替换 

有限定类型替换   
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/guide3.png)

```java
//类型擦除说明
public class ErasedTypeEquivalence {
    public static void main(String[] args) {
        Class c1 = new ArrayList<String>().getClass();
        Class c2 = new ArrayList<Integer>().getClass();
        System.out.println(c1 == c2); //true
    }
}
```

## 反射
```java
Class cl = Class.forName("com.ric.Test")
Class cl = Test.class
Test test = new Test();
Class cl = test.getClass();
Class cl = ClassLoader.loadClass("com.ric.Test")
//利用反射创建类,调用默认参数构造器创建对象
Test test = cl.NewInstance()

```

### 利用反射分析类的能力
- `getFields` `getDeclaredFields`
- `getMethods` `getDeclaredMethods`
- `getConstructors` `getDeclaredConstructors` 
- `setAccessible` 两种调用方法

## 异常

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/guide4.png)

- 受查异常`checkException` 绿色  Java 代码在编译过程中，如果受检查异常没有被 catch/throw 处理的话，就没办法通过编译 
- 非受查异常 `uncheckedException`  红色 `RuntimeException` 及其子类都统称为非受检查异常  

### try catch finally
> 不要在 finally 语句块中使用 return! 当 try 语句和 finally 语句中都有 return 语句时，try 语句块中的 return 语句会被忽略。这是因为 try 语句中的 return 返回值会先被暂存在一个本地变量中，当执行到 finally 语句中的 return 之后，这个本地变量的值就变为了 finally 语句中的 return 返回值

- `try-with-resources` 代替`try-catch-finally` 可以省略finally中关闭流的方法


## I/O

### 序列化
- 序列化 : 数据结构或对象转换成二进制字节流
- 反序列化: 

> 序列化的主要目的是通过网络传输对象或者说是将对象存储到文件系统、数据库、内存中

> 不想进行序列化可以使用 `transient` 关键字修饰
> 只能修饰变量,修饰的变量 在反序列化后变量值将会被置成类型的默认值.如:int 类型 反序列化后结果就是0(原值会丢失)


### IO流
- 按照流流向分,输入流和输出流
- 按照操作单元分,字节流和字符流

- `InputStream/Reader` 输入流的基类, `字节/字符`输入流
- `OutputStream/Writer` 输出流的基类,`字节/字符`输出流

### 既然有了字节流,为什么还要有字符流
> 不管是文件读写还是网络发送接收，信息的最小存储单元都是字节，那为什么 I/O 流操作要分为字节流操作和字符流操作呢?

> 答: 字符流是由java虚拟机将字节转换得到的,而这个过程**非常耗时,容易乱码**,**音视频等媒体文件**使用字节流比较好,字符使用字符流比较好


## 重点

### java只有值传递?
Java 中将实参传递给方法（或函数）的方式是 **值传递**:
- 如果参数是基本类型,传递的是基本类型的字面量值得拷贝,会创建副本
- 如果参数是引用类型, 传递得是实参所引用的对象在堆中地址值的拷贝

### ⭐💣IO模型
待  

### 浮点精度丢失问题
> 浮点数之间的等值判断，基本数据类型不能用==来比较，包装数据类型不能用 equals 来判断 
> 判断相等方法: 指定误差范围,两数相减的绝对值`Math.abs()` 小于误差范围 可以认为是相等的 

BigDecimal 常见方法
- `add`
- `subtract`
- `multiply`
- `divide` 
- `compareTo()`
- `setScale(3,RoundingMode.HALF_UP)` 保留三位小数四舍五入

> 禁止使用`new BigDecimal(double)` 构造 `BigDecimal` 对象,推荐使用`String构造`或者 `BigDecimal.valueOf()`




# ⭐ 容器  
![](https://gitee.com/riiicc/JavaCoreTech/raw/master/part9/photo/p914-1.png)

## 集合概览
 - `Collection` 接口
   - `List` `Set` `Queue` 
 - `Map` 接口

## List, Set, Queue, Map 四者的区别
- `List` 有序可重复
- `Set` 无序不可重复
- `Queue` 有序可重复,队列顺序
- `Map` 键值存储,无序 key 不可重复,value 可重复

## List
- `ArrayList` : `Object []`数组实现 查询快 线程不安全
- `Vector` : `Object []`数组实现,线程安全
- `LinkedList` : 双向链表 增删快 线程不安全

> LinedList 也有随机访问方法,只不过效率很低,没有实现 `RandomAccess` ,但是可以用get方法获取元素
> 判断是否支持随机访问 `list instanceof RandomAccess`

## Set
全部线程不安全
- `HashSet` 无序 唯一,基于hashMap 底层采用HashMap (哈希表)
- `LinkedHashSet` : 有序 唯一 HashSet的子类, 内部通过 LinkedHashMap实现, (链表,哈希表)
- `TreeSet` 有序唯一,红黑树 

### HashSet 查重
> 当你把对象加入HashSet时，HashSet 会先计算对象的hashcode值来判断对象加入的位置，
> 同时也会与其他加入的对象的 hashcode 值作比较，如果没有相符的 hashcode，HashSet 会假设对象没有重复出现。
> 但是如果发现有相同 hashcode 值的对象，这时会调用equals()方法来检查 hashcode 相等的对象是否真的相同。如果两者相同，HashSet 就不会让加入操作成功

## Queue
- `PriorityQueue` `Object []` 数组来实现二叉堆
- `ArrayQueue` `Object []`

## Map
- `HashMap` 非同步,
- `LinkedHashMap` 继承自 `HashMap` 
- `HashTable` 数组+链表 同步
- `TreeMap` 红黑树

> Map 的三种视图 `entrySet` `keySet` `values`

### HashMap
> `HashMap` 可以存储 null 的 key 和 value，但 null 作为键只能有一个，null 作为值可以有多个；  
> `Hashtable` 不允许有 null 键和 null 值，否则会抛出 `NullPointerException`  
> `HashMap` 默认的初始化大小为 16。之后每次扩充，容量变为原来的 2 倍    

> HashMap 多线程操作导致死循环,不建议在多线程下使用 HashMap,因为多线程下使用 HashMap 还是会存在其他问题比如数据丢失。
> 多线程下应该使用`ConcurrentHashMap`

### HashMap遍历
- `map.entrySet().iterator()`
- `map.keySet().iterator()`
- `for (Map.Entry<Integer, String> entry : map.entrySet())`
- `for (Integer key : map.keySet())`
- `map.forEach((key, value) -> {});`
- `map.entrySet().stream().forEach()`
- `map.keySet().stream().forEach()`

## comparable  Comparator 
- `comparable` 接口 包含compareTo 方法进行排序,一般实现接口重写compareTo方法进行自定义排序
- `Comparator` 接口 比较器,一般通过匿名类方式定义一个比较规则,配合`Collections.sort()`


## 无序性和不可重复性的含义是什么

- 无序性!= 随机 无序按照数据哈希值决定
- 不可重复 equals 判断返回false (同时重写equals 和hashcode)

## Collections 工具类

### 排序
```java
void reverse(List list)//反转
void shuffle(List list)//随机排序
void sort(List list)//按自然排序的升序排序
void sort(List list, Comparator c)//定制排序，由Comparator控制排序逻辑
void swap(List list, int i , int j)//交换两个索引位置的元素
void rotate(List list, int distance)//旋转。当distance为正数时，将list后distance个元素整体移到前面。当distance为负数时，将 list的前distance个元素整体移到后面

```

### 同步视图
```java
synchronizedCollection(Collection<T>  c) //返回指定 collection 支持的同步（线程安全的）collection。
synchronizedList(List<T> list)//返回指定列表支持的同步（线程安全的）List。
synchronizedMap(Map<K,V> m) //返回由指定映射支持的同步（线程安全的）Map。
synchronizedSet(Set<T> s) //返回指定 set 支持的同步（线程安全的）set。
```
## 集合使用注意事项


### 集合判空
> 判断所有集合内部的元素是否为空，使用 `isEmpty()` 方法，而不是 `size()==0` 的方式
> 因为 `isEmpty()` 方法的可读性更好，并且时间复杂度为 `O(1)`
> size() 方法的时间复杂度也是 `O(1)`，不过，也有很多复杂度不是 `O(1)` 的，比如 `java.util.concurrent` 包下的某些集合`ConcurrentLinkedQueue` 、`ConcurrentHashMap`

### 对象集合转Map
> 在使用 `java.util.stream.Collectors` 类的 `toMap()` 方法转为 Map 集合时，一定要注意当 value 为 null 时会抛 NPE 异常
> 因为 `toMap()` 方法 ，其内部调用了 Map 接口的 `merge()` 方法,`merge()` 方法会先调用 `Objects.requireNonNull()` 方法判断 value 是否为空


### 集合遍历
> 不要在 foreach 循环里进行元素的 `remove/add` 操作。remove 元素请使用 Iterator 方式，如果并发操作，需要对 Iterator 对象加锁
> Java8 开始，可以使用 `Collection#removeIf()`方法删除满足特定条件的元素


### 数组转集合
> 使用工具类 `Arrays.asList()` 把数组转换成集合时，不能使用其修改集合相关的方法， 它的 `add/remove/clear` 方法会抛出 `UnsupportedOperationException` 异常
> `Arrays.asList()` 方法返回的并不是 `java.util.ArrayList` ，而是 `java.util.Arrays` 的一个内部类,这个内部类并没有实现集合的修改方法或者说并没有重写这些方法

```java
List myList = Arrays.stream(myArray2).boxed().collect(Collectors.toList());
```

## 为什么是面向接口编程   
`List<Object> objects = new ArrayList<>()`
https://stackoverflow.com/questions/17059139/listobject-listobject-new-arraylistobject  

> 核心: 面向接口编程  程序指向接口，而不是实现(Programming to an interface, not to an implementation)
> 使用 `List` 后续修改代码譬如改成LinkedList 不会对旧代码有影响( breaking the rest of the code )


# ⭐并发  
## 线程和进程  
- 进程：操作系统的并发,进程间通信复杂,重量级
- 线程：进程内部并发,线程间通信简单,轻量级

> 本质区别： 是否单独占有内存地址空间及其他系统资源(I/O)  
> 进程是操作系统进行资源分配的基本单位,  
> 线程是系统进行调度(cpu时间片分配)的基本单位

## 线程隔离的内存数据区
- java栈 :Java方法执行的线程内存模型,栈帧 局部变量表、操作数栈、动态连接、方法出口
- 本地方法栈: JVM 运行 Native 方法准备的空间
- 程序计数器 :记录当前线程执行的位置,是描述本地方法运行过程的内存模型


## 并发并行的区别
- 并发 :同一时间段,多个任务都在执行
- 并行: 单位时间内,多个任务同时执行

## 多线程带来的问题
内存泄漏,死锁,线程安全问题等

## 线程的生命周期和状态
六种状态:  
- `NEW` 线程创建,未调用 `start` 方法
- `RUNNABLE` 表示当前线程正在运行中。处于RUNNABLE状态的线程在Java虚拟机中运行，也有可能在等待其他系统资源（比如I/O）
- `BLOCKED` 阻塞
- `WAITING` 等待 `wait()`
- `TIMED_WAITING` 计时等待 `sleep(long millis)` `wait(long millis)`
- `TERMINATED`

> 线程创建之后它将处于 NEW（新建） 状态，调用 start() 方法后开始运行，线程这时候处于 READY（可运行） 状态。可运行状态的线程获得了 CPU 时间片（timeslice）后就处于 RUNNING（运行） 状态
## 线程状态切换
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/基础12.png)


## 上下文切换
> 线程切换意味着需要保存当前线程的上下文，留待线程下次占用 CPU 的时候恢复现场。并加载下一个将要占用 CPU 的线程上下文。这就是所谓的 上下文切换

出现上下文切换的情况:  
- `sleep(),wait()`
- 系统分配的时间片用完
- 线程被阻塞
- 运行结束

## 死锁  

```java
public class DeadLock {

    static Object a = new Object();
    static Object b = new Object();

    public static void main(String[] args) {
        new Thread(()->{
            synchronized (a){
                System.out.println(Thread.currentThread().getName()+"持有a试图获取b");
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (b){
                    System.out.println(Thread.currentThread().getName()+"获取b");
                }
            }
        },"aa").start();

        new Thread(()->{
            synchronized (b){
                System.out.println(Thread.currentThread().getName()+"持有b试图获取a");
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (a){
                    System.out.println(Thread.currentThread().getName()+"获取a");
                }
            }
        },"bb").start();
    }
}
```


## sleep 和wait区别
- sleep 不会释放锁,wait释放锁
- wait用于线程间交互通信,sleep同于暂停执行
- wait 需要 notify 恢复或者定时恢复(指定时长),sleep 自动苏醒

## 为什么不直接调用run方法
>调用 start() 方法方可启动线程并使线程进入就绪状态，直接执行 run() 方法的话不会以多线程的方式执行

## synchronized 关键字

### 理解
解决多线程间访问资源的同步问题,可以保证被它修饰的方法或者代码块,在任意时刻只有一个线程执行.

### 使用
- 修饰实例方法,作用域当前对象实例`synchronized void method()`
- 修饰静态方法,作用与类`synchronized static void method()`
- 修饰代码块,`synchronized(this|object)` 表示进入同步代码库前要获得给定对象的锁。`synchronized(类.class) `表示进入同步代码前要获得 当前 class 的锁

### 双重校验锁实现对象单例
```java
public class Singleton {

    private volatile static Singleton uniqueInstance;

    private Singleton() {
    }

    public  static Singleton getUniqueInstance() {
       //先判断对象是否已经实例过，没有实例化过才进入加锁代码
        if (uniqueInstance == null) {
            //类对象加锁
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}

```
### JDK1.6 之后的 synchronized 关键字底层做了哪些优化
Java 6 及其以后，一个对象其实有四种锁状态，它们级别由低到高依次是:   
- 无锁状态
- 偏向锁状态
- 轻量级锁状态
- 重量级锁状态

### synchronized 和 ReentrantLock 的区别
- `synchronized` 依赖于 JVM 而 `ReentrantLock` 依赖于 API
- `ReentrantLock` 比 `synchronized` 增加了一些高级功能
    - 公平锁
    - `Condition` 通知


## volatile 关键字

volatile 关键字主要功能:  
- 保证变量的**内存可见性** 
- 禁止volatile 变量与普通变量**重排序**

> 内存可见性: 线程之间的可见性,当一个线程修改了共享变量时(会立即将变量值刷新到主内存),另一个线程可以读取到这个修改后的值

> 重排序: 为优化程序性能，对原有的指令执行顺序进行优化重新排序。重排序可能发生在多个阶段，比如编译重排序、CPU重排序等


## JMM java 内存模型
> JMM 抽象了线程和主内存之间的关系,JMM有如下特点:

- 所有的共享变量(如volatile 修饰的)都存在主内存中
- 每个线程都保存了一份该线程使用到的共享变量的副本
- 如果线程A和线程B之间通信必须经历如下2个步骤
  - 线程A将本地内存A更新过的共享便令刷新到主内存去
  - 线程B到主内存中读取线程A之前已经更新过的共享变量

> 注意: 线程A无法直接访问线程B的工作内存,线程间通信必须经过主内存
> 线程对共享变量的所有操作都必须在自己的本地内存中进行,不能直接从主内存中读取
> JMM通过控制主内存与每个线程的本地内存之间的交互,来提供内存可见性的保证

## 并发编程的三个重要特性
- 原子性: `synchronized` 可以保证代码片段的原子性
- 可见性: `volatile` 保证内存可见性
- 有序性: `volatile` 禁止指令重排序

## synchronized 关键字和 volatile 关键字的区别

- `synchronized` 可以修饰代码块和方法,能保证可见性和原子性,用于多线程访问资源同步
- `volatile` 只能修饰变量,能保证可见性不能保证原子性,用于多线程间变量的可见


## ThreadLocal
> ThreadLocal是一个本地线程副本变量工具类。内部是一个弱引用的Map来维护

> 最常见的ThreadLocal使用场景为用来解决数据库连接、Session管理等。数据库连接和Session管理涉及多个复杂对象的初始化和关闭
## 线程池
好处:  
- 降低资源消耗,线程复用
- **控制并发数量**
- 方便线程的统一管理

### Runnable Callable
> `Runnable` 不会返回结果,不会抛出检查异常( `Callable` 可以)
> 工具类 `Executors` 可以将 `Runnable` 对象转为 `Callable` 对象  
> `Executors.callable(Runnable task)`
> `Executors.callable(Runnable task, Object result)`

### execute()方法和 submit()方法的区别
- `execute()` 用于提交不需要返回值的任务,无法判断任务是否执行成功
- `submit()` 用于需要返回值的任务,线程池会返回一个Future 对象 可以判断任务是否成功,获取返回值

### 创建线程池
- `Executors.newFixedThreadPool(int)` 固定数量的线程
- `Executors.newSingleThreadExecutor()` 只有一个线程,顺序执行
- `Executors.newCachedThreadPool()` 可扩容的线程池,空闲线程默认保留60s

**上述线程池创建都是用`ThreadPoolExecutor()方法` 搭配不同参数进行创建**  
**不建议使用上述方法创建线程池,建议通过`ThreadPoolExecutor()`构造方法自定创建线程池**


## Atomic原子类  
> 原子类说简单点就是具有原子/原子操作特征的类  
> 并发包 `java.util.concurrent` 的原子类都存放在 `java.util.concurrent.atomic`
### CAS
> CAS全称:比较并交换 (compare and swap)  
> Java 中通过 `Unsafe` 类来实现CAS原理

### 原子类
- 基本类型 `AtomicInteger` `AtomicLong` `AtomicBoolean`
- 数组类型 `AtomicIntegerArray` `AtomicLongArray` `AtomicReferenceArray`
- 引用类型 `AtomicReference` `AtomicStampedReference` 
- ...

### AtomicInteger
```java
public final int get() //获取当前的值
public final int getAndSet(int newValue)//获取当前的值，并设置新的值
public final int getAndIncrement()//获取当前的值，并自增
public final int getAndDecrement() //获取当前的值，并自减
public final int getAndAdd(int delta) //获取当前的值，并加上预期的值
boolean compareAndSet(int expect, int update) //如果输入的数值等于预期值，则以原子方式将该值设置为输入值（update）
public final void lazySet(int newValue)//最终设置为newValue,使用 lazySet 设置之后可能导致其他线程在之后的一小段时间内还是可以读到旧的值。

```

## AQS
> AQS是 `AbstractQueuedSynchronizer` 的简称，即**抽象队列同步器**,类在 `java.util.concurrent.locks` 包下面
> `ReentrantLock``，Semaphore` ，`ReentrantReadWriteLock`，`SynchronousQueue`，`FutureTask`等等皆是基于AQS

### 原理
待理解

### AQS 定义两种资源的共享方式
- `Exclusive` 独占: 只有一个线程能执行,如 `ReentrantLock` 
  - 公平锁
  - 非公平锁
- `Share` 共享:多线程可以同时执行 如 `CountDownLatch` `Semapore` `CyclicBarrier` 读写锁(读可`new ReentrantReadWriteLock().readLock()`以看作共享,写`writeLock()`可以看作独占)

### 关于AQS组件
`CountDownLatch` `Semapore` `CyclicBarrier` 

详见JUC文档    
[juc的辅助类](Java/JUC开发?id=juc的辅助类)  

# JVM


# 新特性









# Spring

## 模块
> `Spring Core` `Spring Aspects` `Spring AOP` `Spring Data` `Spring Web` 

## IOC AOP
> `IoC (inverse of control) `控制反转,一种设计思想, 将对象创建和管理的权力交给Spring   
> 将对象之间的相互依赖关系交给 IoC 容器来管理，并由 IoC 容器完成对象的注入。这样可以很大程度上简化应用的开发
> 只需要使用配置文件或者注解配置,不用考虑对象如何创建 
>  IoC 容器是 Spring 用来实现 IoC 的载体， IoC 容器实际上就是个 Map（key，value），Map 中存放的是各种对象


> `AOP(Aspect-Oriented Programming:面向切面编程)` 
> 减少重复代码,降低耦合
> 基于动态代理,如果代理对象实现了某个接口,Spring AOP会使用JDK Proxy, 没有实现接口的对象使用 Cglib 


## Spring AOP 和 AspectJ AOP 有什么区别
Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。 Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)


## Bean
> bean 代指的就是那些被 IoC 容器所管理的对象  
> 可以通过 `xml` 或者注解配置bean

### bean的作用域
- `prototype` 多实例 调用(获取Bean)时创建
- `singleton` 单实例 **默认值**  IOC容器启动即创建
- ~~request 同一次请求创建一个实例~~
- ~~session 同一个session 创建一个实例~~


### 单例bean的线程安全问题
> 如果bean 实例中有定义可变的成员变量 ,当多个线程操作同一个bean存在资源竞争

解决办法:  
- 在bean中尽量避免定义可变的成员变量
- 使用 `ThreadLocal` 定义成员变量,通过其 `get` `set` 方法修改值

### @component @bean区别
> `@Component` 作用于类,通过类路径扫描(@ComponentScan)或者自动侦测装配到容器中
> `@Bean` 作用于方法,通过主动配置装配,自定义性更强,很多地方只能使用`@Bean` 来注册,如引用第三方类库配置


### 声明为Bean 的注解
- @Component
- @Bean
- @Repository
- @Service
- @Controller

### Bean 的生命周期
- 1. 通过构造器创建Bean 实例(无参构造)  
- 2. 设置Bean的属性值和对其他Bean的引用(调用Set方法)
- 3. 把 bean 实例传递 bean 前置处理器的方法 `postProcessBeforeInitialization` 
- 4. 调用 bean 的初始化的方法（需要进行配置初始化的方法）`init-method`  
- 5. 把 bean 实例传递 bean 后置处理器的方法 `postProcessAfterInitialization`  
- 6. bean可以调用了(初始化完成 可以使用)
- 7. 当容器关闭时候，调用 bean 的销毁的方法（需要进行配置销毁的方法）`destroy-method` 

> 关于 `initMethod` `destroyMethod`   
> xml配置 `init-method="initMethod" destroy-method="destoryMethod"`   
> 注解 `@Bean(initMethod = "initMethod",destroyMethod = "destroyMethod" )`  



## SpringMVC 流程
- 客户端发送请求,直接到 `DispatcherServlet`
- `DispatcherServlet` 根据请求信息调用 `HandlerMapping` 解析请求对应的 `Handler`(Controller)
- 到达 `Handler` 后,由`HandlerAdapter` 适配器处理
- `HandlerAdapter` 根据`Handler` 调用真正的处理器开始处理请求,执行业务逻辑
- 返回`ModelAndView` `Model` 是数据对象, `View` 是逻辑上的 `View` 
- `ViewResolver` 根据 `View` 查找真实的View
- `DispaterServlet` 把返回的 `Model` 传给 `View` (视图渲染)
- 把 `View` 返回给浏览器  


## Spring 用到了那些设计模式
- 工厂模式 BeanFactory ApplicationContext
- 代理模式 Spring Aop
- 单例设计模式 Bean 默认单例
- 适配器模式 AOP增强(通知) 


## Spring 事务
- 原子性: 一个事务中的所有操作,要么全部成功,要么全失败
- 一致性:事务开始和结束后数据的完整性没有被破坏
- 隔离性:数据库允许多个并发事务,同时对数据进行读写,隔离性可以防止多个事务并发导致数据不一致
- 持久性: 事务结束后,对数据的修改是永久的


> 你的程序是否支持事务首先取决于数据库 ，比如使用 MySQL 的话，如果你选择的是 `innodb` 引擎可以支持事务的。
> 但是，如果你的 MySQL 数据库使用的是 `myisam` 引擎的话，那不好意思，从根上就是不支持事务的  

### mysql 怎么保证原子性
> 要在异常发生时，对已经执行的操作进行回滚，在 `MySQL` 中，恢复机制是通过 `回滚日志（undo log）` 实现的

### 编程式事务 和 声明式事务
- 编程式事务: 通过 `TransactionTemplate` 或者 `TransactionManager` 手动管理事务，实际应用中很少使用
- 声明式事务: 在 XML 配置文件中配置或者直接基于注解（推荐使用）: 实际是通过 AOP 实现（基于`@Transactional` 的全注解方式使用最多）


配置:  

xml配置文件配置:  
```xml
<!-- 创建事务管理器  -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <!-- 注入 数据源 -->
    <property name="dataSource" ref="dataSource"/>
</bean>
<!-- 开启 事务 注解  -->
<tx:annotation-driven transaction-manager="transactionManager"/>
```

注解配置:  
`@EnableTransactionManagement`   
`@Transactional(propagation = Propagation.REQUIRED, isolation = Isolation.REPEATABLE_READ, timeout = -1, readOnly = false, rollbackFor = {}, noRollbackFor = {})`



- `propagation` 事务传播行为
- `isolation` 事务隔离级别
- `timeout` 超时时间
- `readOnly` 是否只读,默认false ,设置为true 只能读不能写
- `rollbackFor` 出现那些异常进行回滚
- `noRollbackFor` 出现那些异常不回滚

注意:
- `@Transactional` 只能作用在public 方法上
- 如果写在类上,那么对类中所有的public 方法生效
- 不推荐在接口上使用
- `@Transactional` 注解的方法所在的类**必须**被 `Spring` 管理，否则不生效

[Spring事务](Java/Spring?id=事务) 

## 常用注解  

### Bean 相关
- `@Configuration` 允许在 Spring 上下文中注册额外的 bean 或导入其他配置类
- `@ComponentScan` value 属性指定要扫描的包,就可以将 `@Component @Controller @Service @Repository` 注解的类放入IOC容器中
- `@Autowired` `@Qualifier("bookDao2")` 指定寻找id为bookDao2 的bean
- `@Primary` 注解设置 Bean的默认装配优先级
- `@Scope` 设置单实例
- `@GetMapping("users")` 等价于 `@RequestMapping(value="/users",method=RequestMethod.GET)` 


### 传值
- `@PathVariable`
- `@RequestHeader`
- `@RequestParam`
- `@CookieValue`


### 配置信息
- `@Value("${property}")` `@Value("张三")` ,也支持SpEL `@Value("#{20-3}")`
- `@ConfigurationProperties(prefix = "person")`
- `@PropertySource(value = {"classpath:/application.properties"})`

### 异常处理
- `@ControllerAdvice + @ExceptionHandler` 处理全局异常;底层是 `ExceptionHandlerExceptionResolver` 支持
- `@ResponseStatus + 自定义异常` 底层是 ResponseStatusExceptionResolver

```java
@ControllerAdvice
public class ExceptionHandlerTest {

  @ExceptionHandler({NullPointerException.class,ArrayIndexOutOfBoundsException.class})
  public String handlerExceptionTest(Exception e){

      //支持直接返回视图
      return "login";
  }
}

@ResponseStatus(value = HttpStatus.FORBIDDEN,reason = "用户数量太多")
public class UserTooManyException extends RuntimeException {
    public UserTooManyException(String message) {
        super(message);
    }
}

```


## 自动配置原理

`@SpringBootApplication` 内部  

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication{}
```
`@EnableAutoConfiguration`  内部  

```java
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
``` 


> 1. spring boot先加载所有自动配置类 ***AutoConfiguration
> 2. 每个配置类按照其内部条件生效,默认都会绑定配置文件中指定的值`(xxxxxAutoConfiguration ---> xxxxProperties(@ConfigurationProperties) ----> application.properties)`
> 3. 生效的配置类会给容器中装配很多组件,根据对应配置文件进行装配
> 4. 配置项中很多需要条件加载 `@Conditional***` 所以不会加载无用配置



## Junit 

[Junit5](Java/SpringBoot?id=junit5) 

- `@DisplayName` 为测试类或者测试方法设置展示名称
- `@BeforeEach` 表示在每个单元测试前执行
- `@AfterEach` 每个单元测试后执行
- `@BeforeAll` 所有单元测试前执行
- `@AfterAll` 所有单元测试后执行
- `@Tag` 表示单元测试类别
- `@Disable` 表示测试类或者测试方法不执行
- `@Timeout` 测试方法运行超时报错 默认秒,可以指定单位 
- `@ExtendWith` 为测试类或者测试方法提供扩展类引用
- `@RepeatedTest` 指定重复执行次数 ,重复执行测试方法


### 断言

方法 | 说明 | 
---------|----------|
 `assertEquals` | 判断两个对象或两个原始类型是否相等 | 
 `assertNotEquals` | 判断两个对象或两个原始类型是否不相等 |
 `assertSame` | 判断两个对象引用是否指向同一个对象 |
 `assertNotSame` | 判断两个对象引用是否指向不同的对象 |
 `assertTrue` | 判断给定的布尔值是否为 true |
 `assertFalse` | 判断给定的布尔值是否为 false |
 `assertNull` | 判断给定的对象引用是否为 nul |
 `assertNotNull` | 判断给定的对象引用是否不为 null |
 `assertArrayEquals` | 判断两个对象或原始类型的数组是否相等 |

# MyBatis

## #{}和${}的区别是什么
- `${}` 是properties文件中的变量占位符,属于静态文本替换,对应位置会被替换为参数值
- `#{}` mybatis 会将sql中的 `#{}` 替换为 ? 号,在sql执行前会将?替换为参数值

## Xml 映射文件中标签
- `select|insert|update|delete`
- `resultMap|sql|include`
- `trim|where|if|choose|when|otherwise|foreach`

## Dao 接口的工作原理是什么？Dao 接口里的方法，参数不同时，方法能重载吗 
> Dao接口的全限定类名就是映射xml文件的namespace值,方法名就是xml文件`MappedStatement`中的id值,方法参数就是传递给sql的参数
> 通过接口全限定类名+方法名 可以唯一定位一个 `MappedStatement` 
> `com.mybatis3.mappers.StudentDao.findStudentById` 就会找到 namespace 为`com.mybatis3.mappers.StudentDao` 下 id为`findStudentById` 的`MappedStatement`

> Dao 接口里的方法可以重载，但是 Mybatis 的 XML 里面的 ID 不允许重复 
> Dao 接口方法可以重载，但是需要满足以下条件:
> > 仅有一个无参方法和一个有参方法,多个有参方法时，参数数量必须一致  

## MyBatis 执行批量插入，能返回数据库主键列表吗？ 
批量插入和单条插入道理相同   
- `useGeneratedKeys` 开启自增,标识其使用了自增id
- `keyProperty` 将自增的主键的值赋值给对应实体类的属性(赋值给id属性)

## 动态sql
> `if` `where` `trim` `choose when otherwise` `foreach` `sql include`


## MyBatis如何封装返回结果
-  使用`<resultMap>` 标签指定映射关系
-  使用别名作为属性名,自动映射匹配

## MyBatis 多对一\一对多 查询 

### 多对一三种方法
[MyBatis多对一三种方法](Java/MyBatis?id=多对一映射关系处理的三种方法)

- 通过级联属性赋值
- 通过 `association` 标签
- 通过 `association` 标签 + 分布查询

### 一对多 两种
[MyBatis一对多 两种方法](Java/MyBatis?id=解决一对多映射)
- 使用 `collection` 标签
- 分步查询

## MyBatis 的延迟加载
> MyBatis 仅支持 `association` 关联对象和 `collection` 关联集合对象的延迟加载， `association` 指的就是一对一， `collection` 指的就是一对多查询

> `azyLoadingEnabled` :延迟加载的全局开关,当开启时,所有关联对象(分布查询的第二步,第三步...)都会延迟加载
> `aggressiveLazyLoading` :当开启时,任何方法的调用都会加载该对象的所有属性,否则每个属性会按需加载
使用配置文件或者xml配置进行配置开启延迟加载  
```xml
  <settings>
        <!-- 开启全局延迟加载-->
        <setting name="lazyLoadingEnabled" value="true"/>
    </settings>
```

或者

```properties
mybatis.configuration.lazy-loading-enabled=true
#false 为按需加载
mybatis.configuration.aggressive-lazy-loading=false

```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/mybatis2.png)

> 基本原理: 使用 `CGLIB` 创建目标你对象的代理对象,当调用目标方法是,进入拦截器方法,比如调用 `a.getB().getName()` ，
> 拦截器` invoke()` 方法发现 `a.getB()` 是 `null` 值，那么就会单独发送事先保存好的查询关联 B 对象的 sql，把 B 查询上来，
> 然后调用 `a.setB(b)`，于是 a 的对象 b 属性就有值了，接着完成 a.getB().getName() 方法的调用

## MyBatis 的 Xml 映射文件中，不同的 Xml 映射文件，id 是否可以重复
> 不同的xml文件,如果配置了namespace 那么id可以重复,如果没有配置namespace 那么id不能重复
> 原因就是 使用了`namespace+id`(即 mapper全限定类名+方法名) 作为 `Map<String, MappedStatement>` 的 key使用


## ⭐MyBatis 中如何执行批处理
> 使用 `BatchExecutor` 完成批处理  


## ⭐MyBatis 都有哪些 Executor 执行器？它们之间的区别是什么  
> MyBatis 有三种基本的 Executor 执行器， `SimpleExecutor` 、 `ReuseExecutor` 、 `BatchExecutor`  

- `SimpleExecutor` 每执行一次update或者select 就开启一个Statement 对象,用完立即关闭 Statement 对象

## MyBatis 映射文件中，如果 A 标签通过 include 引用了 B 标签的内容，请问，B 标签能否定义在 A 标签的后面，还是说必须定义在 A 标签的前面
随便 无顺序之分
> MyBatis 解析 A 标签，发现 A 标签引用了 B 标签，但是 B 标签尚未解析到，尚不存在，
> 此时，MyBatis 会将 A 标签标记为未解析状态，然后继续解析余下的标签，包含 B 标签，待所有标签解析完毕，
> MyBatis 会重新解析那些被标记为未解析的标签，此时再解析 A 标签时，B 标签已经存在，A 标签也就可以正常解析完成了


## 简述 MyBatis 的 Xml 映射文件和 MyBatis 内部数据结构之间的映射关系

> 在 Xml 映射文件中， `<parameterMap>` 标签会被解析为 `ParameterMap` 对象，其每个子元素会被解析为 `ParameterMapping` 对象。 
> `<resultMap>` 标签会被解析为 `ResultMap` 对象，其每个子元素会被解析为 `ResultMapping` 对象。
> 每一个 `<select>、<insert>、<update>、<delete>` 标签均会被解析为 `MappedStatement` 对象，标签内的 sql 会被解析为 `BoundSql` 对象


## 一级缓存 
> 一级缓存是 `SqlSession` 级别的,默认开启,通过同一个 `SqlSession` 查询的数据会被缓存,
> 下次查询相同的数据,就会从缓存中直接获取,不会从数据库重新访问

一级缓存失效的四种情况:   
- 不同的 `SqlSession` 对应不同的一级缓存
- 同一个 `SqlSession` 但是查询条件不同
- 同一个 `SqlSession` 两次查询期间执行了任何一次增删改操作
- 同一个 `SqlSession` 两次查询间手动清空了缓存 `sqlSession.clearCache();`



## 二级缓存
> 二级缓存是 SqlSessionFactory 级别, 通过同一个SqlSessionFactory 创建的 SqlSession查询的结果会被缓存,
> 此后若再次执行相同的查询语句,结果就会从缓存中获取

二级缓存开启的条件:  
- 在核心配置文件中,设置全局配置属性 `cacheEnable=true` (默认为true) 
- 在映射文件中设置标签 `<cache />`
- 二级缓存必须存在 `SqlSession` 关闭或提交之后有效,提交之前在一级缓存中
- 查询的数据所转换的实体类型必须实现序列化接口

> 两次查询之间执行了任意的增删改操作,会使一级和二级缓存同时失效  


## 缓存查询顺序  
- 先查询二级缓存
- 二级没有命中,查询一级缓存
- 一级没有命中,查询数据库
- `SqlSession` 关闭后一级缓存中的数据会写入二级缓存





