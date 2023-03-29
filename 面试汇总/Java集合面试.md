# 什么是集合
数据容器  set list map  queue

# 集合和数组区别 

数组 定长，存储基本数据类型或者引用类型   
集合 变长，存储引用类型，同一个集合类型可能存储不同类型元素

# 常用集合类   
- Collection
  - List有序可重 可null
    - ArrayList  对象数组 
    - LinkedList 双向链表有序 
    - Vector 对象数组 线程安全
    - Stack 栈
  - Set 无序唯一 只有一个null
    - HashSet Hashmap实现
    - TreeSet
    - LinkedHashSet 继承自hashset 内部通过linkedhashmap实现
- Map 键值对 无序 key唯一 value 可重复 
  - HashMap key只有一个null value可null
  - TreeMap 不能有null键，可以有null值
  - HashTable 线程安全 key value 都不能为null
  - CurrentHashMap 线程安全  key value 都不能为null


# Collection 

## List  

### 迭代器Iterator   
- Iterator 接口提供遍历任何Collection接口

使用迭代器可以 一边遍历一边修改Collection 

### Iterator 和ListIterator
iterator 可以遍历set和list集合 ，只能单向遍历   
ListIterator 只能遍历List  可以双向遍历，多了`hasPrevious` `previous`方法   


### Random Access 和随机访问  
实现randomAccess接口 说明支持高效的随机访问    
arraylist就实现了这个接口  
LinkedList 虽然没有实现这个接口，但是也有get方法 只不过这个方法是采用遍历获取指定值，使用简单二分查找优化（使用index 和列表size比较 大于size的一半就从后面倒序遍历，小于size的一半就从开始遍历搜索）  


### ArrayList 优缺点  
优点： 以数组实现，提供随机访问，**顺序**添加元素 获取元素高效    
缺点：删除元素，中间插入元素 需要元素复制操作`System.arraycopy`，耗费性能


### List 和数组之间转换   
list.toArray()     
Arrays.asList()  


### Array List和linkedlist区别
- 数据结构实现 对象数组  和双向循环链表
- 随机访问效率差距
- 增删效率
- 内存空间占用 arrayList有空间浪费 linked占用更多，因为每个节点都需要prev 和next区域   

### ArrayList 和Vector 
Vector 采用Synncronized 来实现同步，线程安全   
ArrayList 没有同步性能更佳   
扩容：ArrayList 扩容是1.5倍扩容，也就是增加50%；Vector 扩容是2倍扩容，也就是增加100%


### 为什么 ArrayList elementData加上transient　修饰　　
arraylist 实现了 Serializable接口 说明arraylist支持序列化 transient不希望elementData序列化    

每次序列化时，先调用 defaultWriteObject 序列化Arraylist 中的非transient元素，加快了序列化速度，减少了序列化之后的文件大小  

## List 和Set的区别   
list : 有序 可重复 可null多个
set ： 无序 不可重复  只有一个null   
treeset : 有序，不可重复 ；此处的有序和list中的有序不同   
treeset 底层通过treemap实现是红黑树，其中对象必须实现comparable 接口 实现对元素的排序    
list 的顺序是加入list 的顺序   


## Set接口   

### HashSet 实现原理  
HashSet是基于HashMap实现 HashSet的值是存放在HashMap的key上的，   
hashMap的value是`private static final Object PRESENT = new Object();`


### Hashset唯一  
hashset是基于hashmap的， hashmap的key重复会覆盖，所以hashset是唯一的




# Map

## Hash 算法
hashcode就是在hash表中有对应的位置 由hash算法得出  

## 链表 
增删快，内存利用效率高，大小不固定，扩展灵活  

查找效率低，实际是遍历查找


## Hashmap  

1.8 之前 hashmap 由数组`Node<K,V>`+链表实现，数组是主题，链表主要是为了解决hash冲突(拉链法)  

1.8之后为了解决拉链法 hash 冲突导致 链表过长读取性能下降，在链表长度大于8时，并且数组长度大于64，将链表替换为红黑树，以缩短搜索时间   

**链表长度大于8 并且数组长度大于64 才会转为红黑树**   
链表长度大于8 数组长度小于64 会先对数组进行扩容 而不会转为红黑树   

红黑树是一种平衡二叉树（平衡的二叉搜索树）  增删改查的时间复杂度都是O(log2N ) 也就是O(logn)，红黑树不追求绝对平衡，只需保证最长路径不超过最短路径的2倍

阈值是8 出于空间和时间的考量，小于8 的时候红黑树的优势不明显

回转链表的时候阈值是6，也就是当红黑树节点降低为6时，红黑树节点转为链表节点

## 负载因子为什么是 0.75
这是空间时间的权衡结果，  扩容值 = 容量 * 负载因子   

## hashmap原理  
hashmap是基于哈希表的的map接口非同步实现   
内部是由数组+链表实现   

1. put时利用key的hashcode计算当前元素在数组中的下标   
2. 放入值进行比较 若value相同，覆盖，若value不同 放入链表
3. 获取值根绝hashcode找到下标，获取，判断value是否相同，从而找到具体值

## 哈希冲突解决
1. 拉链法
2. 再哈希法 通过调用不同的hash方法
3. 开放地址法 再散列 通过两次调用hash

## hashmap扩容
扩容为原来的两倍，长度是2的幂次方

## 使用String Integer 作为Key
官方内部重写了 hashcode和equeals方法 不容易出现hash问题   
保证了key的数据类型不可变   

## 使用Objcet作为key的注意事项
重写hashcode 和equals方法

## hashMap和hashtable的区别
- 线程安全
- 效率
- 对null key hashtable不行 
- hashtable 初始11 扩容2n+1 hashmap 初始16扩容2倍
- hashmap的数据结构是 数组+链表+红黑树 
- hashtable 数据结构和1.8之前的hashmap相同 即数组+链表  

## Treemap
有序 kv 集合 通过红黑树实现  
根据键的自然顺序排序，或者根据映射提供的Comparator排序   

## hashmap 和 currenthashmap
currenthashmap 对桶数组进行了分割分段，使用分段锁相对于hashtable的全表锁粒度更细，并发性能更好    

JDK1.7版本的ReentrantLock(Segment)+Segment+HashEntry    

在1.8之后currenthashmap启用了synchronized+CAS+HashEntry+红黑树 弃用了分段锁机制

##  CurrentHashMap具体实现  
数据分段存储，每段数据上有锁，一个线程占用锁访问其中一个段数据时，其他段的数据也能被其他线程访问    

**JDK1.7中CurrentHashMap 采用 Segment+HashEntry 实现**

一个CurrentHashMap里包含一个Segment数组，Segment结构和HashMap类似，是一种数组+链表的结构 

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/currenthashmap内部实现.png)

**JDK1.8中采用 Node+CAS+Synchroized**

详解待查



# Collections 和辅助工具类  

## Array 和 ArrayList的区别 
Array可以存储基本类型和对象 ArrayList只能存储对象   

## Comparable 和Comparator 的区别  
comparable 接口 来自Java.lang包  通过CompareTo方法进行排序     
comparator 接口 来自Java.util包 通过compare方法用来排序

一般实现排序的方式是 继承comparable 重写 compareTo方法    
或者使用`Collections.sort(List<T> list, Comparator<? super T> c)`来实现  

## Collection 和Collections 
Collection 是一个集合接口  
Collections 是集合工具类   

## TreeMap 和 TreeSet  

TreeSet底层是TreeMap实现  
TreeMap 要求放入的对象必须实现Comparable 接口，从而实现对元素的排序   


## 确保集合不被修改  
`Collections.unmodifiableCollection(list);`

## 将非安全集合变为安全
注意是返回值接收的变量才是对应效果的集合   

`Collections.unmodifiableCollection(list)`     
`Collections.synchronizedCollection​(Collection<T> c)`	  
`CollectionssynchronizedList​(List<T> list)`   
`Collections.synchronizedMap​()`    

```java
List<Object> objects1 = Collections.synchronizedList(objects);
```














