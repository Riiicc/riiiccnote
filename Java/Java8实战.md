## Lambda表达式  
### 有效表达式
- `(String s) -> s.length()`  返回值为int s的长度
- `(Apple a) ->a.getWeight() > 150 ` 返回值为Boolean weight是否超过150
- `() -> 42` 没有参数 直接返回int
- `(Apple a1,Apple a2) -> a1.getWeight().compareTo(a2.getWeight())` 返回int(compareTo的返回结果)
- 多行lambda表达式
```java
(int x,int y)->{
    System.out.println("Result");
    System.out.println(x+y);
}
```
### 基本语法
- `(parameters) -> expression`  
- `(parameters) -> { statements; }`    

### 函数式接口  
- 函数式接口就是只定义**一个**抽象方法的接口  
- 函数式接口带有 `@FunctionalInterface` 的标注  
- 这个标注用于表示该接口会设计成一个函数式接口。如果你用 `@FunctionalInterface` 定义了一个接口，而它却不是函数式接口的话，编译器将返回一个提示原因的错误  
- `@FunctionalInterface` 不是必需的，但对于为此设计的接口而言，使用它是比较好的做法

### 使用函数式接口  

#### Predicate  
根据Boolean 判断 `T -> boolean`
```java
@FunctionalInterface
public interface Predicate<T>{
    boolean test(T t);
}
```

#### Consumer
遍历执行操作 `T -> void`
```java
@FunctionalInterface
public interface Consumer<T>{
    void accept(T t);
}
```

#### Function
转换列表 `T -> R`
```java
@FunctionalInterface
public interface Function<T, R>{
    R apply(T t);
}
```
Java 8为我们前面所说的函数式接口带来了一个专门的版本，以便在输入和输出都是原始类
型时避免自动装箱的操作;  
例如:
- `IntPredicate` `LongPredicate`
- `IntConsumer`  `LongConsumer` 

上述接口会将`<T>` 直接限定为对应基础类型,和`Function,Predicate`等并无继承实现关系
```java
@FunctionalInterface
public interface IntFunction<R> {
    /**
     * Applies this function to the given argument.
     *
     * @param value the function argument
     * @return the function result
     */
    R apply(int value);
}
```
### Lambda 对局部变量的限制
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/实战1.png)

### 方法引用   
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/实战2.png)  

> 注:第三个方法 会默认将第**一个参数**作为方法目标   

构建方法引用的三种方法:
- 指向**静态方法**的方法引用 `Class::staticMethod`  
- 指向任意类型**实例方法**的方法引用`Class::instanceMethod`
- 指向现有对象的实例方法的方法引用 `object::instanceMethod` 

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/实战3.png)  

### 复合Lambda实例  

#### 比较器复合 
```java
    List<Apple> aplist = new ArrayList<>();

    aplist.sort(Comparator.comparing(Apple::getWeight));

    aplist.sort(Comparator.comparing(Apple::getWeight).reversed());

    aplist.sort(Comparator.comparing(Apple::getWeight).reversed().thenComparing(Apple::getColor)); 
```

#### 谓词复合  
谓词接口包括三个方法： `negate` 、 `and` 和 `or` ，让你可以重用已有的 Predicate 来创建更复杂的谓词
```java
    ArrayList<String> strings = new ArrayList<>();
    strings.add("1");
    strings.add("2");
    strings.add("3");
    strings.add("4"); 
    Predicate<String> none = (String s) ->!s.equals("2");// 1,3,4  不等于2
    Predicate<String> negate = none.negate(); //2  等于2
    Predicate<String> and = none.and((String s) ->!s.equals("3")); //1,4 不等于2 并且不等于3
    Predicate<String> or = negate.or((String s) ->s.equals("3")); //2,3  等于2或者等于3
```
#### 函数复合  
 Function 接口为此配了 `andThen` 和 `compose` 两个默认方法，它们都会返回 Function 的一个实例  
```java
    Function<Integer,Integer> y = x -> x + 1;
    Function<Integer,Integer> g = x -> x * 2;
    Function<Integer, Integer> a = y.andThen(g);  // 函数 g(y(x))
    Function<Integer, Integer> b = y.compose(g); // 函数 y(g(x))

    List<Integer> map2 = map(Arrays.asList(1, 2, 3), y); //2,3,4
    List<Integer> map21 = map(Arrays.asList(1, 2, 3), g);//2,4,6
    List<Integer> map22 = map(Arrays.asList(1, 2, 3), a);//4,6,8  (1+1)*2, (2+1)*2, (3+1)*2
    List<Integer> map23= map(Arrays.asList(1, 2, 3), b);//3,5,7, (1*2)+1 

    public static <T,R> List<R> map(List<T> list, Function<T,R> f){
        ArrayList<R> rlist = new ArrayList<>();
        for (T t : list) {
            rlist.add(f.apply(t));
        }
        return rlist;
    }
```

## 引入流、使用流     
### 关于流 
> 流只能遍历一次，类比IO流   

流操作 `filter` `map` `limit` `sorted` `distinct` `forEach` `count` `collect`
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/实战8.png)

### 筛选和切片   
> Streams接口支持filter方法，这个操作接受一个**谓词** （返回Boolean的函数）  
>> 主要包含`filter`过滤 `distinct`去重  `limit`限制  `skip` 跳过

```java
        List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4, 5, 6, 7, 8);
        numbers.stream().filter(i -> i % 2 == 0) //2 2 4 6 8
                .distinct()//2 4 6 8
                .limit(3)//2 4 6
                .skip(1)//4 6
                .forEach(System.out::println);
```

### 映射 
> 流支持**map**方法，会接受一个函数作为参数  

```java
        ArrayList<Apple> apples = new ArrayList<>();
        List<Integer> collect = apples.stream()
                .map(Apple::getWeight)
                .collect(Collectors.toList());

        List<Long> collect1 = apples.stream()
                .map(Apple::getWeight)
                .map(Integer::longValue)
                .collect(Collectors.toList());


```
> 流的**扁平化（flatMap）**，例如给定单词列表["Hello","World"] ，你想要返回列表 ["H","e","l", "o","W","r","d"] 代码过程如下  

```java

        List<String> strings = Arrays.asList("Hello", "World");
        //第一次尝试
        List<String[]> collect2 = strings.stream().map(s -> s.split(""))
                .distinct().collect(Collectors.toList());
        //第二次
        List<Stream<String>> collect3 =
                strings.stream().map(s -> s.split("")).map(Arrays::stream)
                        .distinct().collect(Collectors.toList());
        //使用flatMap
        List<String> collect4 =
                strings.stream().map(s -> s.split("")).flatMap(Arrays::stream)
                        .distinct().collect(Collectors.toList());
```
> 使用 flatMap 方法的效果是，各个数组并不是分别映射成一个流，而是映射成流的内容。所
> 有使用 map(Arrays::stream) 时生成的单个流都被合并起来，即扁平化为一个流。图5-6说明了
> 使用 flatMap 方法的效果。


两个案例参考  
```java
//1
        List<Integer> numbers1 = Arrays.asList(1, 2, 3);
        List<Integer> numbers2 = Arrays.asList(3, 4);
        //返回 [(1, 3), (1, 4), (2, 3), (2, 4), (3, 3), (3, 4)]
        List<int[]> pairs =
                numbers1.stream()
                        .flatMap(i -> numbers2.stream()
                                .map(j -> new int[]{i, j})
                        )
                        .collect(Collectors.toList());
//2
        // 返回[(2, 4), (3, 3)]
        List<int[]> pairs1 =
                numbers1.stream()
                        .flatMap(i ->
                                numbers2.stream()
                                        .filter(j -> (i + j) % 3 == 0)
                                        .map(j -> new int[]{i, j})
                        )
                        .collect(Collectors.toList());
```
### 查找和匹配  
> Stream API通过 
> allMatch 、 （匹配的所有）
> anyMatch 、 （任一匹配）
> noneMatch 、 （不匹配的所有）
> findFirst  （第一个）
> findAny  （任意一个）
> 方法提供了这样的工具  

> anyMatch 、 allMatch 和 noneMatch 这三个操作都用到了我们所谓的短路，这就是大家熟悉的Java中 && 和 || 运算符短路在流中的版本

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/实战9.png)

### 归约 reduce  
> reduce 的初始值也会作为流内容的一部分参与`reduce(BinaryOperator<T> accumulator);` 的内部计算，注意初始值的用法，逻辑不是默认值`getOrElse`这样的

```java
List<Integer> numbers = Arrays.asList(2, 3, 4, 5);
//有初始值
Integer reduce = numbers.stream().reduce(1, (a, b) -> a + b); //1+2+3+4+5
Integer reduce2 = numbers.stream().reduce(1, Integer::sum); //同上

Integer reduce1 = numbers.stream().reduce(2, (a, b) -> a * b);// 2 *2*3*4*5  

// 无初始值
Optional<Integer> sum = numbers.stream().reduce((a, b) -> (a + b)); //无初始值可能为空，所以使用Optional     

// 最大值
Optional<Integer> max = numbers.stream().reduce(Integer::max); 

//计算list长度
int count = numbers.stream().map(d -> 1).reduce(0, (a, b) -> a + b);

// 根据上面的numbers 内容 此时返回值永远为0
Optional<Integer> min = numbers.stream().reduce(0,Integer::min);

```
### 操作汇总
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/实战10.png)

### 数值流
> Java 8引入了三个原始类型特化流接口来解决这个问题： `IntStream` 、 `DoubleStream` 和`LongStream` ，
> 分别将流中的元素特化为 int 、 long 和 double ，从而避免了暗含的**装箱**成本。
> 比如对数值流求和的 sum ，找到最大元素的 max 。此外还有在必要时再把它们转换回对象流的方法  

> 将流转换为特化版本的常用方法是`mapToInt` `mapToDouble` `mapToLong` 这些方法和前面说的map方法的工作方式一样,
> 只是他们返回的而是一个***特化流***,而不是`Stream<T>`
```java
// 对属性求和
int calories = menu.stream()  //返回Stream<T>
        .mapToInt(Dish::getCalories) //返回IntStream
        .sum();

```
> 一旦有了数值流，你可能会想把它转换回非特化流。例如， `IntStream` 上的操作只能产生原始整数： `IntStream` 的 `map` 操作接受的Lambda必须接受 `int` 并返回 `int` 

```java
IntStream intStream = menu.stream().mapToInt(Dish::getCalories); 
Stream<Integer> stream = intStream.boxed();
```
>  Optional 可以用Integer 、 String 等参考类型来参数化。对于三种原始流特化，也分别有一个 Optional 原始类型特化版本： OptionalInt 、 OptionalDouble 和 OptionalLong

```java
OptionalInt maxCalories = menu.stream()
        .mapToInt(Dish::getCalories)
        .max();

int max = maxCalories.orElse(1);
```
> Java 8引入了两个可以用于 IntStream 和 LongStream 的静态方法，帮助生成这种范围：range 和 rangeClosed 。
> 这两个方法都是第一个参数接受起始值，第二个参数接受结束值。但range 是不包含结束值的，而 rangeClosed 则包含结束值

```java
//50个偶数  取值范围1~100
IntStream intStream = IntStream.rangeClosed(1, 100).filter(t -> t % 2 == 0);
long count = intStream.count(); //50
//49个偶数  取值范围1~99
IntStream intStream1 = IntStream.range(1, 100).filter(t -> t % 2 == 0);
long count1 = intStream1.count();//49

```
### 构建流  
> 你可以使用静态方法 Stream.of ，通过显式值创建一个流。它可以接受任意数量的参数

```java

Stream<String> stream = Stream.of("Java 8 ", "Lambdas ", "In ", "Action");
stream.map(String::toUpperCase).forEach(System.out::println);

//空流
Stream<String> emptyStream = Stream.empty(); 

//求和
int[] numbers = {2, 3, 5, 7, 11, 13};
int sum = Arrays.stream(numbers).sum();

// 文件流
long uniq = 0;
try (Stream<String> lines = Files.lines(Paths.get("d://test.txt"),Charset.defaultCharset())){
    uniq = lines.flatMap(l -> Arrays.stream(l.split(""))).distinct().count();
    System.out.println(uniq);
}catch (Exception e){
}

```
> Stream API提供了两个静态方法来从函数生成流： `Stream.iterate` 和 `Stream.generate` 。  

```java
//iterate 方法接受一个初始值（在这里是 0 ），还有一个依次应用在每个产生的新值上的Lambda（ UnaryOperator<t> 类型）
//iterate 方法生成了一个所有正偶数的流：流的第一个元素是初始值 
Stream.iterate(0, n -> n + 2)
        .limit(10)
        .forEach(System.out::println);

// generate 接受一个 Supplier<T> 类型的Lambda提供新的值
Stream.generate(Math::random)
        .limit(5)
        .forEach(System.out::println);

//生成一个全是1的无限流
IntStream ones = IntStream.generate(() -> 1);
```



## 收集器Collectors
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/实战11.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/实战12.png)
### 规约和汇总  
利用`Collectors.counting`方法计数  

> idea提示下面的情况由上到下为不断优化的结果  
```java
        Long collect1 = transactions.stream().collect(Collectors.counting());
        //使用自带count方法计数
        long count = transactions.stream().count();

        int size = transactions.size();
```


查找流中的最大值`maxBy`和最小值`minBy`   
> idea提示下面的情况由上到下为不断优化的结果  
```java
        Optional<Transaction> max1 =
                transactions.stream().collect(Collectors.maxBy(Comparator.comparingInt(Transaction::getValue)));
        Optional<Transaction> min1 =
                transactions.stream().collect(Collectors.minBy(Comparator.comparingInt(Transaction::getValue)));
        //
        Optional<Transaction> max2 = transactions.stream().max(Comparator.comparingInt(Transaction::getValue));
        Optional<Transaction> min2 = transactions.stream().min(Comparator.comparingInt(Transaction::getValue));
```
> `Collectors` 类专门为汇总提供了一个工厂方法：` Collectors.summingInt`  
> `Collectors.summingLong` 和 `Collectors.summingDouble` 方法的作用完全一样，可以用于求和字段为 long 或 double 的情况。  
> 但汇总不仅仅是求和；还有 `Collectors.averagingInt` ，连同对应的 `averagingLong` 和 `averagingDouble` 可以计算数值的平均数
>  `Collectors.summarizing` 操作你可以就数出菜单中元素的个数，并得到总和、平均值、最大值和最小值;同样有`summarizingLong` 和 `summarizingDouble`

```java
        transactions.stream().collect(Collectors.averagingInt(Transaction::getValue));
        transactions.stream().collect(Collectors.summingInt(Transaction::getValue));
        //上面的优化形式
        transactions.stream().mapToInt(Transaction::getValue).sum();

        IntSummaryStatistics IntSum = transactions.stream().collect(Collectors.summarizingInt(Transaction::getValue));
        IntSum.getAverage();
        IntSum.getCount();
        IntSum.getMax();
        IntSum.getMin();
        IntSum.getSum();

```
> `joining` 工厂方法返回的收集器会把对流中每一个对象应用 toString 方法得到的所有字符串连接成一个字符串。 

下面两种效果相同  
```java
        String collect3 = transactions.stream().map(t -> t.getTrader().getCity()).collect(Collectors.joining(","));
        String collect4 =
                transactions.stream().map(Transaction::getTrader).map(Trader::getName).collect(Collectors.joining(","));
        
        //同样可以使用reduc实现joining
        Optional<String> collect6 =
                transactions.stream().map(Transaction::getTrader).map(Trader::getName).collect(Collectors.reducing((a
                        , b) -> a + b));
        String collect7 = transactions.stream().map(Transaction::getTrader).collect(Collectors.reducing("",
                Trader::getName, (a, b) -> a + b));
        String collect7 = transactions.stream().map(Transaction::getTrader).map(Trader::getName).reduce("",
                (a, b) -> a + b);
```

> 广义的规约汇总:事实上，我们已经讨论的所有收集器，都是一个可以用 `reducing` 工厂方法定义的归约过程的特殊情况而已。` Collectors.reducing` 工厂方法是所有这些特殊情况的一般化。

```java
        Optional<Transaction> collect5 =
                transactions.stream().collect(Collectors.reducing((a, b) -> a.getValue() > b.getValue() ? a : b));
        //上面的简化
        Optional<Transaction> collect5 =
                transactions.stream().reduce((a, b) -> a.getValue() > b.getValue() ? a : b);

```

### 分组  

基本使用 

```java
        Map<Integer, List<Transaction>> collect =
                transactions.stream().collect(Collectors.groupingBy(Transaction::getYear));
```

> 自定义grouping 分组

```java
        Map<String, List<Transaction>> collect8 = transactions.stream().collect(Collectors.groupingBy(tr -> {
            if (tr.getValue() <= 300) {
                return "laji";
            } else {
                return "keyi";
            }
        }));
```

> 多级分组 (同时按照两个属性分组) 生成一个嵌套Map

```java
        Map<Integer, Map<String, List<Transaction>>> collect9 =
                transactions.stream().collect(Collectors.groupingBy(Transaction::getYear, Collectors.groupingBy(tr -> {
            if (tr.getValue() <= 300) {
                return "laji";
            } else {
                return "keyi";
            }

        })));

```

> 按子组收集数据

```java
        //收集每年多少条数据
        Map<Integer, Long> collect10 = transactions.stream().collect(Collectors.groupingBy(Transaction::getYear,
                Collectors.counting()));
```
> 普通的单参数 `groupingBy(f)` （其中 f 是分类函数）实际上是 `groupingBy(f,toList())` 的简便写法  

```java
        //获取每年最高交易订单 注意Optional
        Map<Integer, Optional<Transaction>> collect11 =
                transactions.stream().collect(Collectors.groupingBy(Transaction::getYear,
                Collectors.maxBy(Comparator.comparingInt(Transaction::getValue))));

```
> 因为分组操作的 Map 结果中的每个值上包装的 Optional 没什么用，所以你可能想要把它们去掉。要做到这一点，或者更一般地来说，把收集器返回的结果转换为另一种类型，你可以使用`Collectors.collectingAndThen` 工厂方法返回的收集器

```java
        // 通过 collectingAndThen 将上面的示例 返回值Optional 改为Transaction 
        Map<Integer, Transaction> collect12 = transactions.stream()
                .collect(Collectors.groupingBy(Transaction::getYear, Collectors.collectingAndThen(Collectors.maxBy(Comparator.comparingInt(Transaction::getValue)),Optional::get)));
        //使用summingInt
        Map<Integer, Integer> collect = transactions.stream().collect(groupingBy(Transaction::getYear,
                summingInt(Transaction::getValue)));

        Map<Integer, List<Integer>> collect1 = transactions.stream().collect(groupingBy(Transaction::getYear,
                mapping(Transaction::getValue, toList())));
        //通过Collection 可以指定具体的HashSet LinkedList  ArrayList...
        Map<Integer, HashSet<Integer>> collect2 = transactions.stream().collect(groupingBy(Transaction::getYear,
                mapping(Transaction::getValue, toCollection(HashSet::new))));

```


### 分区  
> `partitioningBy` 分区函数,分区函数返回一个Boolean,结果分成两组 true 和 false 同样支持 `groupingBy` 的嵌套形式  

```java
        //将数据按照是否是2011年进行分组 
        Map<Boolean, List<Transaction>> collect3 = transactions.stream().collect(partitioningBy(Transaction::is2011));
        //等价显式版
        Map<Boolean, List<Transaction>> collect =
                transactions.stream().collect(partitioningBy(t -> t.getYear() == 2011));
        collect.get(true);//获取2011年的list
        //返回示例 
        //{false=[pork, beef, chicken, prawns, salmon],true=[french fries, rice, season fruit, pizza]}

```

### 示例-将数字按质数和非质数分区

> 设你要写一个方法，它接受参数 int n，并将前n个自然数分为质数和非质数。

```java
    public boolean isPrime(int candidate) {
        return IntStream.rangeClosed(2, candidate).noneMatch(i -> candidate % i == 0);
    }

    public  boolean isPrimeSqrt(int candidate){
        int sqrt = (int)Math.sqrt((double) candidate);
        return IntStream.rangeClosed(2,sqrt).noneMatch(i->candidate%i ==0);
    }
    public Map<Boolean, List<Integer>> partitionPrimes(int n) {
        return IntStream.rangeClosed(2, n).boxed()
                .collect(
                        partitioningBy(this::isPrime));
    }

```




## Optional 取代Null （重点）
`java.util.Optional<T>`    

### 声明一个Optional 
```java
//1
Optional<Car> car = Optional.empty();  

Car car1 = new Car();
//2
Optional<Car> car11 = Optional.of(car1);
//3
Optional<Car> car12 = Optional.ofNullable(car1);
```

### 应用Optional的几种方式  

#### map从 Optional 对象中提取和转换值
>  map 操作会将提供的函数应用于流的每个元素   


> 你可以把 Optional 对象看成一种特殊的集合数据，它**至多包含一个元素**。如果 Optional 包含一个值，
> 那函数就将该值作为参数传递给 map ，对该值进行转换。如果 Optional 为空，就什么也不做
```java
Insurance insurance1 = new Insurance();
Optional<Insurance> insurance11 = Optional.ofNullable(insurance1);
Optional<String> s = insurance11.map(Insurance::getName);
```
#### 使用 flatMap 链接 Optional 对象  
> flatMap 可以将多个流扁平化合并为一个流   

```java
Optional<Person> person1 = Optional.empty();
String s =
        person1.flatMap(Person::getCar).
                flatMap(Car::getInsurance)
                .map(Insurance::getName)
                .orElse("nullDefault");
```
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/实战4.png) 

> 域模型中使用 Optional**无法序列化**， 由于 Optional 类设计时就没特别考虑将其作为类的字段使用，所以它也并未实现
> Serializable 接口。由于这个原因，如果你的应用使用了某些要求序列化的库或者框架，在
> 域模型中使用 Optional ，有可能引发应用程序故障

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/实战5.png)
#### 使用 filter 剔除特定的值  

```java

//创建
Insurance insurance1 = new Insurance();
Optional<Insurance> insurance11 = Optional.ofNullable(insurance1);

//过滤条件  练习谓词复合
Predicate<Insurance> p = x->x.getName().equals("1"); 
Predicate<Insurance> p1 = x->x.getName().equals("2");
Predicate<Insurance> negate = p1.negate();

//name ==1或者name ==2的   否则返回空Insurance 注意两个创建方式 
Insurance insurance = insurance11.filter(p.or(p1)).orElseGet(Insurance::new);
//name==1并且name ==2的 举例
Insurance insurance1 = insurance11.filter(p.and(p1)).orElse(new Insurance());

//name!=2的
Insurance insurance13 = insurance11.filter(negate).orElseGet(Insurance::new);
```

#### Optional类的方法
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/实战6.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/实战7.png) 

### 使用Optional
#### 用 Optional 封装可能为 null 的值
```java
Optional<Object> value = Optional.ofNullable(map.get("key"));
```
#### 异常与 Optional 的对比  
> 由于某种原因，函数无法返回某个值，这时除了返回 null ，Java API比较常见的替代做法是抛出一个异常。
> 这种情况比较典型的例子是使用静态方法 Integer.parseInt(String) ，将String 转换为 int 。  

> 基础类型的Optional `OptionalInt` 、 `OptionalLong` 以及 `OptionalDouble`对象应该避免使用,
> 因为基础类型的 Optional 不支持 map 、flatMap 以及 filter 方法


#### 判断、过滤内容
```java
    //判断password 长度及内容
    String password = "password";
    Optional<String>  opt = Optional.ofNullable(password);

    Predicate<String> len6 = pwd -> pwd.length() > 6;
    Predicate<String> len10 = pwd -> pwd.length() < 10;
    Predicate<String> eq = pwd -> pwd.equals("password");

    boolean result = opt.map(String::toLowerCase).filter(len6.and(len10 ).and(eq)).isPresent();
    System.out.println(result);
```


## 新的日期和时间API

### LocalDate、LocalTime 

> LocalDate 类。该类的实例是一个不可变对象，它只提供了简单的日期，并不含当天的时间信息。
> 另外，它也不附带任何与时区相关的信息。

```java
        LocalDate date = LocalDate.of(2014, 3, 1);
        LocalDate.of(2014, Month.MARCH,16);
        date.getYear(); //2014
        date.getMonth();//MARCH
        date.getMonthValue();//3
        date.getDayOfMonth();//1
        date.getDayOfWeek();//SATURDAY
        date.getDayOfWeek().getValue();//6
        int len = date.lengthOfMonth();//31
        boolean leap = date.isLeapYear();//false 闰年
        int i3 = date.lengthOfYear();//当前年份总天数 365
        long l = date.toEpochDay();//从1970-1-1 到现在的天数
```
> 还可以通过传递 `TemporalField` 参数给 get 方法拿到同样的信息, `TemporalField` 是一个接口，
> 它定义了如何访问 `temporal` 对象某个字段的值。 `ChronoField` 枚举实现了这一接口，所以你可以很方便地使用 get 方法得到枚举元素的值

```java
//使用 TemporalField 读取 LocalDate 的值
        int i = date.get(ChronoField.YEAR);//2014
        int i2 = date.get(ChronoField.MONTH_OF_YEAR);//3
        int i1 = date.get(ChronoField.DAY_OF_MONTH);//1
```
> 一天中的时间，比如13:45:20，可以使用 LocalTime 类表示。你可以使用 of 重载的两个工厂方法创建 LocalTime 的实例。
> 第一个重载函数接收小时和分钟，第二个重载函数同时还接收秒。同 LocalDate 一样， `LocalTime` 类也提供了一些 getter 方法访问这些变量的值

```java
        LocalTime time = LocalTime.of(23, 45, 20);
        int hour = time.getHour();//23
        int minute = time.getMinute();//45
        int second = time.getSecond();//20
        LocalDate date = LocalDate.parse("2014-03-18");
        LocalTime time = LocalTime.parse("13:45:20");
```
> 合并日期和时间, `LocalDateTime` ，是 `LocalDate` 和 `LocalTime` 的合体。它同时表示了日期和时间，但不带有时区信息
> 可以直接创建,也可以通过日期和时间合并构造

```java
//直接构造1
LocalDateTime dt1 = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45, 20);
//通过LocalDate 和LocalTime构造
LocalDateTime dt2 = LocalDateTime.of(date, time);
//在LocalDate 基础上构造
LocalDateTime dt3 = date.atTime(13, 45, 20);
LocalDateTime dt4 = date.atTime(time);
//在 LocalTime 基础上构造
LocalDateTime dt5 = time.atDate(date);
```

> 可以使用 `toLocalDate` 或者 `toLocalTime` 方法，从 `LocalDateTime` 中提取 `LocalDate` 或者 `LocalTime` 组件

```java
LocalDate date1 = dt1.toLocalDate();
LocalTime time1 = dt1.toLocalTime();
```


### Instant 、 Duration 以及 Period  

### 操纵、解析和格式化日期
```java
//通过with 相关修改date值  直接修改
LocalDate date1 = LocalDate.of(2014, 3, 18);
LocalDate date2 = date1.withYear(2011);
LocalDate date3 = date2.withDayOfMonth(25);
LocalDate date4 = date3.with(ChronoField.MONTH_OF_YEAR, 9);

//间接修改
LocalDate date1 = LocalDate.of(2014, 3, 18);
LocalDate date2 = date1.plusWeeks(1);
LocalDate date3 = date2.minusYears(3);
LocalDate date4 = date3.plus(6, ChronoUnit.MONTHS);

```
> 格式化日期 DateTimeFormatter 实例都是线程安全的。DateTimeFormatter 类还支持一个静态工厂方法，它可以按照某个特定的模式创建格式器

```java
LocalDate date = LocalDate.of(2014, 3, 18);
String s1 = date.format(DateTimeFormatter.BASIC_ISO_DATE);//20140318
String s2 = date.format(DateTimeFormatter.ISO_LOCAL_DATE);//2014-03-18
//这里不能HH:mm:dd 无法编译
String s3 = date.format(DateTimeFormatter.ofPattern("yyyy-MM-dd"));//2014-03-18


//按照特定模式创建格式器  
DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy/MM/dd");
LocalDate now = LocalDate.now();
String format = now.format(dateTimeFormatter);
LocalDate parse = LocalDate.parse(format,dateTimeFormatter);
//下面会报错,默认parse是yyyy-MM-dd格式
LocalDate parse = LocalDate.parse(format);

```


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/实战18.png)

### TemporalAdjusters 

```java
        LocalDate date = LocalDate.of(2021, 12, 30);
        //当前日期下一个周五如果 当前日期是周五就返回这个日期
        LocalDate with = date.with(TemporalAdjusters.nextOrSame(DayOfWeek.FRIDAY));
        //当前日期上一个周五如果 当前日期是周五就返回这个日期
        LocalDate with1 = date.with(TemporalAdjusters.previousOrSame(DayOfWeek.FRIDAY));
        //当前日期上一个周四 不论当前周几
        LocalDate with3 = date.with(TemporalAdjusters.previous(DayOfWeek.THURSDAY));
        //本月最后一天
        LocalDate with2 = date.with(TemporalAdjusters.lastDayOfMonth());

```
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/实战19.png)  

###  处理不同的时区和历法
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/实战20.png)
## 参考资料
> - 《Java 8 In Action》
> - optional部分参考[Optional指南(博客)](https://www.cnblogs.com/qing-gee/p/12453827.html)
