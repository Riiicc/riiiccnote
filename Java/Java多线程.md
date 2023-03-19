

# 待办
- FutureTask能够在高并发环境下确保任务只执行一次，怎么确保的
- SecurityManager 在classloader 和 threadgroup中都有 Java的安全管理器  
- 重量级锁的理解  
- 《JVM 3 12.5》中关于协程 纤程的讲解收录入第一节  
- threadlocal 源码分析不完善  
- LongAdder 源码分析不完善
- StampedLock实现



# 基础篇
## 进程、线程、管程、的基本概念

`进程`就是应用程序在内存中分配的空间，也就是**正在运行的程序**，**各个进程之间互不干扰**。同时进程保存着程序每一个时刻运行的状态

CPU采用时间片轮转的方式运行进程：CPU为每个进程分配一个时间段，称作它的时间片。如果在时间片结束时进程还在运行，则暂停这个进程的运行，并且CPU分配给另一个进程（这个过程叫做上`下文切换`）。如果进程在时间片结束前阻塞或结束，则CPU立即进行切换，不用等待时间片用完   

`上下文切换`（有时也称做进程切换或任务切换）是指 CPU 从一个进程（或线程）切换到另一个进程（或线程）。上下文是指某一时间点 CPU 寄存器和程序计数器的内容。  

当进程暂停时，它会保存当前进程的状态（进程标识，进程使用的资源等），在下一次切换回来时根据之前保存的状态进行恢复，接着继续执行    

`线程`,轻量级进程，一个进程可能包含了多个线程，每个线程负责一个单独的子任务。     
**进程让操作系统的并发性成为了可能，而线程让进程的内部并发成为了可能。** 

> **多进程的方式也可以实现并发，为什么我们要使用多线程？**    
> 多进程方式确实可以实现并发，但使用多线程，有以下几个好处：    
> **进程间的通信比较复杂，而线程间的通信比较简单**，通常情况下，我们需要使用共享资源，这些资源在线程间的通信比较容易。   
> **进程是重量级的，而线程是轻量级的，故多线程方式的系统开销更小。**  

**进程和线程的区别**   
进程是一个独立的运行环境，而线程是在进程中执行的一个任务。他们两个**本质的区别是是否单独占有内存地址空间及其它系统资源（比如I/O）**   
- 进程单独占有一定的内存地址空间，所以进程间存在内存隔离，数据是分开的，数据共享复杂但是同步简单，各个进程之间互不干扰；而线程共享所属进程占有的内存地址空间和资源，数据共享简单，但是同步复杂。    
- 进程单独占有一定的内存地址空间，一个进程出现问题不会影响其他进程，不影响主程序的稳定性，可靠性高；一个线程崩溃可能影响整个程序的稳定性，可靠性较低。   
- 进程的创建和销毁不仅需要保存寄存器和栈信息，还需要资源的分配回收以及页调度，开销较大；线程只需要保存寄存器和栈信息，开销较小。    

另外一个重要区别是，**进程是操作系统进行资源分配的基本单位，而线程是操作系统进行调度的基本单位**，即CPU分配时间的单位 。   

> `管程`  
> Java虚拟机可以支持方法级的同步和方法内部一段指令序列的同步，这两种同步结构都是使用管程（Monitor，更常见的是直接将它称为“锁”）来实现的《深入理解java虚拟机-同步指令》 

## 用户线程和守护线程 

Java有两种线程，一种是`用户线程`，一种是`守护线程`。  
一般情况下不做特别说明配置，默认都是用户线程。它是系统的工作线程，它会完成这个程序需要完成的业务操作。

`thread.setDaemon(true)`可以把一个用户线程变成一个守护线程。

**守护线程主要是用作程序后台调度以及支持性工作**。为其它线程服务的，在后台默默地完成一些系统性的服务，比如垃圾回收线程。    
守护线程作为一个服务线程，没有服务对象就没有必要继续运行了，如果用户线程全部结束了，意味着程序需要完成的业务操作已经结束了，系统可以退出了。所以假如当系统只剩下守护线程的时候，java虚拟机会自动退出。   

**Daemon属性需要在启动线程之前设置**，不能在启动线程之后设置。Java垃圾回收就是一个典型的守护线程,若JVM中都是守护线程，当前JVM将退出。

## 多线程入门类和接口
JDK提供了Thread类和Runnalble接口来让我们实现自己的“线程”类。   
- 继承Thread类，并重写run方法；
- 实现Runnable接口的run方法；


### 继承Thread类
```java
public class Demo {
    public static class MyThread extends Thread {
        @Override
        public void run() {
            System.out.println("MyThread");
        }
    }

    public static void main(String[] args) {
        Thread myThread = new MyThread();
        myThread.start();
    }
}
```

> 注意要调用`start()`方法后，该线程才算启动！   
> 我们在程序里面调用了`start()`方法后，**虚拟机会先为我们创建一个线程，然后等到这个线程第一次得到时间片时再调用`run()`方法**。  
> 注意不可多次调用`start()`方法。在第一次调用`start()`方法后，再次调用`start()`方法会抛出异常。  

### 实现Runnable接口

```java
public class Demo {
    public static class MyThread implements Runnable {
        @Override
        public void run() {
            System.out.println("MyThread");
        }
    }

    public static void main(String[] args) {
        new MyThread().start();

        // Java 8 函数式编程，可以省略MyThread类
        new Thread(() -> {
            System.out.println("Java 8 匿名内部类");
        }).start();
        //上面的方法等同于
        new Thread(new Runnable(){
            @Override
            public void run() {
                
            }
        }).start();
    }
}
```

实际情况下，我们大多是直接调用下面两个构造方法：

```java
Thread(Runnable target)
Thread(Runnable target, String name)
```

### Thread类的常用方法
这里介绍一下Thread类的几个常用的方法：
- `currentThread()`：静态方法，返回对当前正在执行的线程对象的引用；
- `start()`：开始执行线程的方法，java虚拟机会调用线程内的run()方法；
- `yield()`：yield在英语里有放弃的意思，同样，这里的yield()指的是当前线程愿意让出对当前处理器的占用。这里需要注意的是，就算当前线程调用了yield()方法，程序在调度的时-候，也还有可能继续运行这个线程的；
- `sleep()`：静态方法，使当前线程睡眠一段时间；
- `join()`：使当前线程等待另一个线程执行完毕之后再继续执行，内部调用的是Object类的wait方法实现的；

 ### Thread和Runnable 接口比较 

实现一个自定义的线程类，可以有**继承Thread类**或者**实现Runnable接口**这两种方式，它们之间有什么优劣呢？     

- 由于Java“单继承，多实现”的特性，Runnable接口使用起来比Thread更灵活。
- Runnable接口出现更符合面向对象，将线程单独进行对象的封装。
- Runnable接口出现，降低了线程对象和线程任务的耦合性。
- 如果使用线程时不需要使用Thread类的诸多方法，显然使用Runnable接口更为轻量。

> 所以，我们通常优先使用`实现Runnable接口`这种方式来自定义线程类

### Callable

Callable与Runnable类似，同样是只有一个抽象方法的函数式接口。不同的是，**Callable提供的方法是有返回值的，而且支持泛型**。

```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

> `Callable`一般是配合线程池工具`ExecutorService`来使用的。我们会在后续章节解释线程池的使用。    
> 这里只介绍`ExecutorService`可以使用`submit`方法来让一个`Callable`接口执行。它会返回一个`Future`，我们后续的程序可以**通过这个`Future`的`get`方法得到结果**  
> **get方法会阻塞当前线程，直到得到结果，所以实际编码中建议使用可以设置超时时间的重载get方法，如果超时会抛出`TimeoutException`**

```java
// 自定义Callable
class Task implements Callable<Integer>{
    @Override
    public Integer call() throws Exception {
        // 模拟计算需要一秒
        Thread.sleep(1000);
        return 2;
    }
     public static void main(String args[]) throws ExecutionException, InterruptedException {
        // 使用
        ExecutorService executor = Executors.newCachedThreadPool();
        Task task = new Task();
        Future<Integer> result = executor.submit(task);
        // 注意调用get方法会阻塞当前线程，直到得到结果。
        System.out.println(result.get()); 
        // 所以实际编码中建议使用可以设置超时时间的重载get方法。如下,100秒超时时间,如果超时会抛出TimeoutException
        System.out.println(result.get(100,TimeUnit.SECONDS));
    }
}
//输出结果2
```

### Future、FutureTask

####  Future接口
`Future`定义了**操作异步任务执行**的一些方法   
`Future`是Java5新加的一个接口，它提供了一种异步并行计算的功能。    
如果主线程需要执行一个很耗时的计算任务，我们就可以通过`Future`把这个任务放到异步线程中执行。主线程继续处理其他任务或者先行结束，再通过`Future`获取计算结果   
`Future`接口只有几个比较简单的方法：   

```java
public abstract interface Future<V> {
    public abstract boolean cancel(boolean paramBoolean);
    public abstract boolean isCancelled();
    public abstract boolean isDone();
    public abstract V get() throws InterruptedException, ExecutionException;
    public abstract V get(long paramLong, TimeUnit paramTimeUnit)
            throws InterruptedException, ExecutionException, TimeoutException;
}
```  

> `cancel`方法是**试图取消**一个线程的执行。  
> 注意是试图取消，并不一定能取消成功。因为任务可能已完成、已取消、或者一些其它因素不能取消，存在取消失败的可能。`boolean`类型的返回值是**是否取消成功**的意思。参数`paramBoolean`表示是否采用中断的方式取消线程执行。

?> 所以有时候，为了**让任务有能够取消的功能**，就使用`Callable`来代替`Runnable`。如果为了可取消性而使用 `Future`但又不提供可用的结果，则可以声明 `Future<?>`形式类型、并返回 `null`作为底层任务的结果

#### FutureTask类
`FutureTask`是实现的`RunnableFuture`接口的，而`RunnableFuture`接口同时继承了`Runnable`接口和`Future`接口：

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/futuretask的继承关系.png)

**那FutureTask类有什么用？为什么要有一个FutureTask类？**   
`Future`只是一个接口，而它里面的`cancel，get，isDone`等方法要**自己实现起来都是非常复杂**的。所以JDK提供了一个`FutureTask`类来供我们使用

```java
// 自定义Callable，与上面一样
class Task implements Callable<Integer>{
    @Override
    public Integer call() throws Exception {
        // 模拟计算需要一秒
        Thread.sleep(1000);
        return 2;
    }
    public static void main(String args[]){
        // 使用1 线程池
        ExecutorService executor = Executors.newCachedThreadPool();
        FutureTask<Integer> futureTask = new FutureTask<>(new Task());
        executor.submit(futureTask);
        System.out.println(futureTask.get());
    }

    public static void main2(String args[]){
        // 使用2
        FutureTask<Integer> futureTask = new FutureTask<>(new Task());
        Thread t = new Thread(futureTask,"t1");
        t.start();
        System.out.println(futureTask.get());
    }
}
```

> 使用上与第一个Demo有一点小的区别。首先，调用submit方法是没有返回值的。这里实际上是调用的submit(Runnable task)方法，而上面的Demo，调用的是submit(Callable<T> task)方法。
> 然后，这里是使用FutureTask直接取get取值，而上面的Demo是通过submit方法返回的Future去取值。
> 在很多高并发的环境下，有可能Callable和FutureTask会创建多次。**FutureTask能够在高并发环境下确保任务只执行一次。参看FutureTask源码**。  

**FutureTask的几个状态**   

```java
/**
  *
  * state可能的状态转变路径如下：
  * NEW -> COMPLETING -> NORMAL
  * NEW -> COMPLETING -> EXCEPTIONAL
  * NEW -> CANCELLED
  * NEW -> INTERRUPTING -> INTERRUPTED
  */
private volatile int state;
private static final int NEW          = 0;
private static final int COMPLETING   = 1;
private static final int NORMAL       = 2;
private static final int EXCEPTIONAL  = 3;
private static final int CANCELLED    = 4;
private static final int INTERRUPTING = 5;
private static final int INTERRUPTED  = 6;
```

> state表示任务的运行状态，初始状态为NEW。运行状态只会在set、setException、cancel方法中终止。COMPLETING、INTERRUPTING是任务完成后的瞬时状态

#### Future优缺点分析

**优点**  
- 结合线程池Future的异步多线程任务能显著提高程序执行效率

**缺点**    
- `get()`方法容易导致阻塞，调用get方法会导致主程序等待返回，导致get后面的代码延迟执行，一般放在代码最后   
  - `get(long timeout,TimeUnit tn)`设置 get方法超时时间  
- 判定是否返回需要使用`isDone()`轮询获取状态，耗费资源  

跳转[CompletableFuture](/Java/Java多线程?id=Completablefuture)


## 线程组和线程优先级  

### 线程组ThreadGroup  
- **每一个`Thread`必然存在于一个`ThreadGroup`中,`Thread` 不能独立于`ThreadGroup`存在**
- 如果在`new Thread`时没有显式指定，那么默认将父线程（当前执行new Thread的线程）线程组设置为自己的线程组 
- `ThreadGroup`管理下属`Thread`，`TreadGroup`是一个标准的向下引用树状结构，防止**上级线程被下级线程引用而无法有效的被GC回收** 

```java
public class Demo {
    public static void main(String[] args) {
        Thread testThread = new Thread(() -> {
            System.out.println("testThread当前线程组名字：" +
                    Thread.currentThread().getThreadGroup().getName());
            System.out.println("testThread线程名字：" +
                    Thread.currentThread().getName());
        });

        testThread.start();
        System.out.println("执行main方法线程名字：" + Thread.currentThread().getName());
    }
}
//执行main方法线程名字：main
// testThread当前线程组名字：main
// testThread线程名字：Thread-0
```

### 线程优先级
- 优先级范围1-10
- 默认优先级为5(NORM_PRIORITY),最小为1(MIN_PRIORITY), 最大值为10(MAX_PRIORITY)  
- 更高优先级比低优先级有**更高几率**得到执行,优先级高度依赖于系统  
- Java优先级不可靠,真正的调用顺序由**操作系统的线程调度算法决定**  
- 如果某个线程优先级大于线程所在线程组的最大优先级，那么该线程的优先级将会失效，取而代之的是线程组的最大优先级  

使用Thread类的setPriority()实例方法来设定线程的优先级   

```java
public class Demo {
    public static void main(String[] args) {
        Thread a = new Thread();
        System.out.println("我是默认线程优先级："+a.getPriority());
        Thread b = new Thread();
        b.setPriority(10);
        System.out.println("我是设置过的线程优先级："+b.getPriority());
    }
}
// 我是默认线程优先级：5
// 我是设置过的线程优先级：10
```

!> **Java程序中对线程所设置的优先级只是给操作系统一个建议，操作系统不一定会采纳。而真正的调用顺序，是由操作系统的线程调度算法决定的**

### 线程组的常用方法及数据结构
```java
//获取当前线程名称
Thread.currentThread().getThreadGroup().getName()

// 复制一个线程数组到一个线程组
Thread[] threads = new Thread[threadGroup.activeCount()];
TheadGroup threadGroup = new ThreadGroup();
threadGroup.enumerate(threads);

//  线程组统一异常处理  uncaughtException 只针对线程组
public class ThreadGroupDemo {
    public static void main(String[] args) {
        ThreadGroup threadGroup1 = new ThreadGroup("group1") {
            // 继承ThreadGroup并重新定义以下方法
            // 在线程成员抛出unchecked exception
            // 会执行此方法
            public void uncaughtException(Thread t, Throwable e) {
                System.out.println(t.getName() + ": " + e.getMessage());
            }
        };

        // 这个线程是threadGroup1的一员
        Thread thread1 = new Thread(threadGroup1, new Runnable() {
            public void run() {
                // 抛出unchecked异常
                throw new RuntimeException("测试异常");
            }
        });

        thread1.start();
    }
}

```
> 线程得统一异常处理除了上面的uncaughtException ,还可以用**setUncaughtExceptionHandler**方法为任意线程安装一个处理器,
> 也可以用静态方法**setDefaultUncaughtExceptionHandler**为所有线程安装一个默认处理器  


```java
// setUncaughtExceptionHandler
public class Demo {
    public static void main(String[] args) {
        Thread thread = new Thread(){
            @Override
            public void run() {
                throw new RuntimeException("123123");
            }

        };
        thread.setUncaughtExceptionHandler((Thread t,Throwable e)->{
            System.out.println(t.getName() + ": " + e.getMessage());
        });
        thread.start();
    }
    // 输出  Thread-0: 123123
}
```  

```java
// setDefaultUncaughtExceptionHandler
public class Demo {
    public static void main(String[] args) {
        Thread thread = new Thread(){
            @Override
            public void run() {
                throw new RuntimeException("123123");
            }
        };
        Thread thread1 = new Thread(){
            @Override
            public void run() {
                throw new RuntimeException("xxxxx");
            }
        };
        Thread.setDefaultUncaughtExceptionHandler((Thread t,Throwable e)->{
            System.out.println(t.getName() + ": " + e.getMessage());
        });
        thread.start();
        thread1.start();
    }
    // 输出  Thread-0: 123123
    //      Thread-1: xxxxx
}
```

###  线程组的数据结构
**线程组源码解析**  
- 线程组是一个树状的结构，每个线程组下面可以有多个线程或者线程组。线程组可以起到统一控制线程的优先级和检查线程的权限的作用  

```java
public class ThreadGroup implements Thread.UncaughtExceptionHandler {
    private final ThreadGroup parent; // 父亲ThreadGroup
    String name; // ThreadGroupr 的名称
    int maxPriority; // 线程最大优先级
    boolean destroyed; // 是否被销毁
    boolean daemon; // 是否守护线程
    boolean vmAllowSuspension; // 是否可以中断

    int nUnstartedThreads = 0; // 还未启动的线程
    int nthreads; // ThreadGroup中线程数目
    Thread threads[]; // ThreadGroup中的线程

    int ngroups; // 线程组数目
    ThreadGroup groups[]; // 线程组数组
}
```


```java
// 私有构造函数
private ThreadGroup() { 
    this.name = "system";
    this.maxPriority = Thread.MAX_PRIORITY;
    this.parent = null;
}

// 默认是以当前ThreadGroup传入作为parent  ThreadGroup，新线程组的父线程组是目前正在运行线程的线程组。
public ThreadGroup(String name) {
    this(Thread.currentThread().getThreadGroup(), name);
}

// 构造函数
public ThreadGroup(ThreadGroup parent, String name) {
    this(checkParentAccess(parent), parent, name);
}

// 私有构造函数，主要的构造函数
private ThreadGroup(Void unused, ThreadGroup parent, String name) {
    this.name = name;
    this.maxPriority = parent.maxPriority;
    this.daemon = parent.daemon;
    this.vmAllowSuspension = parent.vmAllowSuspension;
    this.parent = parent;
    parent.add(this);
}
```  

```java
// 检查parent ThreadGroup
private static Void checkParentAccess(ThreadGroup parent) {
    parent.checkAccess();
    return null;
}

// 判断当前运行的线程是否具有修改线程组的权限
public final void checkAccess() {
    SecurityManager security = System.getSecurityManager();
    if (security != null) {
        security.checkAccess(this);
    }
}
```


## Java线程的状态及主要转化方法
> 线程可以看作是轻量级进程 **操作系统线程的状态其实和操作系统进程的状态是一致的**

操作系统线程主要有以下三个状态
- 就绪状态 `Ready` 线程正在等待使用CPU，经调度程序调用之后可进入running状态
- 执行状态 `Running`线程正在使用CPU。
- 等待状态 `Waiting`线程经过等待事件的调用或者正在等待其他资源（如I/O）。  

### Java线程的6个状态   

```java
// Thread.State 源码
public enum State {
    NEW,
    RUNNABLE,
    BLOCKED,
    WAITING,
    TIMED_WAITING,
    TERMINATED;
}
```

#### NEW
> NEW状态的线程此时尚未启动,**并未调用线程的start方法**

1. 重复调用start方法会报错 `IllegalThreadStateException`
2. 一个线程执行完毕(此时处于TERMINATED状态),再次调用会抛出同样的错误 
3. 在调用一次start()之后，`threadStatus`的值会改变`(threadStatus !=0) = true`

```java
// start()的源码：
public synchronized void start() {
    //TERMINATED threadStatus ==2 
    if (threadStatus != 0)
        throw new IllegalThreadStateException();

    group.add(this);

    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {

        }
    }
}
```


#### RUNNABLE
> 表示当前线程正在运行中。处于RUNNABLE状态的线程在Java虚拟机中运行，也有可能在等待其他系统资源（比如I/O）。
> Java线程的**RUNNABLE**状态其实是包括了传统操作系统线程的**ready和running**两个状态的。  

#### BLOCKED
阻塞状态。处于`BLOCKED`状态的线程正**等待锁的释放**以进入同步区。  


#### WAITING 
等待状态, 处于等待状态的线程变成RUNNABLE状态需要其他线程唤醒  
调用下面3个方法会使线程进入等待状态
1. `Object.wait()` 
2. `Thread.join()` 底层为wait方法
3. `LockSupport.part()` 除非获得调用许可,否则禁用当前线程进行线程调度  

#### TIMED_WAITING
计时(超时) 等待 ,线程等待一个具体时间,时间到后会被自动唤醒.

1. `Thread.sleep(long millis)`：使当前线程睡眠指定时间；
2. `Object.wait(long timeout)`：线程休眠指定时间，等待期间可以通过notify()/notifyAll()唤醒；
3. `Thread.join(long millis)`：等待当前线程最多执行millis毫秒，如果millis为0，则会一直执行；
4. `LockSupport.parkNanos(long nanos)`： 除非获得调用许可，否则禁用当前线程进行线程调度指定时间；
5. `LockSupport.parkUntil(long deadline)`：同上，也是禁止线程进行调度指定时间；

### 线程状态的转换
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/基础12.png) 

##### BLOCKED与RUNNABLE状态的转换

处于BLOCKED状态的线程是因为在等待锁的释放。假如这里有两个线程a和b，a线程提前获得了锁并且暂未释放锁，此时b就处于BLOCKED状态。

```java
@Test
public void blockedTest() {

    Thread a = new Thread(new Runnable() {
        @Override
        public void run() {
            testMethod();
        }
    }, "a");
    Thread b = new Thread(new Runnable() {
        @Override
        public void run() {
            testMethod();
        }
    }, "b");

    a.start();
    b.start();
    System.out.println(a.getName() + ":" + a.getState()); // 输出？
    System.out.println(b.getName() + ":" + b.getState()); // 输出？
}

// 同步方法争夺锁
private synchronized void testMethod() {
    try {
        Thread.sleep(2000L);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

上述代码输出是？   
可能会觉得线程a会先调用同步方法，同步方法内又调用了`Thread.sleep()`方法，必然会输出`TIMED_WAITING`，而线程b因为等待线程a释放锁所以必然会输出`BLOCKED`  

其实不然，有两点需要注意:  
一是在测试方法`blockedTest()`内还有一个`main线程`，   
二是启动线程后执行run方法还是需要消耗一定时间的。**不打断点的情况下，上面代码中都应该输出`RUNNABLE`**

要想输出原期望值 即 `TIMED_WAITING` 和`BLOCKED` 就需要在main调用ab线程中进行暂停，并且这个暂停时间不能超过a等待时间  

```java
public void blockedTest() throws InterruptedException {
    ······
    a.start();
    Thread.sleep(1000L); // 需要注意这里main线程休眠了1000毫秒，而testMethod()里休眠了2000毫秒
    b.start();
    System.out.println(a.getName() + ":" + a.getState()); // 输出 TIMED_WAITING
    System.out.println(b.getName() + ":" + b.getState()); // 输出 BLOCKED
}
```

#### WAITING状态与RUNNABLE状态的转换  
**Object.wait()**   
> 调用wait方法前线程必须持有对象锁   
> 调用wait方法会释放当前锁,直到有其他线程调用`notify()`或者`notifyAll()`方法唤醒等待锁的线程   
> `notify` 和`notifyAll` 方法不一定会唤醒之前调用`wait`方法的线程, `notifyAll` 唤醒所有等待线程,也不一定会马上执行 之前的线程,具体调用顺序依赖系统调度   

**Thread.join()**  
> 调用join方法不会释放锁,会一直等待当前线程执行完毕(TERMINATED)

```java
public void blockedTest() {
    ······
    a.start();
    a.join();//等待a执行完毕 才会继续执行
    b.start();
    System.out.println(a.getName() + ":" + a.getState()); // 输出 TERMINATED
    System.out.println(b.getName() + ":" + b.getState());
}
```

> 要是没有调用join方法，**main线程不管a线程是否执行完毕都会继续往下走**。    
> a线程启动之后马上调用了join方法，这里**main线程就会等到a线程执行完毕**，所以这里a线程打印的状态固定是TERMIATED。   
> 至于b线程的状态，有可能打印`RUNNABLE`（尚未进入同步方法），也有可能打印`TIMED_WAITING`（进入了同步方法）。   

#### TIMED_WAITING与RUNNABLE状态转换
- `TIMED_WAITING`与`WAITING`状态类似，只是TIMED_WAITING状态等待的时间是指定的。  

1. `Thread.sleep(long)`使当前线程睡眠指定时间。需要注意这里的“睡眠”只是暂时使线程停止执行，并不会释放锁。时间到后，线程会重新进入`RUNNABLE`状态。
2. `Object.wait(long)`

> wait(long)方法使线程进入TIMED_WAITING状态。这里的wait(long)方法与无参方法wait()相同的地方是，都可以通过其他线程调用notify()或notifyAll()方法来唤醒。
> 不同的地方是，有参方法wait(long)就算其他线程不来唤醒它，经过指定时间long之后它会自动唤醒，拥有去争夺锁的资格。  

3. `Thread.join(long)`调用`a.join(1000L)`，main线程等待a执行1s，之后继续执行，因为是指定了具体a线程执行的时间的，并且执行时间是小于a线程sleep的时间，所以a线程状态输出`TIMED_WAITING`,b线程状态仍然不固定（RUNNABLE或BLOCKED）

### 线程中断
在Java中没有办法立即停止一条线程，然而停止线程却显得尤为重要，如取消一个耗时操作。因此，Java提供了一种用于停止线程的协商机制―—中断，也即`中断标识协商机制`

中断只是一种协作协商机制，Java没有给中断增加任何语法，中断的过程完全需要程序员自己实现。   
若要中断一个线程，你需要手动调用该线程的interrupt方法，该方法也仅仅是将线程对象的中断标识设成true;接着你需要自己写代码不断地检测当前线程的标识位，如果为true，表示别的线程请求这条线程中断，此时究竟该做什么需要你自己写代码实现。  

- `interrupt()` 中断线程,不会立刻终止线程,而是将线程的中断状态设置为true(默认为false),  
  - 如果该线程被sleep,wait等调用阻塞,调用该方法会抛出`InterruptedException`,中断状态同时被重置，导致线程继续执行，此时需要再catch块中调用`Thread.currentThread().interrupt();`
- `Thread.interrupted()` 静态方法,**测试当前线程是否被中断,重置线程中断状态**,调用一次使线程中断状态设置为**false**(重置中断状态)
- `isInterrupted()` 测试当前线程是否被中断。调用这个方法并不会影响线程的中断状态。

```java
//第一个
    public void interrupt() {
        if (this != Thread.currentThread())
            checkAccess();

        synchronized (blockerLock) {
            Interruptible b = blocker;
            if (b != null) {
                interrupt0();           // Just to set the interrupt flag
                b.interrupt(this);
                return;
            }
        }
        interrupt0();
    }
    private native void interrupt0();

//第2个
public static boolean interrupted() {
        return currentThread().isInterrupted(true);
}

//第3个
public boolean isInterrupted() {
        return isInterrupted(false);
}

//上面两个方法最终调用的同一个本地方法,但是传入的参数不同 
//ClearInterrupted 的值决定状态是否重置  不是设置状态值 
private native boolean isInterrupted(boolean ClearInterrupted);
```



## Lock Support
`LockSupport`位于 `java.util.concurrent.locks.LockSupport`用于创建锁和其他同步类的基本线程阻塞原语   
`LockSupport`是一个线程阻塞的工具类，所有的方法都是**静态方法**，可以让线程再任意位置阻塞，阻塞之后也有对应的唤醒方法，底层调用的是Unsafe类中的native方法 `UNSAFE.park(false, 0L);` 

LockSupport提供park()和unpark()方法实现阻塞线程和解除线程阻塞的过程LockSupport和每个使用它的线程都有一个许可(permit)关联。每个线程都有一个相关的permit, **permit最多只有一个，重复调用unpark也不会积累凭证**。

当调用`unpark(Tharead)`方法时如果有凭证，则会直接消耗掉这个凭证然后正常退出;  
如果无凭证，就必须阻塞等待凭证可用;而`park`则相反，它会增加一个凭证，但**凭证最多只能有1个，累加无效**

### 线程等待唤醒机制
- 使用Object中的wait()方法让线程等待，使用notify()唤醒  
  - wait 和notify必须在同步代码块或同步方法里，且成对出现
  - 必须先wait 后notify才能有效，否则一直wait
- 使用JUC包中的Condition的await()方法等待，使用signal() 唤醒
  - Condition的等待和唤醒方法必须在锁块中`locak.lock();`
  - 必须先await，再signal(),否则一直wait
- 使用LockSupport 的park()进行等待，使用unpart()唤醒
  - 先进行unpark() 再park()依然可以唤醒，此时的park()没有阻塞的作用

```java
public class TestLock {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            System.out.println(Thread.currentThread().getName());
            LockSupport.park();
            System.out.println("唤醒");
        }, "t1");
        t1.start();
        TimeUnit.SECONDS.sleep(1);
        new Thread(()->{
            LockSupport.unpark(t1);
            System.out.println("发出通知");
        }).start();
    }
}
// t1
// 发出通知
// 唤醒
```


## 线程间通信

### 锁与同步 
在Java中，锁的概念都是基于对象的，所以我们又经常称它为对象锁。线程和锁的关系，一个锁同一时间只能被一个线程持有。

线程同步是线程之间按照一定的顺序执行。为了达到线程同步，我们可以使用锁来实现它  


```java
//使用对象锁实现a执行完 b再执行
public class ObjectLock {
    private static Object lock = new Object();

    static class ThreadA implements Runnable {
        @Override
        public void run() {
            synchronized (lock) {
                for (int i = 0; i < 100; i++) {
                    System.out.println("Thread A " + i);
                }
            }
        }
    }

    static class ThreadB implements Runnable {
        @Override
        public void run() {
            synchronized (lock) {
                for (int i = 0; i < 100; i++) {
                    System.out.println("Thread B " + i);
                }
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new Thread(new ThreadA()).start();
        Thread.sleep(10);
        new Thread(new ThreadB()).start();
    }
}
//输出结果 是 a0-a99  -> b0-b99
```

> 这里声明了一个名字为lock的对象锁。我们在ThreadA和ThreadB内需要同步的代码块里，都是用synchronized关键字加上了同一个对象锁lock。上文我们说到了，根据线程和锁的关系，同一时间只有一个线程持有一个锁，那么线程B就会等线程A执行完成后释放lock，线程B才能获得锁lock  
> sleep方法睡眠了10毫秒，是为了防止线程B先得到锁,否在可能先打印b



### 等待/通知机制
上面一种基于`锁`的方式，线程需要不断地去尝试获得锁，如果失败了，再继续尝试。这可能会耗费服务器资源。  

> 通过Object类的`wait()` `notify()` `notifyAll()` 方法来实现    
> `notify()`方法会随机叫醒一个正在等待的线程，而`notifyAll()`会叫醒所有正在等待的线程。

> 一个锁同一时刻只能被一个线程持有。而假如线程A现在持有了一个锁lock并开始执行，它可以使用`lock.wait()`让自己进入等待状态。这个时候，lock这个锁是被**释放**了的。     
> 这时，线程B获得了lock这个锁并开始执行，它可以在某一时刻，使用`lock.notify()`，通知之前持有lock锁并进入等待状态的线程A，让其继续执行。    

?> 需要注意的是，`lock.notify()`这个时候线程B并没有释放锁lock，**除非线程B这个时候使用lock.wait()释放锁，或者线程B执行结束自行释放锁，线程A才能得到lock锁**

```java
public class WaitAndNotify {
    private static Object lock = new Object();

    static class ThreadA implements Runnable {
        @Override
        public void run() {
            synchronized (lock) {
                for (int i = 0; i < 5; i++) {
                    try {
                        System.out.println("ThreadA: " + i);
                        lock.notify();
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                lock.notify();
            }
        }
    }

    static class ThreadB implements Runnable {
        @Override
        public void run() {
            synchronized (lock) {
                for (int i = 0; i < 5; i++) {
                    try {
                        System.out.println("ThreadB: " + i);
                        lock.notify();
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                lock.notify();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new Thread(new ThreadA()).start();
        Thread.sleep(1000);
        new Thread(new ThreadB()).start();
    }
}

// 输出：
// ThreadA: 0
// ThreadB: 0
// ThreadA: 1
// ThreadB: 1
// ThreadA: 2
// ThreadB: 2
// ThreadA: 3
// ThreadB: 3
// ThreadA: 4
// ThreadB: 4
```  

> 线程A和线程B首先打印出自己需要的东西，然后使用notify()方法叫醒另一个正在等待的线程，然后自己使用wait()方法陷入等待并释放lock锁  
> 等待/通知机制使用的是使用**同一个对象锁**，如果你两个线程使用的是不同的对象锁，那它们之间是不能用等待/通知机制通信的


### 信号量

JDK提供了一个类似于信号量功能的类`Semaphore`。但这里是**介绍一种基于volatile关键字的自己实现的信号量通信**

```java
public class Signal {
    private static volatile int signal = 0;

    static class ThreadA implements Runnable {
        @Override
        public void run() {
            while (signal < 5) {
                if (signal % 2 == 0) {
                    System.out.println("threadA: " + signal);
                    synchronized (this) {
                        signal++;
                    }
                }
            }
        }
    }

    static class ThreadB implements Runnable {
        @Override
        public void run() {
            while (signal < 5) {
                if (signal % 2 == 1) {
                    System.out.println("threadB: " + signal);
                    synchronized (this) {
                        signal = signal + 1;
                    }
                }
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new Thread(new ThreadA()).start();
        Thread.sleep(1000);
        new Thread(new ThreadB()).start();
    }
}

// 输出：
threadA: 0
threadB: 1
threadA: 2
threadB: 3
threadA: 4
```

使用了一个`volatile`变量`signal`来实现了“信号量”的模型。这里需要注意的是，`volatile`变量需要进行原子操作。`signal++`并不是一个原子操作，所以我们需要使用`synchronized`给它“上锁”

> 信号量的应用场景：  
> 假如在一个停车场中，车位是我们的公共资源，线程就如同车辆，而看门的管理员就是起的“信号量”的作用。  
> 因为在这种场景下，多个线程（超过2个）需要相互合作，我们用简单的“锁”和“等待通知机制”就不那么方便了。这个时候就可以用到信号量。保证进来的车都有车位，车位满了，后面的车只能等待  
> 其实JDK中提供的很多多线程通信工具类都是基于信号量模型的。我们会在后面第三篇的文章中介绍一些常用的通信工具类。 


### join方法
join()方法是Thread类的一个实例方法。它的作用是让**当前线程**陷入“等待”状态，等join的这个线程执行完成后，再继续执行当前线程。
> 有时候，主线程创建并启动了子线程，如果子线程中需要进行大量的耗时运算，**主线程往往将早于子线程结束之前结束**。要想主线程等待子线程执行完毕再继续执行,就需要用到join方法

> join方法 及其重载方法底层都利用了wait方法

```java
public class Join {
    static class ThreadA implements Runnable {

        @Override
        public void run() {
            try {
                System.out.println("我是子线程，我先睡一秒");
                Thread.sleep(1000);
                System.out.println("我是子线程，我睡完了一秒");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new ThreadA());
        thread.start();
        thread.join();
        System.out.println("如果不加join方法，我会先被打出来，加了就不一样了");
    }
}
```

> 注意`join()`方法有两个重载方法，一个是`join(long)`， 一个是`join(long, int)`。
> 实际上，通过源码你会发现，join()方法及其重载方法底层都是利用了`wait(long)`这个方法。
> 对于`join(long, int)`，通过查看源码(JDK 1.8)发现，底层并没有精确到纳秒，而是对第二个参数做了简单的判断和处理

### sleep方法
sleep 方法是Thread 类的**静态**方法 :
- `Thread.sleep(long)`
- `Thread.sleep(long,int)` 后面的int是nano纳秒数,但是并不精确到纳秒,第二个参数nanos大于0.5毫秒时或者小于1毫秒，进一位。

> **sleep方法不会释放当前锁**,只是释放`cpu`资源,而`wait`方法会释放锁  

- `sleep` 和 `wait` 的区别   
  - wait可以指定时间，也可以不指定；而sleep**必须指定时间**。
  - wait**释放cpu资源，同时释放锁**；sleep**释放cpu资源，但是不释放锁**，所以易死锁。
  - wait必须放在同步块或同步方法中，而sleep可以在任意位置。



# 原理篇

## 并发编程模型的两个关键问题

1. 线程间如何**通信**？ 即：线程之间以何种机制来交换信息
2. 线程间如何**同步**？ 即：线程以何种机制来控制不同线程间操作发生的相对顺序  


有**两种并发模型**可以解决这两个问题： 
- 消息传递并发模型
- 共享内存并发模型  


| 模型类型         | 通信方式                                                                  | 同步方式                                                               |
| ---------------- | ------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| 消息传递并发模型 | **线程之间没有公共状态**，线程间的通信必须通过发送消息来**显式**进行通信  | 发送消息天然同步，因为发送消息总是在接收消息之前，因此同步是**隐式**的 |
| 共享内存并发模型 | 线程之间共享程序的公共状态，通过**写-读内存中的公共状态**进行**隐式**通信 | **必须显式的指定某段代码需要在线程之间互斥执行，**同步是**显式**的     |

## Java 内存模型 

**Java中，使用的是`共享内存并发模型`。**

### 运行时内存的划分 
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/Java运行时数据区.png)

对于每一个线程来说，**栈都是私有的，而堆是共有的**。  

也就是说在栈中的变量(局部变量、方法定义参数、异常处理器参数)不会在线程之间共享，也就不会有内存可见性（下文会说到）的问题，也不受内存模型的影响。  
而在堆中的变量是共享的，本文称为共享变量。  

所以，**内存可见性是针对的共享变量**


> 线程之间的共享变量存在主内存中，每个线程都有一个私有的本地内存，存储了该线程已读、写共享变量的副本。
> 本地内存是Java内存模型的一个抽象概念，**并不真实存在**。它涵盖了缓存、写缓冲区、寄存器等。

> Java线程之间的通信由Java内存模型（简称JMM）控制，从抽象的角度来说，**JMM定义了线程和主内存之间的抽象关系**  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/JMM抽象示意图.jpg ':size=50%')

上图总结: 
- 所有的共享变量都保存在主内存中
- 每个线程都保存了一份该线程使用到的共享变量的副本
- 如果线程A和线程B之间进行通信的话,必须经过两个步骤:
  - 线程A 将本地内存的共享变量更新到主内存中
  - 线程B 读取主内存中的共享变量,保存到本地内存

**线程A无法直接访问线程B的工作内存,线程间通信必须经过主内存**    
JMM规定,**线程对共享变量的所有操作必须在自己的本地内存中进行,不能直接从主内存中操作**    

举例：线程B并不是直接去主内存中读取共享变量的值，而是先在本地内存B中找到这个共享变量，发现这个共享变量已经被更新了，然后本地内存B去主内存中读取这个共享变量的新值，并拷贝到本地内存B中，最后线程B再读取本地内存B中的新值

怎么知道这个共享变量的被其他线程更新了呢？   
这就是JMM的功劳了，也是JMM存在的必要性之一。**JMM通过控制主内存与每个线程的本地内存之间的交互，来提供内存可见性保证**。   
**JMM通过内存屏障来实现内存的可见性以及禁止重排序**。  

?> 为了程序员的方便理解，提出了`happens-before`，它更加的简单易懂

### JMM与Java内存区域划分的区别与联系

**区别**   
两者是不同的概念层次。JMM是抽象的，他是用来描述一组规则，通过这个规则来控制各个变量的访问方式，围绕原子性、有序性、可见性等展开的。而Java运行时内存的划分是具体的，是JVM运行Java程序时，必要的内存划分。

**联系**
都存在私有数据区域和共享数据区域。一般来说，**JMM中的主内存属于共享数据区域，他是包含了堆和方法区**；同样，**JMM中的本地内存属于私有数据区域，包含了程序计数器、本地方法栈、虚拟机栈**。

## 重排序

计算机在执行程序时，为了**提高性能，编译器和处理器常常会对指令做重排(重新排序以优化性能)**

```java
a = b + c;
d = e - f ;
```

先加载b、c（注意，即有可能先加载b，也有可能先加载c），但是在执行add(b,c)的时候，**需要等待b、c装载结束才能继续执行**，  
也就是增加了停顿，那么后面的指令也会依次有停顿,这降低了计算机的执行效率。

为了减少这个停顿，我们可以先加载e和f,然后再去加载add(b,c),这样做对程序（串行）是没有影响的,但却减少了停顿。既然add(b,c)需要停顿，那还不如去做一些有意义的事情。

**指令重排对于提高CPU处理性能十分必要。虽然由此带来了乱序的问题，但是这点牺牲是值得的**

指令重排一般分为一下三种:  
- 编译器优化重排
- 指令并行重排
- 内存系统重排

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/重排序的三种情况.png)

指令重排可以保证串行语义一致，**但是没有义务保证多线程间的语义也一致**。所以在多线程下，**指令重排序可能会导致一些问题**。

### 数据依赖性
如果两个操作访问同一个变量，且这两个操作中有一个为写操作，此时这两个操作之间就存在数据依赖关系   
**存在数据依赖关系，禁止重排序**，因为重排序会导致运行结果的不同     

数据依赖的三种关系：  
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/数据依赖的三种关系.png)

只要重排序两个操作的执行顺序，程序的执行结果将会被改变。前面提到过，编译器和处理器可能会对操作做重排序。编译器和处理器在重排序时，会遵守数据依赖性，编译器和处理器不会改变存在数据依赖关系的两个操作的执行顺序。

### 顺序一致性模型和JMM的保证

#### 数据竞争与顺序一致性

当程序未正确同步的时候，就可能存在**数据竞争**。  
如果程序中包含了数据竞争，那么运行的结果往往充满了不确定性，比如读发生在了写之前，可能就会读到错误的值；如果一个线程程序能够正确同步，那么就不存在数据竞争。

Java内存模型（JMM）对于正确同步多线程程序的内存一致性做了以下保证：    
**如果程序是正确同步的，程序的执行将具有顺序一致性。 即程序的执行结果和该程序在顺序一致性模型中执行的结果相同**。

这里的同步包括了使用`volatile、final、synchronized`等关键字来实现多线程下的同步。  

若没有正确使用`volatile、final、synchronized`，那么即便是使用了同步（单线程下的同步），**JMM也不会有内存可见性的保证，可能会导致你的程序出错，并且具有不可重现性**，很难排查


#### 顺序一致性模型

顺序一致性内存模型是一个理想化的**理论参考模型**,提供了极强的内存可见性保证(物理上的光滑平面没摩擦)

顺序一致性模型的两大特性:  
- 一个线程的所有操作必须按照程序的顺序(java 代码的顺序)来执行
- 不管程序是否同步,所有线程都只能看到一个单一的操作执行顺序,在顺序一致性模型中,每个操作必须**是原子性的,且立刻对所有线程可见**  

假设有两个线程A和B并发执行，线程A有3个操作，他们在程序中的顺序是`A1->A2->A3`，线程B也有3个操作，`B1->B2->B3`   
若正确同步，得到的顺序就是`A1->A2->A3->B1->B2->B3`   
若没有使用同步，得到的顺序可能是`B1->A2->A3->A1->B2->B3`操作的执行整体上无序，但是两个线程都只能看到这个执行顺序。之所以可以得到这个保证，是因为**顺序一致性模型中的每个操作必须立即对任意线程可见**

**但是JMM没有这样的保证。**
比如，在当前线程把写过的数据缓存在本地内存中，在没有刷新到主内存之前，这个写操作仅对当前线程可见；从其他线程的角度来观察，这个写操作根本没有被当前线程所执行。只有当前线程把本地内存中写过的数据刷新到主内存之后，这个写操作才对其他线程可见。在这种情况下，当前线程和其他线程看到的执行顺序是不一样的。


####  JMM中同步程序的顺序一致性效果
在顺序一致性模型中，所有操作完全按照程序的顺序串行执行。但是JMM中，临界区内（同步块或同步方法中）的代码可以发生重排序（但不允许临界区内的代码“逃逸”到临界区之外，因为会破坏锁的内存语义）

虽然线程A在临界区做了重排序，但是因为锁的特性，线程B无法观察到线程A在临界区的重排序。这种重排序既提高了执行效率，又没有改变程序的执行结果    

> 顺序一致性模型可以理解为，顺序统一的JMM，JMM是对整体一致性保证，同时对内部进行重排优化   

**JMM在不改变（正确同步的）程序执行结果的前提下，尽量为编译期和处理器的优化打开方便之门**。

#### JMM中未同步程序的顺序一致性效果
对于未同步的多线程程序，JMM只提供最小安全性：线程读取到的值，要么是之前某个线程写入的值，要么是默认值，不会无中生有  
为了实现这个安全性，JVM在堆上分配对象时，首先会对内存空间清零，然后才会在上面分配对象（这两个操作是同步的）。

**JMM没有保证未同步程序的执行结果与该程序在顺序一致性中执行结果一致。因为如果要保证执行结果一致，那么JMM需要禁止大量的优化，对程序的执行性能会产生很大的影响。**

####  差异
未同步程序在JMM和顺序一致性内存模型中的执行特性有如下差异： 
1. 顺序一致性保证单线程内的操作会按程序的顺序执行；JMM不保证单线程内的操作会按程序的顺序执行。（因为重排序，但是JMM保证单线程下的重排序不影响执行结果） 
2. 顺序一致性模型保证所有线程只能看到一致的操作执行顺序，而JMM不保证所有线程能看到一致的操作执行顺序。（因为JMM不保证所有操作立即可见） 
3. JMM不保证对64位的long型和double型变量的写操作具有原子性，而顺序一致性模型保证对所有的内存读写操作都具有原子性。

## happens-before 

一方面，程序员需要JMM提供一个强的内存模型来编写代码；另一方面，编译器和处理器希望JMM对它们的束缚越少越好，这样它们就可以最可能多的做优化来提高性能，希望的是一个弱的内存模型。  
JMM考虑了这两种需求，并且找到了平衡点，**对编译器和处理器来说，只要不改变程序的执行结果（单线程程序和正确同步了的多线程程序），编译器和处理器怎么优化都行**

JMM提供了`happens-before规则（JSR-133规范）`，满足了程序员的需求——简单易懂，并且提供了足够强的内存可见性保证。换言之，**程序员只要遵循happens-before规则，那他写的程序就能保证在JMM中具有强的内存可见性。**

happens-before关系的定义如下：   
1. 如果一个操作happens-before另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。 
2. 两个操作之间存在happens-before关系，并不意味着Java平台的具体实现必须要按照happens-before关系指定的顺序来执行。**如果重排序之后的执行结果，与按happens-before关系来执行的结果一致，那么JMM也允许这样的重排序。**

**如果操作A happens-before操作B，那么操作A在内存上所做的操作对操作B都是可见的，不管它们在不在一个线程。**


### 天然的happens-before关系
在Java中，有以下天然的happens-before关系：
- `程序顺序规则`：一个线程中的每一个操作，happens-before于该线程中的任意后续操作。
- `监视器锁规则`：对同一个锁的解锁`unlock`，happens-before于随后对这个锁的加锁`lock`。
- `volatile变量规则`：对一个volatile域的写，happens-before于**任意后续**对这个volatile域的读。
- `start启动规则`：如果线程A执行操作`ThreadB.start()`启动线程B，那么A线程的`ThreadB.start()`操作happens-before于线程B中的任意操作、
- `中断规则`：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生，即先于`interrupted()`检测
- `join规则`：如果线程A执行操作`ThreadB.join()`并成功返回，那么线程 B中的任意操作happens-before于线程A从`ThreadB.join()`操作成功返回
- `对象终结规则`：一个对象的初始化完成先行发生于它的`finalize`方法的开始
- `传递性`：如果A happens-before B，且B happens-before C，那么A happens-before C。

**举例** 

```java
int a = 1; // A操作
int b = 2; // B操作
int sum = a + b;// C 操作
System.out.println(sum);

// 1> A happens-before B 
// 2> B happens-before C 
// 3> A happens-before C
```

真正在执行指令的时候，其实JVM有可能对操作A & B进行重排序，因为无论先执行A还是B，他们都对对方是可见的，并且不影响执行结果。
如果这里发生了重排序，这**在视觉上违背了happens-before原则，但是JMM是允许这样的重排序的**。

所以，我们只关心happens-before规则，不用关心JVM到底是怎样执行的。**只要确定操作A happens-before操作B就行了**。

重排序有两类，JMM对这两类重排序有不同的策略：
- 会改变程序执行结果的重排序，比如 A -> C，JMM要求编译器和处理器都不许禁止这种重排序。
- 不会改变程序执行结果的重排序，比如 A -> B，JMM对编译器和处理器不做要求，允许这种重排序

## volatile 

- 保证变量的内存可见性
- 禁止volatile变量与普通变量重排序

从volatile的内存语义上来看，volatile可以**保证内存可见性**且**禁止重排序**。  

在保证内存可见性这一点上，volatile有着与锁相同的内存语义，所以可以作为一个**轻量级的锁**来使用。   
但由于volatile仅仅保证对**单个volatile变量的读/写具有原子性**，而锁可以保证整个临界区代码的执行具有原子性。所以**在功能上，锁比volatile更强大；在性能上，volatile更有优势**。

### 内存可见性

```java
public class VolatileExample {
    int a = 0;
    volatile boolean flag = false;

    public void writer() {
        a = 1; // step 1
        flag = true; // step 2
    }

    public void reader() {
        if (flag) { // step 3
            System.out.println(a); // step 4
        }
    }
}
```

使用volatile关键字修饰了一个boolean类型的变量flag。   
所谓内存可见性，指的是当一个线程对volatile修饰的变量进行写操作（比如step 2）时，JMM会立即把该线程对应的本地内存中的共享变量的值刷新到主内存；当一个线程对volatile修饰的变量进行读操作（比如step 3）时，JMM会把立即该线程对应的本地内存置为无效，从主内存中读取共享变量的值  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/volatile内存示意图.jpg)  

而如果flag变量没有用volatile修饰，在step 2，线程A的本地内存里面的变量就不会立即更新到主内存，那随后线程B也同样不会去主内存拿最新的值，仍然使用线程B本地内存缓存的变量的值`a = 0，flag = false`


### 禁止重排序
为了提供一种比锁更轻量级的线程间的通信机制，JSR-133专家组决定增强volatile的内存语义：严格限制编译器和处理器对volatile变量与普通变量的重排序。  

以内存可见性的代码为例，若step1 和step2重排序，那么step4打印出的结果就有可能是0 而不是1了


### 内存屏障
JVM是怎么还能限制处理器的重排序的呢？它是通过**内存屏障**来实现的。   

什么是内存屏障？硬件层面，内存屏障分两种：`读屏障（Load Barrier）`和`写屏障（Store Barrier）`。

- `写屏障（Store Barrier）`告诉处理器在写屏障之前将所有存储在缓存(store bufferes)中的数据同步到主内存。也就是说当看到Store屏障指令，就必须把该指令之前所有写入指令执行完毕才能继续往下执行 
  - 在写指令之后插入写屏障，强制把写缓冲区的数据刷回到主内存中
- `读屏障（Load Barrier）`处理器在读屏障之后的读操作，都在读屏障之后执行。也就是说在Load屏障指令之后就能够保证后面的读取数据指令一定能够读取到最新的数据
  - 在读指令之前插入读屏障，让工作内存或CPU高速缓存当中的缓存数据失效，重新回到主内存中获取最新数据


内存屏障有两个作用：
- 阻止屏障两侧的指令重排序；
- 强制把写缓冲区/高速缓存中的脏数据等写回主内存，或者让缓存中相应的数据失效  
  - 注意这里的缓存主要指的是CPU缓存，如L1，L2等(JMM中的线程本地内存)

因此重排序时不允许把内存屏障之后的指令重排序到内存屏障之前，**对一个volatile变量的写,先行发生于任意后续对这个volatile变量的读**，也叫写后读

编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。编译器选择了一个比较保守的JMM内存屏障插入策略，这样可以保证在任何处理器平台，任何程序中都能得到正确的volatile内存语义。这个策略是:  
- 在每个`volatile写`操作前插入一个`StoreStore`屏障；
- 在每个`volatile写`操作后插入一个`StoreLoad`屏障；
- 在每个`volatile读`操作后插入一个`LoadLoad`屏障；
- 在每个`volatile读`操作后再插入一个`LoadStore`屏障。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/内存屏障.png) 

再逐个解释一下这几个屏障。注：下述Load代表读操作，Store代表写操作  
- `LoadLoad屏障`：对于这样的语句Load1; LoadLoad; Load2，在Load2及后续读取操作要读取的数据被访问前，保证Load1要读取的数据被读取完毕。
- `StoreStore屏障`：对于这样的语句Store1; StoreStore; Store2，在Store2及后续写入操作执行前，保证Store1的写入操作对其它处理器可见。
- `LoadStore屏障`：对于这样的语句Load1; LoadStore; Store2，在Store2及后续写入操作被刷出前，保证Load1要读取的数据被读取完毕。
- `StoreLoad屏障`：对于这样的语句Store1; StoreLoad; Load2，在Load2及后续所有读取操作执行前，保证Store1的写入对所有处理器可见。它的开销是四种屏障中最大的（冲刷写缓冲器，清空无效化队列）。在大多数处理器的实现中，这个屏障是个万能屏障，兼具其它三种内存屏障的功能


volatile与普通变量的重排序规则:  
1. 如果第一个操作是volatile读，那无论第二个操作是什么，都不能重排序；  
2. 如果第二个操作是volatile写，那无论第一个操作是什么，都不能重排序；  
3. 如果第一个操作是volatile写，第二个操作是volatile读，那不能重排序。    


| 第一个操作 | 第二个操作 普通读写 | 第二个操作 volatile读 | 第二个操作 volatile写 |
| ---------- | ------------------- | --------------------- | --------------------- |
| 普通读写   | 可以重排            | 可以重排              | 不可以重排            |
| volatile读 | 不可以重排          | 不可以重排            | 不可以重排            |
| volatile写 | 可以重排            | 不可以重排            | 不可以重排            |

###  volatile示例

#### volatile的禁止重排序
在禁止重排序这一点上，volatile也是非常有用的。比如我们熟悉的单例模式，其中有一种实现方式是`双重锁检查`，比如这样的代码:   

```java
public class Singleton {

    private static Singleton instance; // 不使用volatile关键字

    // 双重锁检验
    public static Singleton getInstance() {
        if (instance == null) { // 第7行
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton(); // 第10行
                }
            }
        }
        return instance;
    }
}
```

如果这里的变量声明不使用volatile关键字，是可能会发生**错误**的。第十行`instance = new Singleton()`它可能会被重排序   
在赋值阶段需要三个操作： 
1. 分配内存
2. 初始化对象
3. 将对象指向分配的内存地址   

若发生重排序，就可能会导致 1-3-2的顺序    
比如线程A在第10行执行了步骤1和步骤3，但是步骤2还没有执行完。这个时候线程A执行到了第7行，它会判定instance不为空，然后直接返回了一个未初始化完成的instance   

利用volatile禁止`2.初始化对象`和`3.将对象指向分配的内存地址`重排序

#### volatile不保证原子性示例

```java
//不保证原子性
class MyData {
    volatile int number = 0;

    //方法添加 synchronized 即可保证
    public void addNumPlus() {
        number++;
    }
}

public class Test01 {
    public static void main(String[] args) {
        MyData myData = new MyData();
        for (int i = 0; i < 20; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    myData.addNumPlus();
                }
            }).start();
        }
        while (Thread.activeCount()>2){
            Thread.yield();
        }

        //此时number 小于20000
        System.out.println(myData.number);
    }
}
```
关于 `n++` 多线程下的问题(非原子操作): `n++` 被拆分为3个指令, 获取值,执行加操作,返回结果,这三个操作不是原子操作,会出现写丢失,结果被其他线程覆盖   
- 解决原子性问题  
  - synchronized、lock
  - `AutomaticInteger`...  

### volatile 最佳实践

- 可以用做单一赋值，但不能参与符合运算赋值`i++...`
- 状态标志，判断业务是否结束
- 开销较低的读，写锁策略
- DCL双端锁(单例模式双重检查实现)

## synchronized 与锁


**Java多线程的锁都是基于对象(实例对象和类对象)的，Java中的每一个对象都可以作为一个锁**
### synchronized
在java5及其以前，只有synchronized,这个是`重量级锁`，是操作系统级别的重量级操作。假如锁的竞争比较激烈，性能下降。因为存在用户态和内核态之间的转换。   
在JavaSE1.6的时候，对synchronized进行了优化，引入了`偏向锁`和`轻量级锁`，以及锁的存储结构和升级过程，减少了获取锁和释放锁的性能消耗，有些情况下它也就不那么重了  

synchronized可以修饰`普通方法`，`静态方法`和`代码块`。当synchronized修饰一个方法或者一个代码块的时候，它能够保证在同一时刻最多只有一个线程执行该段代码。
- 对于普通同步方法，锁是当前实例对象（不同实例对象之间的锁互不影响）。
- 对于静态同步方法，锁是当前类的Class对象。
- 对于同步方法块，锁是Synchonized括号里配置的对象。  
当一个线程试图访问同步代码块时，它首先必须得到锁，退出或抛出异常时必须释放锁。

`synchronized` 三种形式的用法示例 

```java
// 关键字在实例方法上，锁为当前实例
public synchronized void instanceLock() {
    // code
}

// 关键字在静态方法上，锁为当前Class对象
public static synchronized void classLock() {
    // code
}

// 关键字在代码块上，锁为括号里面的对象
public void blockLock() {
    Object o = new Object();
    synchronized (o) {
        // code
    }
}
```

临界区概念:  
> 某一块代码区域，它同一时刻只能由一个线程执行，如果synchronized关键字在方法上，那临界区就是整个方法内部。   
> 而如果是使用synchronized代码块，那临界区就指的是代码块内部的区域。

等价的 `synchronized` 方法:

```java
// 关键字在实例方法上，锁为当前实例
public synchronized void instanceLock() {
    // code
}

// 关键字在代码块上，锁为括号里面的对象
public void blockLock() {
    synchronized (this) {
        // code
    }
}
```

```java
// 关键字在静态方法上，锁为当前Class对象
public static synchronized void classLock() {
    // code
}

// 关键字在代码块上，锁为括号里面的对象
public void blockLock() {
    synchronized (this.getClass()) {
        // code
    }
}
```

> **实际开发中，要考虑锁的性能损耗，尽量缩小加锁的范围，能缩区块就不锁方法，能不加锁就不加锁，能用对象锁不要用类锁**   

### 锁的升级

Java 6 为了减少获得锁和释放锁带来的性能消耗，引入了`偏向锁`和`轻量级锁`。在Java 6 以前，所有的锁都是`重量级锁`。
所以在Java 6 及其以后，一个对象其实有四种锁状态，它们级别由低到高依次是:    
- 无锁状态
- 偏向锁
- 轻量级锁
- 重量级锁

几种锁会随着竞争情况逐渐升级，锁的升级很容易发生，但是锁降级发生的条件会比较苛刻，锁降级发生在`Stop The World`期间，当JVM进入`安全点`的时候，会检查是否有闲置的锁，然后进行降级。

当对象状态为偏向锁时，Mark Word存储的是偏向的线程ID；   
当状态为轻量级锁时，Mark Word存储的是指向线程栈中Lock Record的指针；   
当状态为重量级锁时，Mark Word为指向堆中的monitor对象的指针。

#### 偏向锁

大多数情况下,锁不仅不存在多线程竞争，而且总是由同一线程多次获得，于是引入了偏向锁。 

偏向锁会**偏向于第一个访问锁的线程**，如果在接下来的运行过程中，该锁没有被其他的线程访问，则持有偏向锁的线程将**永远不需要触发同步**。
也就是说，**偏向锁在资源无竞争情况下消除了同步语句，连CAS操作都不做了，提高了程序的运行性能**。


**原理：**  
一个线程在第一次进入同步块时，会在对象头和栈帧中的锁记录里存储锁的偏向的线程ID。当下次该线程进入这个同步块时，会去检查锁的Mark Word里面是不是放的自己的线程ID。

如果是，表明该线程已经获得了锁，以后该线程在进入和退出同步块时不需要花费CAS操作来加锁和解锁 ；   
如果不是，就代表有另一个线程来竞争这个偏向锁。这个时候会尝试使用CAS来替换Mark Word里面的线程ID为新线程的ID，这个时候要分两种情况：
- 成功，表示之前的线程不存在了， Mark Word里面的线程ID为新线程的ID，锁不会升级，仍然为偏向锁；
- 失败，表示之前的线程仍然存在，那么暂停之前的线程，设置偏向锁标识为0，并设置锁标志位为00，升级为`轻量级锁`，会按照轻量级锁的方式进行竞争锁。 

**撤销(关闭)偏向锁大概的过程如下：**
- 在一个安全点（在这个时间点上没有字节码正在执行）停止拥有锁的线程。
- 遍历线程栈，如果存在锁记录的话，需要修复锁记录和Mark Word，使其变成无锁状态。
- 唤醒被停止的线程，将当前锁升级成轻量级锁。

**线程竞争偏向锁的过程如下**   
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/偏向锁2.jpg)

> 关闭偏向锁:    
> 在JDK6中，偏向锁是默认启用的。它提高了单线程访问同步资源的性能。但试想一下，如果你的同步资源或代码一直都是多线程访问的，那么消除偏向锁这一步骤对你来说就是多余的。  
> 偏向锁可以提高带有同步但无竞争的程序性能,如果应用程序里所有的锁通常处于竞争状态，那么偏向锁就会是一种累赘，对于这种情况，我们可以一开始就把偏向锁这个默认功能给关闭    

```
-XX:UseBiasedLocking=false
```

!> Java15中逐步废弃偏向锁,默认关闭   

**问题**：当对象进入偏向状态的时候，Mark Word大部分的空间（23个比特）都用于存储持有锁的线程ID了，这部分空间占用了原有存储对象哈希码的位置，那原来对象的哈希码怎么办呢？ ---《JVM 13.3.5》    

> 当一个对象已经计算过一致性哈希码后，它就**再也无法进入偏向锁状态了**    
> 而当一个对象当前正处于偏向锁状态，又收到需要计算其一致性哈希码请求时，它的偏向状态会被立即撤销，并且锁会膨胀为`重量级锁`  
> 轻量级锁状态下JVM会在当前线程的栈帧中创建一个锁记录`LockRecord`空间，用于存储锁对象的Mark Word拷贝，拷贝中可以包含hashcode和GC年龄，释放锁后将这些信息写回对象头  

#### 轻量级锁
多个线程在**不同时段**获取同一把锁，即**不存在锁竞争的情况，也就没有线程阻塞**。针对这种情况，JVM采用轻量级锁来避免线程的阻塞与唤醒。  

**轻量级锁的加锁**
JVM会为每个线程在当前线程的栈帧中创建用于存储锁记录的空间，我们称为`Displaced Mark Word`。如果一个线程获得锁的时候发现是轻量级锁，会把`锁的Mark Word`复制到自己的`Displaced Mark Word`里面。    
然后线程尝试用CAS将`锁的Mark Word`替换为指向锁记录的指针。如果成功，当前线程获得锁，如果失败，表示`Mark Word`已经被替换成了其他线程的锁记录，说明在与其它线程竞争锁，当前线程就尝试使用**自旋**来获取锁。

> 自旋是需要消耗CPU的，如果一直获取不到锁的话，那该线程就一直处在自旋状态，白白浪费CPU资源。解决这个问题最简单的办法就是指定自旋的次数，例如让其循环10次，如果还没获取到锁就进入阻塞状态

但是JDK6采用了更聪明的方式——**适应性自旋**，简单来说就是**线程如果自旋成功了，则下次自旋的次数会更多，如果自旋失败了，则自旋的次数就会减少**。

自旋也不是一直进行下去的，如果自旋到一定程度（和JVM、操作系统相关），依然没有获取到锁，称为自旋失败，那么这个线程会阻塞。同时这个锁就会升级成**重量级锁**

**轻量级锁的释放：**   
在释放锁时，当前线程会使用CAS操作将`Displaced Mark Word`的内容复制回`锁的Mark Word`里面。如果没有发生竞争，那么这个复制的操作会成功。   
如果有其他线程因为自旋多次导致轻量级锁升级成了重量级锁，那么CAS操作会失败，此时会释放锁并唤醒被阻塞的线程,重新争夺锁访问同步块。

#### 重量级锁  
重量级锁依赖于操作`系统的互斥量（mutex）` 实现的，而操作系统中线程间状态的转换需要相对比较长的时间，所以**重量级锁效率很低，但被阻塞的线程不会消耗CPU**。

每一个对象都可以当做一个锁，当多个线程同时请求某个对象锁时，对象锁会设置几种状态用来区分请求的线程：
- `Contention List`：所有请求锁的线程将被首先放置到该竞争队列
- `Entry List`：Contention List中那些有资格成为候选人的线程被移到Entry List
- `Wait Set`：那些调用wait方法被阻塞的线程被放置到Wait Set
- `OnDeck`：任何时刻最多只能有一个线程正在竞争锁，该线程称为OnDeck
- `Owner`：获得锁的线程称为Owner
- `!Owner`：释放锁的线程

当一个线程尝试获得锁时，如果该锁已经被占用，则会将该线程封装成一个ObjectWaiter对象插入到Contention List的队列的队首，然后调用park函数挂起当前线程。
当线程释放锁时，会从Contention List或EntryList中挑选一个线程唤醒，被选中的线程叫做Heir presumptive即假定继承人，假定继承人被唤醒后会尝试获得锁，但synchronized是非公平的，所以假定继承人不一定能获得锁。这是因为对于重量级锁，线程先自旋尝试获得锁，这样做的目的是为了减少执行操作系统同步操作带来的开销。如果自旋不成功再进入等待队列。这对那些已经在等待队列中的线程来说，稍微显得不公平，还有一个不公平的地方是自旋线程可能会抢占了Ready线程的锁。

如果线程获得锁后调用Object.wait方法，则会将线程加入到WaitSet中，当被Object.notify唤醒后，会将线程从WaitSet移动到Contention List或EntryList中去。需要注意的是，**当调用一个锁对象的wait或notify方法时，如当前锁的状态是偏向锁或轻量级锁则会先膨胀成重量级锁**

#### 锁对比

| 锁       | 优点                                                               | 缺点                                             | 适用场景                             |
| -------- | ------------------------------------------------------------------ | ------------------------------------------------ | ------------------------------------ |
| 偏向锁   | 加锁和解锁不需要额外的消耗，和执行非同步方法比仅存在纳秒级的差距。 | 如果线程间存在锁竞争，会带来额外的锁撤销的消耗。 | 适用于只有一个线程访问同步块场景。   |
| 轻量级锁 | 竞争的线程不会阻塞，提高了程序的响应速度。                         | 如果始终得不到锁竞争的线程使用自旋会消耗CPU。    | 追求响应时间。同步块执行速度非常快。 |
| 重量级锁 | 线程竞争不使用自旋，不会消耗CPU。                                  | 线程阻塞，响应时间缓慢。                         | 追求吞吐量。同步块执行时间较长。     |

#### 锁消除  
`锁消除`是指虚拟机即时编译器在运行时，对一些代码要求同步，但是对被检测到不可能存在共享数据竞争的锁进行消除。     
`锁消除`的主要判定依据来源于`逃逸分析`的数据支持，如果判断到一段代码中，在堆上的所有数据都不会逃逸出去被其他线程访问到，那就可以把它们当作栈上数据对待，认为它们是线程私有的，同步加锁自然就无须再进行  

锁消除就是逃逸分析中的`同步省略`，发生在JIT编译器解释运行阶段

```java
public void f() {
    Object hellis = new Object();
    //这个同步锁 会被消除
    synchronized(hellis) {
        System.out.println(hellis);
    }
}
```

#### 锁粗化 
如果虚拟机探测到有这样一串零碎的操作都对同一个对象加锁，将会把加锁同步的范围扩展（粗化）到整个操作序列的外部
感觉像提取公因数   

```java
public String concatString(String s1, String s2, String s3) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    sb.append(s3);
    return sb.toString();
}
```

每个StringBuffer.append()方法中都有一个同步块，锁就是sb对象这里虽然有锁，但是可以被安全地消除掉。在解释执行时这里仍然会加锁，但在经过服务端编译器的JIT即时编译之后，这段代码就会忽略所有的同步措施而直接执行

> 实际上上面的代码会转化为非线程安全的StringBuilder来完成字符串拼接，并不会加锁

### 公平锁和非公平锁   
> `非公平锁`:先尝试占有锁,失败后采用公平锁   
> `公平锁`: 按照线程申请顺序执行,等待队列顺序执行,按照`FIFO`(先进先出)规则    
> `ReentrantLock` 默认为非公平锁,对于 `Synchronized` 也是一种非公平锁    

```java
/** 源码
     * Creates an instance of {@code ReentrantLock}.
     * This is equivalent to using {@code ReentrantLock(false)}.
     */
    public ReentrantLock() {
        sync = new NonfairSync();
    }
    //重载方法 根据fair 来选择公平锁还是非公平锁 
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }

```

**为什么会有公平锁/非公平锁的设计?为什么默认非公平?**
恢复挂起的线程到真正锁的获取还是有时间差的，从开发人员来看这个时间微乎其微，但是从CPU的角度来看，这个时间差存在的还是很明显的。所以**非公平锁能更充分的利用CPU的时间片，尽量减少CPU空闲状态时间**。

> 使用多线程很重要的考量点是线程切换的开销，当采用非公平锁时，当1个线程请求锁获取同步状态，然后释放同步状态，所以刚释放锁的线程在此刻再次获取同步状态的概率就变得非常大，所以就减少了线程的开销。

?> 为了更高的吞吐量，很显然非公平锁是比较合适的，因为节省很多线程切换时间，吞吐量自然就上去了。否则那就用公平锁，大家公平使用


### 可重入锁
可重入锁也叫递归锁，同一个线程在外层方法获得锁后，在进入该线程的内层方法会自动获得锁(同一个锁对象),不会因为之前已经获取过还没释放而阻塞(不能自己阻塞自己)   
- `synchronized` (隐式) 和 `Lock` (显式) 都是可重入锁    
- `synchronized` 不需要手动上锁,解锁,所以是隐式，`Lock` 需要手动上锁,解锁,所以是显式 
   
**重入锁实现可重入性原理或机制是**：每一个锁关联一个线程持有者和计数器，当计数器为 0 时表示该锁没有被任何线程持有，那么任何线程都可能获得该锁而调用相应的方法；当某一线程请求成功后，JVM会记下锁的持有线程，并且将计数器置为 1；此时其它线程请求该锁，则必须等待；而该持有锁的线程如果再次请求这个锁，就可以再次拿到这个锁，同时计数器会递增；当线程退出同步代码块时，计数器会递减，如果计数器为 0，则释放该锁。 

```java
    static  Object object =new Object();
    public static void main(String[] args) {
        new Thread(()->{
            synchronized (object){
                //外层获取
                synchronized (object){
                    //内层自动获取 不用等外层的锁释放
                }
            }
        }).start();
```

```java
    static Lock lock = new ReentrantLock();
    public static void main(String[] args) {
        new Thread(()->{
            lock.lock();
            try {
                //外层
                lock.lock();
                try {
                    //内层
                }finally {
                    lock.unlock();
                }
            }finally {
                lock.unlock();
            }
        }).start();
```

**注意: `lock.lock()` 要和 `lock.unlock()` 对应,否则会影响其他线程获取锁**  

### 死锁
`死锁`是指两个或两个以上的线程在执行过程中,因争夺资源而造成的一种互相等待的现象,若无外力干涉那它们都将无法推进下去，如果系统资源充足，进程的资源请求都能够得到满足，死锁出现的可能性就很低，否则就会因争夺有限的资源而陷入死锁   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/juc2.png)  

**代码示例**

```java
/**
 * 死锁演示
 */
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

产生死锁的原因:
- 系统资源不足
- 进程运行推进顺序不合适
- 资源分配不当

验证是否为死锁
- `jps` 
- `jstack` jvm自带堆栈跟踪工具 

> `jps -l` 然后使用 `jstack pid` 查看

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/juc3.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/juc4.png)

### 自旋锁
`CAS`是实现自旋锁的基础，CAS利用CPU指令保证了操作的原子性，以达到锁的效果  
自旋是指尝试获取锁的线程不会立即`阻塞`，而是采用循环的方式去尝试获取锁，当线程发现锁被占用时，会不断循环判断锁的状态，直到获取。   

**这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗CPU**

?> 线程阻塞后，`挂起线程`和`恢复线程`的操作都需要转入内核态中完成，这些操作给Java虚拟机的并发性能带来了很大的压力 --《JVM》  

**实例代码**

```java
public class SpinLockDemo {
    AtomicReference<Thread>  at= new AtomicReference<>();

    public static void main(String[] args) throws InterruptedException {
        SpinLockDemo dd = new SpinLockDemo();
        new Thread(()->{
            dd.lock();
            try {
                TimeUnit.MILLISECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            dd.unLock();
        },"a").start();
        TimeUnit.MILLISECONDS.sleep(1);

        new Thread(()->{
            dd.lock();
            try {
                TimeUnit.MILLISECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            dd.unLock();
        },"b").start();
    }

    public void lock(){
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName() + "come in");
        while (!at.compareAndSet(null,thread)){
            System.out.println(thread.getName()+"开始自旋等待");
        }
    }

    public void unLock(){
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName() + "unlock");
        at.compareAndSet(thread,null);
    }
}
```

```
acome in
bcome in
b开始自旋等待
b开始自旋等待
...
b开始自旋等待
aunlock
b开始自旋等待
bunlock

```

#### 自旋的说明
自旋等待不能代替阻塞，且先不说对处理器数量的要求，自旋等待本身虽然避免了线程切换的开销，但它是要占用处理器时间的，所以如果锁被占用的时间很短，自旋等待的效果就会非常好，反之如果锁被占用的时间很长，那么自旋的线程只会白白消耗处理器资源，而不会做任何有价值的工作，这就会带来性能的浪费。

因此自旋等待的时间必须有一定的限度，如果自旋超过了限定的次数仍然没有成功获得锁，就应当使用传统的方式去挂起线程。**自旋次数的默认值是十次**，用户也可以使用参数`-XX：PreBlockSpin`来自行更改  

#### 适应性自旋
JDK 6中对自旋锁的优化，引入了`自适应的自旋`  
同一个锁对象上，自旋等待刚刚成功获得过锁，那么下一次自旋次数会增多，反之就会减少

### 读写锁和排它锁
我们前面讲到的`synchronized`用的锁和`ReentrantLock`，其实都是`排它锁`。也就是说，**这些锁在同一时刻只允许一个线程进行访问**。

而读写锁可以在同一时刻允许多个读线程访问。Java提供了`ReentrantReadWriteLock类`作为读写锁的默认实现，   
内部维护了两个锁：一个读锁，一个写锁。通过分离读锁和写锁，使得在`读多写少`的环境下，大大地提高了性能。

注意，即使用读写锁，在写线程访问时，所有的读线程和其它写线程均被阻塞。   

#### 读写锁
读写锁，是一对相关的锁——读锁和写锁，读锁用于只读操作，写锁用于写入操作。读锁可以由多个线程同时保持，而写锁是独占的，只能由一个线程获取

读写锁比较适用于以下情形：
- 高频次的读操作，相对较低频次的写操作；
- 读操作所用时间不会太短。(否则读写锁本身的复杂实现所带来的开销会成为主要消耗成本)  


#### synchronized的不足之处
- 如果临界区是只读操作，其实可以多线程一起执行，但使用synchronized的话，同一时间只能有一个线程执行。
- synchronized无法知道线程有没有成功获取到锁
- 使用synchronized，如果临界区因为IO或者sleep方法等原因阻塞了，而当前线程又没有释放锁，就会导致所有线程等待。

#### ReentrantReadWriteLock
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/读写锁依赖示意图.png)


`ReentrantReadWriteLock类`，顾名思义，是一种读写锁，它是`ReadWriteLock接口`的直接实现，该类在内部实现了具体独占锁特点的写锁，以及具有共享锁特点的读锁，和`ReentrantLock`一样，`ReentrantReadWriteLock`类也是通过定义内部类实现AQS框架的API来实现独占/共享的功能。    

```java
//创建读写锁
private ReadWriteLock rwLock = new ReentrantReadWriteLock();
// 写锁 加锁 
rwLock.writeLock().lock();
// 写锁解锁
rwLock.writeLock().unlock();

// 读锁 加锁
rwLock.readLock().lock();
// 解锁
rwLock.readLock().unlock();`
```

#### 简单案例

```java
//资源类
class MyCache {
    //创建map集合
    private volatile Map<String,Object> map = new HashMap<>();

    //创建读写锁对象
    private ReadWriteLock rwLock = new ReentrantReadWriteLock();

    //放数据
    public void put(String key,Object value) {
        //添加写锁
        rwLock.writeLock().lock();

        try {
            System.out.println(Thread.currentThread().getName()+" 正在写操作"+key);
            //暂停一会
            TimeUnit.MICROSECONDS.sleep(300);
            //放数据
            map.put(key,value);
            System.out.println(Thread.currentThread().getName()+" 写完了"+key);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            //释放写锁
            rwLock.writeLock().unlock();
        }
    }

    //取数据
    public Object get(String key) {
        //添加读锁
        rwLock.readLock().lock();
        Object result = null;
        try {
            System.out.println(Thread.currentThread().getName()+" 正在读取操作"+key);
            //暂停一会
            TimeUnit.MICROSECONDS.sleep(300);
            result = map.get(key);
            System.out.println(Thread.currentThread().getName()+" 取完了"+key);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            //释放读锁
            rwLock.readLock().unlock();
        }
        return result;
    }
}

public class ReadWriteLockDemo {
    public static void main(String[] args) throws InterruptedException {
        MyCache myCache = new MyCache();
        //创建线程放数据
        for (int i = 1; i <=5; i++) {
            final int num = i;
            new Thread(()->{
                myCache.put(num+"",num+"");
            },String.valueOf(i)).start();
        }

        TimeUnit.MICROSECONDS.sleep(300);

        //创建线程取数据
        for (int i = 1; i <=5; i++) {
            final int num = i;
            new Thread(()->{
                myCache.get(num+"");
            },String.valueOf(i)).start();
        }
    }
}

```

```
//执行结果
2 正在写操作2
2 写完了2
3 正在写操作3
3 写完了3
1 正在写操作1
1 写完了1
1 正在读取操作1
2 正在读取操作2
3 正在读取操作3
2 取完了2
3 取完了3
1 取完了1

```

得出结论： 一个资源可以被多个读线程访问,或者一个写线程访问,但是不能同时存在读写线程,读写互斥,读读共享

#### 锁降级 
将**写锁**降级为**读锁** 的过程叫锁降级,但是读锁不能升级为写锁 

如果同一个线程持有了写锁，在没有释放写锁的情况下，还可以继续获取读锁，这就是写锁的降级，即写锁降级成了读锁，如果释放了写锁，那就完全转换成了读锁    
若持有了读锁，是不可以升级为写锁的，也是就是无法继续获取读锁，陷入死锁

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/锁降级过程示例.png)

示例代码: 注意必须为同一个 `ReentrantReadWriteLock` 下的读锁和写锁才可以

```java
//演示读写锁降级
public class Demo1 {

    public static void main(String[] args) {
        //可重入读写锁对象
        ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
        ReentrantReadWriteLock.ReadLock readLock = rwLock.readLock();//读锁
        ReentrantReadWriteLock.WriteLock writeLock = rwLock.writeLock();//写锁

        //锁降级
        //1 获取写锁
        writeLock.lock();
        System.out.println("manongyanjiuseng");
        
        //2 获取读锁
        readLock.lock();
        System.out.println("---read");
        
        //3 释放写锁
        writeLock.unlock();

        //4 释放读锁
        readLock.unlock();
    }
}
// manongyanjiuseng
// ---read
```

上述代码在获取了写锁后，仍然可以继续获取读锁，并成功释放读写锁  

#### 特点和应用

读写锁比较适用于以下情形：
- 高频次的读操作，相对较低频次的写操作；
- 读操作所用时间不会太短。（否则读写锁本身的复杂实现所带来的开销会成为主要消耗成本）

`ReentrantReadWriteLock`类具有如下特点：    
- 支持`公平/非公平`策略与ReadWriteLock类一样，ReentrantReadWriteLock对象在构造时，可以传入参数指定是公平锁还是非公平锁
- 支持`锁重入`
  - 同一读线程在获取了读锁后还可以获取读锁；
  - 同一写线程在获取了写锁之后既可以再次获取写锁又可以获取读锁
- 支持`锁降级`
  - 所谓锁降级，就是：先获取写锁，然后获取读锁，最后释放写锁，这样写锁就降级成了读锁。但是，读锁不能升级到写锁。
- Condition条件支持
  - `ReentrantReadWriteLock`的内部读锁类、写锁类实现了`Lock`接口，所以可以通过`newCondition()`方法获取Condition对象。但是这里要注意，**读锁是没法获取`Condition`对象的**，读锁调用`newCondition()`方法会直接抛出`UnsupportedOperationException`


#### 缺点  
读操作比较多的时候，获取写锁就变得困难了，容易造成锁饥饿问题

如何解决锁饥饿问题？  
- 使用公平锁可以缓解，但是牺牲了吞吐量
- `StampedLock`乐观读锁

### StampedLock 
StampedLock是比ReentrantReadWriteLock更快的一种锁，支持`乐观读`、`悲观读`和`写锁`。和ReentrantReadWriteLock不同的是，StampedLock支持多个线程申请乐观读的同时，还允许一个线程申请写锁     

ReentrantReadWriteLock会发生`写饥饿`的现象，但StampedLock不会。它是怎么做到的呢？   
它的核心思想在于，在读的时候如果发生了写，应该通过重试的方式来获取新的值，而不应该阻塞写操作。这种模式也就是典型的`无锁编程思想`，和CAS自旋的思想一样。这种操作方式决定了StampedLock在读线程非常多而写线程非常少的场景下非常适用，同时还避免了写饥饿情况的发生  

StampedLock有三种访问模式： 
- `Reading模式`悲观读模式，类似ReentrantReadWriteLock读锁
- `Writing模式`写模式，类似ReentrantReadWriteLock写锁
- `Opeimistic reading` 乐观读模式，无锁限制，类似于乐观锁，支持读写并发，若读取时被修改，再升级为悲观读模式

#### 代码实现

```java
class StampTest {
    static int number = 37;
    static StampedLock stampedLock = new StampedLock();

    public void write() {
        long writeLock = stampedLock.writeLock();
        System.out.println("写操作开始");
        try {
            number = number + 3;
            System.out.println("写操作进行中");
        } finally {
            stampedLock.unlockWrite(writeLock);
        }
        System.out.println("写结束");
    }

    //悲观读
    public void read() {
        long readLock = stampedLock.readLock();
        System.out.println("进入悲观读锁");
        for (int i = 0; i < 4; i++) {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("读取中");
        }
        try {
            int result = number;
            System.out.println(result + "结果" + Thread.currentThread().getName());
        } finally {
            stampedLock.unlockRead(readLock);
        }

    }

    //乐观读
    public void tryLock() {
        long stamp = stampedLock.tryOptimisticRead();
        int result = number;
        //值 是否被修改，true 无修改 false 有修改
        System.out.println("是否有修改" + stampedLock.validate(stamp));
        for (int i = 0; i < 4; i++) {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("乐观读取中...值是" + result);
        }
        //值 是否被修改，true 无修改 false 有修改
        //被修改升级悲观读
        if (!stampedLock.validate(stamp)) {
            System.out.println("有人修改过");
            //变悲观读锁
            stamp = stampedLock.readLock();
            try {
                System.out.println("乐观读升级为悲观读");
                result = number;
                System.out.println("悲观读取中...值是" + result);

            } finally {
                stampedLock.unlockRead(stamp);
            }

            System.out.println(Thread.currentThread().getName() + "最终值" + result);
        }
    }

    public static void main(String[] args) {
        StampTest stampTest = new StampTest();
//悲观读
//        new Thread(() -> {
//            stampTest.read();
//        }, "readThread").start();
//        try {
//            TimeUnit.SECONDS.sleep(1);
//        } catch (InterruptedException e) {
//            e.printStackTrace();
//        }

//乐观读
        new Thread(() -> {
            stampTest.tryLock();
        }, "readtryThread").start();

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
//写操作
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "come in");
            stampTest.write();
        }, "writeThread").start();

    }
}
```

#### 特点
- StampedLock不支持重入，没有Re开头
- StampedLock的悲观读锁和写锁都不支持Condition
- StampedLock不要调用中断操作
- StampedLock也是基于state和队列实现的，只不过不能重入。通过引入了乐观读并不加锁，在读多写少的场景中性能会比ReentrantReadWriteLock要好


## CAS与原子操作

### 乐观锁与悲观锁

`悲观锁`就是我们常说的锁。对于悲观锁来说，它**总是认为每次访问共享资源时会发生冲突**，所以必须**对每次数据操作加上锁**，以保证临界区的程序同一时间只能有一个线程在执行

`乐观锁`又称为“无锁”，顾名思义，它是乐观派。乐观锁总是假设对共享资源的访问没有冲突，线程可以不停地执行，无需加锁也无需等待。  
而一旦多个线程发生冲突，**乐观锁通常是使用一种称为CAS的技术来保证线程执行的安全性**    
乐观锁天生免疫死锁。  


> 乐观锁多用于“**读多写少**“的环境，避免频繁加锁影响性能；而悲观锁多用于”**写多读少**“的环境，避免频繁失败和重试影响性能。

### CAS的概念
CAS的全称是：**比较并交换(Compare And Swap)**

在CAS中，有这样三个值：
- V：要更新的变量(var)
- E：预期值(expected)
- N：新值(new)

比较并交换的过程如下：    
判断V是否等于E，如果等于，将V的值设置为N；如果不等，说明已经有其它线程更新了V，则当前线程放弃更新，什么都不做。


简单的例子来解释这个过程：  
如果有一个多个线程共享的变量i原本等于5，我现在在线程A中，想把它设置为新的值6;    
我们使用CAS来做这个事情；
- 首先我们用i去与5对比，发现它等于5，说明没有被其它线程改过，那我就把它设置为新的值6，**此次CAS成功，i的值被设置成了6**；
- 如果不等于5，**说明i被其它线程改过了**（比如现在i的值为2），那么我就什么也不做，**此次CAS失败，i的值仍然为2**。

**CAS是一种原子操作(基于CPU保证)，它是一种系统原语，是一条CPU的原子指令，从CPU层面保证它的原子性**

?> 当多个线程同时使用CAS操作一个变量时，只有一个会胜出，并成功更新，其余均会失败，但失败的线程并不会被挂起，仅是被告知失败，并且允许再次尝试(自旋)，当然也允许失败的线程放弃操作

### CAS示例  
以`AtomicInteger`为例，尝试将原值为5的`AtomicInteger`更新为2023

```java
public static void main(String[] args) throws InterruptedException {
    AtomicInteger atomicInteger = new AtomicInteger(5);
    boolean b = atomicInteger.compareAndSet(5, 2023);//true 赋值成功
    System.out.println(atomicInteger.get());//2023
    boolean b1 = atomicInteger.compareAndSet(5, 2023);//false 赋值失败
    System.out.println(atomicInteger.get());//2023
}

//AtomicInteger 源码 expect期望值，若原值等于期望值就更新为 update值
//最终调用的是Unsafe类native 方法 
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```


### Java实现CAS的原理 - Unsafe类
Java中，有一个 `Unsafe` 类，它在`sun.misc`包中。它里面是一些`native`方法，其中就有几个关于CAS的:
- `var1`表示要操作的对象
- `var2`表示要操作对象中属性地址的偏移量
- `var4`表示需要修改的数据的期望值
- `var5、6`要改为的新值

```java
public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);
public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);
```  

他们都是`public native`的。  
Unsafe中对CAS的实现是C++写的，它的具体实现和操作系统、CPU都有关系

Unsafe类里面还有其它方法用于不同的用途。比如支持线程挂起和恢复的`park`和`unpark`， `LockSupport`类底层就是调用了这两个方法。   
还有支持反射操作的`allocateInstance()`方法。

### 原子操作-AtomicInteger 解析
JDK提供了一些用于原子操作的类，在`java.util.concurrent.atomic`包下面

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/atomic原子类集合.png)  

从名字就可以看得出来这些类大概的用途：  
- 原子更新基本类型
- 原子更新数组
- 原子更新引用
- 原子更新字段（属性）

例： `AtomicInteger` 类的 `getAndAdd(int delta)` 方法是调用 `Unsafe` 类的方法来实现的

```java

    public final int getAndAdd(int delta) {
        return unsafe.getAndAddInt(this, valueOffset, delta);
    }
```

```java
    public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            //获取原值
            var5 = this.getIntVolatile(var1, var2);
            //然后进行cas 成功退出返回，失败继续重拾
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

        return var5;
    }

```

我们来一步步解析这段源码。首先，对象`var1`是`this`，也就是一个`AtomicInteger`对象。然后`var2`是一个常量`valueOffset`。这个常量是在`AtomicInteger`类中声明的

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }
    ...
}
```

同样是调用的`Unsafe`的方法。从方法名字上来看，是得到了一个对象字段偏移量  

> 用于获取某个字段相对Java对象的`起始地址`的偏移量。   
> 一个java对象可以看成是一段内存，各个字段都得按照一定的顺序放在这段内存里，同时考虑到对齐要求，可能这些字段不是连续放置的，   
> 用这个方法能准确地告诉你某个字段相对于对象的起始内存地址的字节偏移量，因为是相对偏移量，所以它其实跟某个具体对象又没什么太大关系，跟class的定义和虚拟机的内存模型的实现细节更相关。   

CAS是`无锁`的基础，它允许更新失败。**所以经常会与while循环搭配，在失败后不断去重试**。   
这里声明了一个`v`，也就是要返回的值。从`getAndAddInt`来看，它返回的应该是原来的值，而新的值的`v + delta`。  

?> 这里使用do-while循环。这种循环不多见，它的**目的是保证循环体内的语句至少会被执行一遍**。这样才能保证`return` 的值`v`是我们期望的值  

可以看到它是在**不断尝试去用CAS更新**。如果更新失败，就继续重试。    

那为什么要把获取“旧值”v的操作放到循环体内呢？其实这也很好理解。  
CAS如果旧值V不等于预期值E，它就会更新失败。说明旧的值发生了变化。那我们当然需要返回的是被其他线程改变之后的旧值(内存可见)，以便再次更新，因此放在了do循环体内。

循环体的条件是一个CAS方法，也就是一个native方法：

```java
    public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/atomicInteger.png)

### AtomicReference 示例

```java
public class TestLock {
    public static void main(String[] args) throws InterruptedException {
        AtomicReference<TUser> a = new AtomicReference<>();

        TUser tUser1 = new TUser("张三",10);
        TUser tUser2 = new TUser("李四",10);
        a.set(tUser1);
        boolean b = a.compareAndSet(tUser1, tUser2);
        boolean b2 = a.compareAndSet(tUser1, tUser2);
        System.out.println(b+"--"+a.get());//true--TUser{name='李四', age=10}

        System.out.println(b2+"--"+a.get());//false--TUser{name='李四', age=10}

    }
}

class TUser {
    private String name;
    private Integer age;

    public TUser(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public TUser() {
    }

    @Override
    public String toString() {
        return "TUser{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

###  CAS实现原子操作的三大问题

#### ABA问题

所谓ABA问题，就是一个值原来是A，变成了B，又变回了A。这个时候使用CAS是检查不出变化的，但实际上却被更新了两次
ABA问题的解决思路是在变量前面追加上**版本号或者时间戳**。   
从JDK 1.5开始，JDK的atomic包里提供了一个类`AtomicStampedReference`或者`AtomicMarkableReference `来解决ABA问题。  
- `AtomicStampedReference`通过 int 类型的版本号来区分是否被修改过
- `AtomicMarkableReference`通过 boolean 型的标识来判断数据是否有更改
  - 在现实业务场景中，不关心引用变量被修改了几次，只是单纯的关心是否更改过。

`AtomicStampedReference`这个类的`compareAndSet`方法的作用是首先检查当前引用是否等于预期引用，并且检查当前标志是否等于预期标志，如果二者都相等，才使用CAS设置为新的值和标志

```java
    public boolean compareAndSet(V   expectedReference,
                                 V   newReference,
                                 int expectedStamp,
                                 int newStamp) {
        Pair<V> current = pair;
        return
            expectedReference == current.reference &&
            expectedStamp == current.stamp &&
            ((newReference == current.reference &&
              newStamp == current.stamp) ||
             casPair(current, Pair.of(newReference, newStamp)));
    }

```

**实例**

```java
public class Test {
    public static void main(String[] args) throws InterruptedException {
        TUser tUser1 = new TUser("张三",10);
        //版本号1
        AtomicStampedReference<TUser> a = new AtomicStampedReference<>(tUser1,1);
        System.out.println(a.getReference());
        System.out.println(a.getStamp());

        TUser tUser2 = new TUser("李四",10);
        
        //更新失败 版本号不对
        a.compareAndSet(tUser1,tUser2,3,2);
        System.out.println(a.getReference());
        System.out.println(a.getStamp());
        //更新成功
        a.compareAndSet(tUser1,tUser2,1,2);
        System.out.println(a.getReference());
        System.out.println(a.getStamp());
    }
}

class TUser {
    private String name;
    private Integer age;

    public TUser(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public TUser() {
    }

    @Override
    public String toString() {
        return "TUser{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

#### 循环时间长开销大
CAS多与自旋结合。如果自旋CAS长时间不成功，会占用大量的CPU资源。  
解决思路是让JVM支持处理器提供的pause指令。    
pause指令能让自旋失败时cpu睡眠一小段时间再继续自旋，从而使得读操作的频率低很多,为解决内存顺序冲突而导致的CPU流水线重排的代价也会小很多。    


#### 只能保证一个共享变量的原子操作

- 使用JDK 1.5开始就提供的`AtomicReference`类保证对象之间的原子性，把多个变量放到一个对象里面进行CAS操作，详见`原子操作类`  
- 使用锁。锁内的临界区代码可以保证只有当前线程能操作。

## 对象的内存布局
在HotSpot虚拟机里，对象在堆内存中的存储布局可以划分为三个部分：`对象头（Header）`、`实例数据（Instance Data）`和`对齐填充（Padding）`

HotSpot虚拟机对象的对象头部分包括两类信息。   
第一类是用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等，这部分数据的长度在32位和64位的虚拟机（未开启压缩指针）中分别为32个比特和64个比特，官方称它为`Mark Word`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/对象头markword.png)

对象头的另外一部分是`类型指针`，即对象指向它的类型元数据的指针，Java虚拟机通过这个指针来确定该对象是哪个类的实例

一个Object默认大小是16字节

### Markword
由于对象头信息是与对象自身定义的数据无关的额外存储成本，考虑到Java虚拟机的空间使用效率，Mark Word被设计成一个非固定的动态数据结构，以便在极小的空间内存储尽量多的信息。它会根据对象的状态复用自己的存储空间。   
例如在32位的HotSpot虚拟机中，对象未被锁定的状态下，Mark Word的32个比特空间里的25个比特将用于存储对象哈希码，4个比特用于存储对象分代年龄，2个比特用于存储锁标志位，还有1个比特固定为0（这表示未进入偏向模式）。对象除了未被锁定的正常状态外，还有轻量级锁定、重量级锁定、GC标记、可偏向等几种不同状态

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/对象头markword2.png)

## AQS  
AQS是`java.util.concurrent.locks.AbstractQueuedSynchronizer`的简称，即**抽象 队列 同步器**，从字面意思上理解:  
- 抽象：抽象类，只实现一些主要逻辑，有些方法由子类实现
- 队列：使用先进先出（FIFO）队列存储数据
- 同步：实现了同步的功能

那AQS有什么用？   
AQS是一个**用来构建锁和同步器的框架**，使用AQS能简单且高效地构造出应用广泛的同步器，比如我们提到的`ReentrantLock`，`CountDownLatch`,`Semaphore`，`ReentrantReadWriteLock`，`SynchronousQueue`，`FutureTask`等等皆是基于AQS的。   


### AQS的结构、Node节点
AQS内部使用了一个`volatile的`变量`state`来作为资源的标识。同时定义了几个获取和改变state的protected方法，子类可以覆盖这些方法来实现自己的逻辑：
- `getState()`
- `setState()`
- `compareAndSetState()`

这三种叫做均是原子操作，其中`compareAndSetState`的实现依赖于`Unsafe类的compareAndSwapInt()`方法。   

而AQS类本身实现的是一些排队和阻塞的机制，比如具体线程等待队列的维护（如获取资源失败入队/唤醒出队等）。它内部使用了一个先进先出（FIFO）的双端队列，并使用了两个指针head和tail用于标识队列的头部和尾部。其数据结构如图:   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/aqs双端队列示意图.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/AQS节点CLH队列.png)

AQS将每条要去抢占资源的线程封装成一个Node节点来实现所得分配  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/AQS节点链表.png)  

### AQS和锁的关系

以一个简单示例，说明锁的调用流程和AQS的关系     

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/reentrantlock和aqs的关系.png)

```java
public class Test {
    ReentrantLock lock = new ReentrantLock();
    public void lockTest(String s1, String s2, String s3) {
        lock.lock();
        try {

        }finally {
            lock.unlock();
        }
    }
}
```

1. 首先构建锁对象，`new ReentrantLock()` 默认实现的是非公平锁，返回一个`NonfairSync`对象。   

```java
// 创建锁对象源码
    public ReentrantLock() {
        sync = new NonfairSync();
    }

    /**
     * Sync object for non-fair locks
     */
     //NonfairSync对象源码
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        /**
         * Performs lock.  Try immediate barge, backing up to normal
         * acquire on failure.
         */
        final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }

        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }
```

2. 调用`lock.lock();`就是在调用`NonfairSync`的lock方法，这个方法是继承自`NonfairSync`的父类`Sync`, `Sync`类是`ReentrantLock`的静态内部类

```java
        final void lock() {
            //CAS修改锁的状态位 state ，0空闲 1占用 这里是尝试修改为1 即尝试获取锁
            if (compareAndSetState(0, 1))
                //将该锁的拥有者设置为当前线程
                setExclusiveOwnerThread(Thread.currentThread());
            else
                //否则调用 AQS的acquire 方法
                acquire(1);
        }
```

3. 调用`lock.unlock();`解锁

```java
    public void unlock() {
        //调用 AQS (Sync 的父类) 的release 方法
        sync.release(1);
    }
```


4. 若创建对象为公平锁`new ReentrantLock(true)`，返回一个`FairSync`,其lock方法就是直接调用AQS的`acquire()`

```java
static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        final void lock() {
            acquire(1);
        }

        /**
         * Fair version of tryAcquire.  Don't grant access unless
         * recursive call or no waiters or is first.
         */
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
```

5. 可以看到公平锁和非公平锁(竞争时)最终都调用了AQS的`acquire()`

```java
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```

6. `tryAcquire()`方法中，公平锁和非公平锁的实现略有不同，公平锁多了一个方法即AQS的`hasQueuedPredecessors`这个方法是判断当前线程前是否还有线程排队

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/公平锁和非公平锁的tryacquire.png)

```java
//返回true 说明当前线程前还有线程排队，false说明当前线程位于队列头或队列是空队列
    public final boolean hasQueuedPredecessors() {
        // The correctness of this depends on head being initialized
        // before tail and on head.next being accurate if the current
        // thread is first in queue.
        Node t = tail; // Read fields in reverse initialization order
        Node h = head;
        Node s;
        return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());
    }
```

### 资源共享模式
资源有两种共享模式，或者说两种同步方式：  
- `独占模式（Exclusive）`：资源是独占的，一次只能一个线程获取。如`ReentrantLock`。
- `共享模式（Share）`：同时可以被多个线程获取，具体的资源个数可以通过参数指定。如`Semaphore/CountDownLatch`。  

一般情况下，子类只需要根据需求实现其中一种模式，当然也有同时实现两种模式的同步类，如`ReadWriteLock`。    
AQS中关于这两种资源共享模式的定义源码（均在内部类`Node`中）    

```java
static final class Node {
    
    // 共享模式结点
    static final Node SHARED = new Node();
    
    // 独占模式结点
    static final Node EXCLUSIVE = null;

    static final int CANCELLED =  1;

    static final int SIGNAL    = -1;

    static final int CONDITION = -2;

    static final int PROPAGATE = -3;

    /**
    * INITAL：      0 - 默认，新结点会处于这种状态。
    * CANCELLED：   1 - 取消，表示后续结点被中断或超时，需要移出队列；
    * SIGNAL：      -1- 发信号，表示后续结点被阻塞了；（当前结点在入队后、阻塞前，应确保将其prev结点类型改为SIGNAL，以便prev结点取消或释放时将当前结点唤醒。）
    * CONDITION：   -2- Condition专用，表示当前结点在Condition队列中，因为等待某个条件而被阻塞了；
    * PROPAGATE：   -3- 传播，适用于共享模式。（比如连续的读操作结点可以依次进入临界区，设为PROPAGATE有助于实现这种迭代操作。）
    * 
    * waitStatus表示的是后续结点状态，这是因为AQS中使用CLH队列实现线程的结构管理，而CLH结构正是用前一结点某一属性表示当前结点的状态，这样更容易实现取消和超时功能。
    */
    volatile int waitStatus;

    // 前驱指针
    volatile Node prev;

    // 后驱指针
    volatile Node next;

    // 结点所包装的线程
    volatile Thread thread;

    // Condition队列使用，存储condition队列中的后继节点
    Node nextWaiter;

    Node() {
    }

    Node(Thread thread, Node mode) { 
        this.nextWaiter = mode;
        this.thread = thread;
    }
}

```

### AQS的主要方法源码解析
AQS的设计是基于**模板方法模式**的，它有一些方法必须要子类去实现的，它们主要有：  
- `isHeldExclusively()`：该线程是否正在独占资源。只有用到condition才需要去实现它。
- `tryAcquire(int)`：独占方式。尝试获取资源，成功则返回true，失败则返回false。
- `tryRelease(int)`：独占方式。尝试释放资源，成功则返回true，失败则返回false。
- `tryAcquireShared(int)`：共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
- `tryReleaseShared(int)`：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false。

> 这些方法都是`protected`方法,为了避免强迫子类中把所有的抽象方法都实现一遍，减少无用功，不使用抽象方法的形式    
> 这样子类只需要实现自己关心的抽象方法即可，比如 `Semaphore` 只需要实现 `tryAcquire `方法而不用实现其余不需要用到的模版方法

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

`AQS acquire`主要有三条流程 ： 
- 调用`tryAcquire`,交由子类`FairSync`和`NonfairSync`实现   
- 调用`addWaiter`,enq入队
- 调用`acquireQueued`，调用cancelAcquire

**acquire执行流程** 
该方法以独占方式获取资源，如果获取到资源，线程继续往下执行，否则进入等待队列，直到获取到资源为止，且整个过程忽略中断的影响。该方法是独占模式下线程获取共享资源的顶层入口  

```java
//arg 始终是1
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```


**Step1**：首先这个方法调用了用户自己实现的方法`tryAcquire`方法尝试获取资源，如果这个方法返回true，也就是表示获取资源成功，那么整个acquire方法就执行结束了，线程继续往下执行,源码`tryAcquire`以公平锁为例

```java
//公平锁的 tryAcquire 简言之就是有占用返回false 无占用并成功占用资源 返回true
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();//没有线程占用就是0 有线程占用就是1
            //若没有被占用，当前线程直接占用
            if (c == 0) {
                //如果当前线程是在队首 或队列为空，就进行cas占用资源，cas占用成功直接将资源占有者设置为当前线程
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            //当前线程和已持有资源的线程是否相同
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
```

**Step2**：如果tryAcquir方法返回false，也就表示尝试获取资源失败。`!tryAcquire(arg)`就是true,这时`acquire`方法会先调用`addWaiter`方法将当前线程封装成`Node`类并加入一个FIFO的双向队列的尾部。  

```java
    //加入队列
    private Node addWaiter(Node mode) {
        // mode为 Node.EXCLUSIVE 独占模式
        // 以当前线程 独占模式，创建新节点
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        // 获取队列尾 ，用于下面判断队列是否为空  
        //也就是当队列不为空，已经有线程进入等待时，再有线程尝试加入队列 
        // A占用线程 B在队首 C线程再进来就进入这个方法
        Node pred = tail;
        if (pred != null) {
            //将当前节点的前指针指向队尾节点
            node.prev = pred;
            // CAS 设置当前节点为 队尾节点
            if (compareAndSetTail(pred, node)) {
                //将尾节点的尾指针设置为当前节点
                pred.next = node;
                return node;
            }
        }
        //基本理解为 如果等待队列为空或者上述CAS失败，再自旋CAS插入
        enq(node);
        return node;
    }
    
    //自旋CAS插入等待队列
    private Node enq(final Node node) {
        //死循环
        for (;;) {
            Node t = tail;
            //再次判断尾节点，如果为空就新建一个空节点作为头节点(CAS)，这是队列的第一个节点(虚拟节点、哨兵节点，用于占位)
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    //此时头尾都是这个节点
                    tail = head;
            } else {
                //再次进入 将node的前指针指向上面刚创建的虚拟节点，入队
                node.prev = t;
                // CAS 将node节点设置为尾节点
                if (compareAndSetTail(t, node)) {
                    // 将虚拟节点的尾指针指向node节点
                    t.next = node;
                    return t;
                }
            }
        }
    }
```

**Step3**：再看acquireQueued这个关键方法。首先要注意的是这个方法中哪个无条件的for循环，这个for循环说明acquireQueued方法一直在自旋尝试获取资源。进入for循环后，首先判断了当前节点的前继节点是不是头节点，如果是的话就再次尝试获取资源，获取资源成功的话就直接返回false（表示未被中断过）   
假如还是没有获取资源成功，判断是否需要让当前节点进入waiting状态，经过 `shouldParkAfterFailedAcquire`这个方法判断，如果需要让线程进入waiting状态的话，就调用LockSupport的park方法让线程进入waiting状态。进入waiting状态后，这线程等待被interupt或者unpark。

```java
    // node 刚入队的node arg = 1
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            // 自旋
            for (;;) {
                // 获取前一个节点，若是队首节点获取的就是虚拟节点 
                final Node p = node.predecessor();
                // 如果 前一个节点是头节点，再次尝试抢占资源
                 // 如果node的前驱结点p是head，表示node是第二个结点，就可以尝试去获取资源了
                if (p == head && tryAcquire(arg)) {
                    // 拿到资源后，将head指向该结点。
                    // 所以head所指的结点，就是当前获取到资源的那个结点或null。
                    setHead(node);
                    //将头节点彻底断开 方便垃圾回收
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                // 如果自己可以休息了，就进入waiting状态，直到被unpark()
                if (shouldParkAfterFailedAcquire(p, node) &&
                    // locksupport.park() 等待
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

**Step4**：进入释放阶段由unpark 调用`release(1)`方法(AQS方法),

```java
//AQS 方法
    public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
            //唤醒下一个线程
                unparkSuccessor(h);
            return true;
        }
        return false;
    }

   // Sync 静态内部类方法  
        protected final boolean tryRelease(int releases) {
            //若对正在在占用的线程进行释放就是 1-1 = 0
            int c = getState() - releases;
            //判断当前线程和占有资源线程是否相同
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                //将占用线程信息清空
                setExclusiveOwnerThread(null);
            }
            // 重置锁状态
            setState(c);
            return free;
        }

private void unparkSuccessor(Node node) {
    // 如果状态是负数，尝试把它设置为0
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);
    // 得到头结点的后继结点head.next
    Node s = node.next;
    // 如果这个后继结点为空或者状态大于0
    // 通过前面的定义我们知道，大于0只有一种可能，就是这个结点已被取消
    if (s == null || s.waitStatus > 0) {
        s = null;
        // 等待队列中所有还有用的结点，都向前移动
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    // 如果后继结点不为空，
    if (s != null)
        LockSupport.unpark(s.thread);
}
 
```

**Step5**： 取消流程`cancelAcquire(node);`




# 工具 
## 线程池
ThreadPool 线程池维护着多个线程，是一种线程使用模式

特点：
- 降低资源消耗，线程复用
- 控制并发数量
- 可以对线程做统一管理

### 构建线程池
- `Executors.newFixedThreadPool(int)` 固定数量的线程
- `Executors.newSingleThreadExecutor()` 只有一个线程,顺序执行
- `Executors.newCachedThreadPool()` 可扩容的线程池,空闲线程默认保留60s

**上述线程池创建都是用`ThreadPoolExecutor()方法` 搭配不同参数进行创建**  
- `Executors.newScheduledThreadPool(int)` 创建一个定长线程池，支持定时及周期性任务执行

```java
public class ThreadPoolDemo1 {
    public static void main(String[] args) {
        //指定线程数量的线程池
        ExecutorService threadPool1 = Executors.newFixedThreadPool(5);
        //单线程的线程池
        ExecutorService executorService = Executors.newSingleThreadExecutor();
        //可扩容的
        ExecutorService threadPool = Executors.newCachedThreadPool();

        try {
            for (int i = 0; i < 10; i++) {
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName()+"正在办理");
                });
            }
        }finally {
            threadPool.shutdown();
        }
    }
}
```

### ThreadPoolExecutor七(5+2)个参数解读

线程池一共有四种构造方法，分别是最基本的5个和额外两个组合

```java
//源码
 public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
    }
```
五个必要参数:
- `corePoolSize` 线程池中**核心线程**数最大值
    > 核心线程：线程池中有两类线程，核心线程和非核心线程。核心线程默认情况下会一直存在于线程池中，即使这个核心线程什么都不干（铁饭碗），而非核心线程如果长时间的闲置，就会被销毁（临时工）
- `maximumPoolSize` 该线程池中**线程总数**最大值 
    > 该值等于核心线程数量 + 非核心线程数量。
- `keepAliveTime`  `unit` 非核心线程闲置超时时长 和 时间单位(TimeUnit是一个枚举类型)
- `BlockingQueue<Runnable> workQueue` 阻塞队列，维护着等待执行的Runnable任务对象

下面是两个非必要参数
- `ThreadFactory threadFactory` 线程工厂
    > 用于批量创建线程，统一在创建线程时设置一些参数，如是否守护线程、线程的优先级等。如果不指定，会新建一个默认的线程工厂
- `RejectedExecutionHandler handler` 拒绝策略
    > 线程数量大于最大线程数就会采用拒绝处理策略

### 线程池底层工作流程  
- 创建线程池,线程并没有创建
- 执行线程池的`execute()` 方法后线程才会创建 
- 先到到核心线程,核心线程满载后进入阻塞队列,阻塞队列满了后,创建新线程进行处理,线程总数到达最大线程数时,后续请求会直接执行拒绝策略


### 四种拒绝策略

- `ThreadPoolExecutor.AbortPolicy` 默认处理策略,丢弃任务并抛出 `RejectedExecutionException` 异常,系统会停止运行
- `ThreadPoolExecutor.CallerRunsPolicy`：由调用线程处理该任务,回退到调用者执行任务
- `ThreadPoolExecutor.DiscardOldestPolicy`：丢弃队列头部（最旧的）的任务，然后重新尝试执行程序（如果再次失败，重复此过程）
- `ThreadPoolExecutor.DiscardPolicy`：丢弃新来的任务，但是不抛出异常。


### 自定义线程池
实际应用中,不建议使用 `Executors` 创建线程池,可能会造成OOM,而是通过 `ThreadPoolExecutor` 方式自定义线程池参数  
- FixedThreadPool 和 SingleThreadPool： 允许的请求队列长度为 `Integer.MAX_VALUE`，可能会堆积大量的请求，从而导致 OOM。
- CachedThreadPool： 允许的创建线程数量为 `Integer.MAX_VALUE`，可能会创建大量的线程，从而导致 OOM


```java
public class ThreadPoolDemo2 {
    public static void main(String[] args) {
        ExecutorService threadPool = new ThreadPoolExecutor(
                2, 5, 2, TimeUnit.SECONDS, new ArrayBlockingQueue<>(3),
                Executors.defaultThreadFactory(), new ThreadPoolExecutor.AbortPolicy()
        );
        try {
            for (int i = 0; i < 10; i++) {
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName()+"正在办理");
                });
            }
        }finally {
            threadPool.shutdown();
        }
    }
}
```

## 阻塞队列
假设一种场景，生产者一直生产资源，消费者一直消费资源，资源存储在一个缓冲池中，生产者将生产的资源存进缓冲池中，消费者从缓冲池中拿到资源进行消费，这就是大名鼎鼎的`生产者-消费者模式`

JDK提供了阻塞队列(BlockingQueue)，实现了生产消费模式，而不用担心多线程环境下存、取共享变量的线程安全问题  

`BlockingQueue`是`Java util.concurrent`包下重要的数据结构，区别于普通的队列，`BlockingQueue`提供了线程安全的队列访问方式，并发包下很多高级同步类的实现都是基于`BlockingQueue`实现的 

### 说明 
当阻塞队列是空时，从队列中获取元素的操作将会被阻塞。  
当阻塞队列是满时，往队列里添加元素的操作将会被阻塞。    
我们不需要关心什么时候需要阻塞线程，什么时候需要唤醒线程，因为这一切BlockingQueue都给你一手包办了

### 分类
- ArrayBlockingQueue：由数组结构组成的有界阻塞队列。
  - 可以初始化队列大小， 且一旦初始化不能改变。构造方法中的fair表示控制对象的内部锁是否采用公平锁，默认是非公平锁
- LinkedBlockingQueue：由链表结构组成的有界（但大小默认值为`Integer.MAX_VALUE`）阻塞队列。
  - 内部结构是链表，具有链表的特性。默认队列的大小是Integer.MAX_VALUE，也可以指定大小。此队列按照先进先出的原则对元素进行排序
- PriorityBlockingQueue：支持优先级排序的无界阻塞队列。
- DelayQueue：使用优先级队列实现妁延迟无界阻塞队列。
- SynchronousQueue：不存储元素的阻塞队列。
  - 没有任何内部容量，甚至连一个队列的容量都没有。并且每个 put 必须等待一个 take，反之亦然
- LinkedTransferQueue：由链表结构绒成的无界阻塞队列。
- LinkedBlockingDeque：由链表结构组成的双向阻塞队列。

### 核心方法 
方法类型 | 抛出异常 | 特殊值| 阻塞| 超时
---------|----------|---------|---------|---------
 插入 | `add(e)` | `offer(e)`| `put(e)`| `offer(e,time,unit)`
 移除 | `remove()` | `poll()`| `take()`| `poll(time,unit)`
 检查 | `element()` | `peek()`| 不可用| 不可用

上表的说明  
- 抛出异常：如果试图的操作无法立即执行，抛异常。当阻塞队列满时候，再往队列里插入元素，会抛出IllegalStateException(“Queue full”)异常。  
  - 当队列为空时，从队列里获取元素时会抛出NoSuchElementException异常 。
- 返回特殊值：如果试图的操作无法立即执行，返回一个特殊值，通常是true / false。
- 一直阻塞：如果试图的操作无法立即执行，则一直阻塞或者响应中断。
- 超时退出：如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行，但等待时间不会超过给定值。返回一个特定值以告知该操作是否成功，通常是 true / false。

```java
//抛出异常
public class BlockingQueueExceptionDemo {

	public static void main(String[] args) {
		BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);

		System.out.println(blockingQueue.add("a"));
		System.out.println(blockingQueue.add("b"));
		System.out.println(blockingQueue.add("c"));

		try {
			//抛出 java.lang.IllegalStateException: Queue full
			System.out.println(blockingQueue.add("XXX"));
		} catch (Exception e) {
			System.err.println(e);
		}
		
		System.out.println(blockingQueue.element());
		
		///
		
		System.out.println(blockingQueue.remove());
		System.out.println(blockingQueue.remove());
		System.out.println(blockingQueue.remove());
		
		try {
			//抛出 java.util.NoSuchElementException
			System.out.println(blockingQueue.remove());			
		} catch (Exception e) {
			System.err.println(e);
		}

		try {
			//element()相当于peek(),但element()会抛NoSuchElementException
			System.out.println(blockingQueue.element());
		} catch (Exception e) {
			System.err.println(e);
		}
		
	}
}
// true
// true
// true
// a
// java.lang.IllegalStateException: Queue full
// a
// b
// c
// java.util.NoSuchElementException
// java.util.NoSuchElementException

```

```java
//特殊值
public class BlockingQueueBooleanDemo {

	public static void main(String[] args) {
		BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);

		System.out.println(blockingQueue.offer("a"));
		System.out.println(blockingQueue.offer("b"));
		System.out.println(blockingQueue.offer("c"));
		System.out.println(blockingQueue.offer("d"));

		System.out.println(blockingQueue.poll());
		System.out.println(blockingQueue.poll());
		System.out.println(blockingQueue.poll());
		System.out.println(blockingQueue.poll());
	}
}
// true
// true
// true
// false
// a
// b
// c
// null
```

```java
//阻塞
public class BlockingQueueBlockedDemo {

	public static void main(String[] args) throws InterruptedException {
		BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
		
		new Thread(()->{
			try {
				blockingQueue.put("a");
				blockingQueue.put("b");
				blockingQueue.put("c");
				blockingQueue.put("c");//将会阻塞,直到主线程take()
				System.out.println("it was blocked.");

			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}).start();
		
		TimeUnit.SECONDS.sleep(2);
		
		try {
	
			blockingQueue.take();
			blockingQueue.take();
			blockingQueue.take();
			blockingQueue.take();
			
			System.out.println("Blocking...");
			blockingQueue.take();//将会阻塞
			
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}

}

```

```java
//超时
public class BlockingQueueTimeoutDemo {

	public static void main(String[] args) throws InterruptedException {
		BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
		
		System.out.println("Offer.");
		System.out.println(blockingQueue.offer("a", 2L, TimeUnit.SECONDS));
		System.out.println(blockingQueue.offer("b", 2L, TimeUnit.SECONDS));
		System.out.println(blockingQueue.offer("c", 2L, TimeUnit.SECONDS));
        //无法进入队列，等2秒后放弃打印false 继续向下执行
		System.out.println(blockingQueue.offer("d", 2L, TimeUnit.SECONDS));
		
		System.out.println("Poll.");
		System.out.println(blockingQueue.poll(2L, TimeUnit.SECONDS));
		System.out.println(blockingQueue.poll(2L, TimeUnit.SECONDS));
		System.out.println(blockingQueue.poll(2L, TimeUnit.SECONDS));
		System.out.println(blockingQueue.poll(2L, TimeUnit.SECONDS));
	}

}
// Offer.
// true
// true
// true
// false
// Poll.
// a
// b
// c
// null
```

### SynchronousQueue队列
SynchronousQueue没有容量。   
与其他BlockingQueue不同，SynchronousQueue是一个不存储元素的BlockingQueue。  
每一个put操作必须要等待一个take操作，否则不能继续添加元素，反之亦然。  

```java

public class SynchronousQueueDemo {
	public static void main(String[] args) {
		BlockingQueue<String> blockingQueue = new SynchronousQueue<>();

		new Thread(() -> {
		    try {       
		        System.out.println(Thread.currentThread().getName() + "\t put A ");
		        blockingQueue.put("A");
		       
		        System.out.println(Thread.currentThread().getName() + "\t put B ");
		        blockingQueue.put("B");        
		        
		        System.out.println(Thread.currentThread().getName() + "\t put C ");
		        blockingQueue.put("C");        
		        
		    } catch (InterruptedException e) {
		        e.printStackTrace();
		    }
		}, "t1").start();
		
		new Thread(() -> {
			try {
				
				try {
					TimeUnit.SECONDS.sleep(5);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				blockingQueue.take();
				System.out.println(Thread.currentThread().getName() + "\t take A ");
				
				try {
					TimeUnit.SECONDS.sleep(5);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				blockingQueue.take();
				System.out.println(Thread.currentThread().getName() + "\t take B ");
				
				try {
					TimeUnit.SECONDS.sleep(5);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				blockingQueue.take();
				System.out.println(Thread.currentThread().getName() + "\t take C ");
				
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}, "t2").start();
	}
	
}
// t1	 put A 
// t2	 take A 
// t1	 put B 
// t2	 take B 
// t1	 put C 
// t2	 take C 
```

##  同步容器与并发容器
`java.util`包下提供了一些容器类，而`Vector`和`Hashtable`是线程安全的容器类，但是这些容器实现同步的方式是通过对方法加锁(sychronized)方式实现的，这样读写均需要锁操作，导致性能低下

### 并发容器介绍
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/并发容器汇总关系图.png)

### ConcurrentMap接口
ConcurrentMap接口继承了Map接口，在Map接口的基础上又定义了四个方法：
- `putIfAbsent`：与原有put方法不同的是，putIfAbsent方法中如果插入的key相同，则不替换原有的value值；
- `remove`：与原有remove方法不同的是，新remove方法中增加了对value的判断，如果要删除的key-value不能与Map中原有的key-value对应上，则不会删除该元素;
- `replace(K,V,V)`：增加了对value值的判断，如果key-oldValue能与Map中原有的key-value对应上，才进行替换操作；
- `replace(K,V)`：与上面的replace不同的是，此replace不会对Map中原有的key-value进行比较，如果key存在则直接替换；

### ConcurrentHashMap类
`ConcurrentHashMap`同HashMap一样也是基于散列表的map，但是它提供了一种与Hashtable完全不同的加锁策略，提供更高效的并发性和伸缩性
ConcurrentHashMap在JDK 1.7 和JDK 1.8中有一些区别  

#### JDK 1.7
ConcurrentHashMap在JDK 1.7中，提供了一种粒度更细的加锁机制来实现在多线程下更高的性能，这种机制叫分段锁(Lock Striping)    
提供的优点是：在并发环境下将实现更高的吞吐量，而在单线程环境下只损失非常小的性能。

可以这样理解`分段锁`，就是将数据分段，**对每一段数据分配一把锁。当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问**

有些方法需要跨段，比如`size()`、`isEmpty()`、`containsValue()`，它们可能需要锁定整个表而不仅仅是某个段，这需要按顺序锁定所有段，操作完毕后，又按顺序释放所有段的锁

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/分段锁机制.png)

`ConcurrentHashMap`是由`Segment`数组结构和`HashEntry`数组结构组成。`Segment`是一种可重入锁`ReentrantLock`，`HashEntry`则用于存储键值对数据

一个ConcurrentHashMap里包含一个Segment数组，Segment的结构和HashMap类似，是一种数组和链表结构， 一个Segment里包含一个HashEntry数组，每个HashEntry是一个链表结构的元素， 每个Segment守护着一个HashEntry数组里的元素，当对HashEntry数组的数据进行修改时，必须首先获得它对应的Segment锁

#### JDK 1.8

而在JDK 1.8中，ConcurrentHashMap主要做了两个优化：

同HashMap一样，链表也会在长度达到8的时候转化为红黑树，这样可以提升大量冲突时候的查询效率；
以某个位置的头结点（链表的头结点或红黑树的root结点）为锁，配合自旋+CAS避免不必要的锁开销，进一步提升并发性能


### ConcurrentNavigableMap接口与ConcurrentSkipListMap类
ConcurrentNavigableMap接口继承了NavigableMap接口，这个接口提供了针对给定搜索目标返回最接近匹配项的导航方法。

ConcurrentNavigableMap接口的主要实现类是ConcurrentSkipListMap类。从名字上来看，它的底层使用的是跳表（SkipList）的数据结构。关于跳表的数据结构这里不做太多介绍，它是一种”空间换时间“的数据结构，可以使用CAS来保证并发安全性。

## CopyOnWrite容器
CopyOnWrite容器即写时复制的容器,当我们往一个容器中添加元素的时候，不直接往容器中添加，而是将当前容器进行copy，复制出来一个新的容器，然后向新容器中添加我们需要的元素，最后将原容器的引用指向新容器。

这样就可以在并发的场景下对容器进行"读操作"而不需要"加锁"，从而达到读写分离的目的。从JDK 1.5 开始Java并发包里提供了两个使用CopyOnWrite机制实现的并发容器 ，分别是`CopyOnWriteArrayList`和`CopyOnWriteArraySet`  


### CopyOnWriteArrayList
`CopyOnWriteArrayList`是`ArrayList` 的一个线程安全的变体，其中所有可变操作（add、set等等）都是通过对底层数组进行一次新的复制来实现的
`CopyOnWriteArrayList`经常被用于`读多写少`的并发场景，是因为`CopyOnWriteArrayList`无需任何同步措施，大大增强了读的性能。   

在Java中遍历线程非安全的List(如：ArrayList和 LinkedList)的时候，若中途有别的线程对List容器进行修改，那么会抛出ConcurrentModificationException异常。**CopyOnWriteArrayList由于其"读写分离"，遍历和修改操作分别作用在不同的List容器，所以在使用迭代器遍历的时候，则不会抛出异常**


**缺点：**   
- 第一个缺点是CopyOnWriteArrayList每次执行写操作都会将原容器进行拷贝一份，数据量大的时候，内存会存在较大的压力，可能会引起频繁Full GC（ZGC因为没有使用Full GC）。比如这些对象占用的内存200M左右，那么再写入100M数据进去，内存就会多占用300M。
- 第二个缺点是CopyOnWriteArrayList由于实现的原因，写和读分别作用在不同新老容器上，在写操作执行过程中，读不会阻塞，但读取到的却是老容器的数据。

现在我们来看一下CopyOnWriteArrayList的add操作源码，它的逻辑很清晰，就是先把原容器进行copy，然后在新的副本上进行“写操作”，最后再切换引用，在此过程中是加了锁的。

```java
public boolean add(E e) {

    // ReentrantLock加锁，保证线程安全
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        // 拷贝原容器，长度为原容器长度加一
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        // 在新副本上执行添加操作
        newElements[len] = e;
        // 将原容器引用指向新副本
        setArray(newElements);
        return true;
    } finally {
        // 解锁
        lock.unlock();
    }
}
```

再来看一下remove操作的源码，remove的逻辑是将要remove元素之外的其他元素拷贝到新的副本中，然后再将原容器的引用指向新的副本中，因为remove操作也是“写操作”所以也是要加锁的

```java
public E remove(int index) {

        // 加锁
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            E oldValue = get(elements, index);
            int numMoved = len - index - 1;
            if (numMoved == 0)
                // 如果要删除的是列表末端数据，拷贝前len-1个数据到新副本上，再切换引用
                setArray(Arrays.copyOf(elements, len - 1));
            else {
                // 否则，将要删除元素之外的其他元素拷贝到新副本中，并切换引用
                Object[] newElements = new Object[len - 1];
                System.arraycopy(elements, 0, newElements, 0, index);
                System.arraycopy(elements, index + 1, newElements, index,
                                 numMoved);
                setArray(newElements);
            }
            return oldValue;
        } finally {
            // 解锁
            lock.unlock();
        }
    }
```

CopyOnWriteArrayList的读操作的源码,就是直接读取，没有加锁

```java
public E get(int index) {
    return get(getArray(), index);
}
 private E get(Object[] a, int index) {
     return (E) a[index];
 }
```

## Atomic 原子操作类   
JDK提供了一些用于原子操作的类，在`java.util.concurrent.atomic`包下面

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/atomic原子类集合1.png)   

[上图API地址](https://www.runoob.com/manual/jdk11api/java.base/java/util/concurrent/atomic/package-summary.html)

针对其作用对象不同分为五大类： 
- 基本类型原子类
- 数组类型原子类
- 引用类型原子类
- 对象的属性修改原子类
- 原子操作增强类

### 基本类型原子类
`AtomicInteger`   
`AtomicBoolean`  
`AtomicLong`  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/AtomicInteger常用API.png)

### 数组类型原子类
- `AtomicIntegerArray`  
- `AtomicLongArray`  
- `AtomicReferenceArray`    

```java
public class AtomicIntegerArrayTest {
    public static void main(String[] args) {
        AtomicIntegerArray atomicIntegerArray = new AtomicIntegerArray(new int[5]);
        for (int i = 0; i < atomicIntegerArray.length(); i++) {
            System.out.println(atomicIntegerArray.get(i));//全是0 默认值
        }
        AtomicIntegerArray array = new AtomicIntegerArray(new int[]{1, 2, 3, 4, 5});
        array.getAndAdd(0, 999);
        System.out.println(array.get(0));//1+999 输出1000
        array.getAndIncrement(4);
        System.out.println(array.get(4)); // 拿到index=4 的值并增加 1 返回  输出5 
    }
}

```

### 引用类型原子类
- `AtomicReference`    
- `AtomicStampedReference` 采用integer版本号，判断多次修改
- `AtomicMarkableReference`采用Boolean，只能判断一次是否修改过(true/false)

### 对象的属性修改原子类
`AtomicIntegerFieldUpdater<T>`基于反射的实用程序，可对指定类的指定 `volatile int字段`进行原子更新。    
`AtomicLongFieldUpdater<T>`基于反射的实用程序，可以对指定类的指定 `volatile long字段`进行原子更新。   
`AtomicReferenceFieldUpdater<T,​V>`基于反射的实用程序，可以对指定类的指定 `volatile引用字段`进行原子更新。   

作用：以一种线程安全的方式操作非线程安全对象内的某些字段

```java
//AtomicIntegerFieldUpdater 实例
public class Test {
    public static void main(String[] args) throws InterruptedException {
        BankAccount bankAccount = new BankAccount();
        CountDownLatch countDownLatch = new CountDownLatch(10);
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                try {

                    for (int j = 1; j <= 1000; j++) {
                        bankAccount.addMoney();
                        bankAccount.addMoney1();
                        bankAccount.addMoney2();
                    }
                } finally {
                    countDownLatch.countDown();
                }
            }, String.valueOf(i)).start();
        }
        countDownLatch.await();
        System.out.println("未加锁的结果"+bankAccount.money);//每次执行结果不同 9976
        System.out.println("加锁的结果"+bankAccount.money1);// 10000
        System.out.println("使用原子类修改属性"+bankAccount.money2);//10000
    }
}

class BankAccount {
    String bankName = "ICBC";
    String bankNo = "12345";
    int money = 0;
    int money1 = 0;
    //必须用volatile
    volatile int money2 = 0;

    //创建更新器 选定字段
    AtomicIntegerFieldUpdater<BankAccount> updater = AtomicIntegerFieldUpdater.newUpdater(BankAccount.class, "money2");

    //使用原子类
    public void addMoney2(){
        updater.getAndIncrement(this);
    }
    //加锁
    public synchronized void addMoney1() {
        money1++;
    }
    //未处理
    public void addMoney() {
        money++;
    }
}
```

```java
//AtomicReferenceFieldUpdater案例
//多线程并发调用一个类的初始化方法，只能初始化一次，只能有一个线程操作成功  
class Fish {
    volatile Boolean isInit = Boolean.FALSE;
    AtomicReferenceFieldUpdater<Fish, Boolean> updater = AtomicReferenceFieldUpdater.newUpdater(Fish.class, Boolean.class, "isInit");

    public void init() {
        if (updater.compareAndSet(this, Boolean.FALSE, Boolean.TRUE)) {
            System.out.println("初始化的线程：" + Thread.currentThread().getName());
        } else {
            System.out.println("==========已被其他线程初始化=============");
        }
    }

    public static void main(String[] args) {
        Fish fish = new Fish();
        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                fish.init();
            }).start();
        }
    }
}
```

### LongAdder 原子操作增强类  
JDK1.8时，`java.util.concurrent.atomic`包中提供了一个新的原子类：`LongAdder`    
`LongAdder`在高并发的场景下会比它的前辈————`AtomicLong`**具有更好的性能（减少乐观锁的重试次数），代价是消耗更多的内存空间**  


- `LongAdder`一个或多个变量共同维持`最初为零`的Long总和。
- `LongAccumulator`一个或多个变量共同维护使用提供的函数更新的运行 long值。
- `DoubleAdder`一个或多个变量共同维持`最初的零`的double总和。
- `DoubleAccumulator`一个或多个变量共同维护使用提供的函数更新的运行 double值。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/Longadder的继承关系.png)

> 在并发量较低的环境下，线程冲突的概率比较小，自旋的次数不会很多，二者相差不大。但是，高并发环境下，N个线程同时进行自旋操作，会出现大量失败并不断自旋的情况，此时AtomicLong的自旋会成为瓶颈。   
> 这就是LongAdder引入的初衷——解决高并发环境下`AtomicLong`的自旋瓶颈问题   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/LongAdder方法摘要.png)

- `LongAdder`只能用来计算加减法，且从零开始计算
- `LongAccumulator`提供了自定义的函数操作

#### 简单案例  
一般用做实时性要求不高的高并发统计，如点赞数  

```java
class Test3{
    public static void main(String[] args) {
        LongAdder longAdder = new LongAdder();
        longAdder.add(3L);//0+3 = 3
        longAdder.add(5L);//3+5 = 8
        longAdder.increment();//8+1 = 9
        System.out.println(longAdder.sum());//9
        longAdder.decrement();
        System.out.println(longAdder.sum());//8

        LongAccumulator longAccumulator = new LongAccumulator((x,y)->x+y,0);
        longAccumulator.accumulate(6);//6
        longAccumulator.accumulate(2);//8
        System.out.println(longAccumulator.get());//8

        LongAccumulator longAccumulator1 = new LongAccumulator(new LongBinaryOperator() {
            @Override
            public long applyAsLong(long left, long right) {
                return left + right;
            }
        }, 2);//起始值是2
        longAccumulator1.accumulate(2);  // 2+2
        longAccumulator1.accumulate(3); // 4+3
        System.out.println(longAccumulator1.get());// 7 
    }
}

```

#### LongAdder为什么这么快?

> `LongAdder`的基本思路就是`分散热点`，将value值分散到一个Cell数组中，不同线程会命中到数组的不同槽中，各个线程只对自己槽中的那个值进行CAS操作，这样热点就被分散了，冲突的概率就小很多。如果要获取真正的long值，只要将各个槽中的变量值累加返回即可      
> `sum()`会将所有`Cell数组`中的`value`和`base`累加作为返回值，核心的思想就是将之前AtomicLong一个value的更新压力分散到多个value中去，从而降级更新热点


```java
   /**
     * Table of cells. When non-null, size is a power of 2.
     */
    transient volatile Cell[] cells;

    /**
     * Base value, used mainly when there is no contention, but also as
     * a fallback during table initialization races. Updated via CAS.
     */
    transient volatile long base;

    /**
     * Spinlock (locked via CAS) used when resizing and/or creating Cells.
     */
    transient volatile int cellsBusy;

```

只有从未出现过并发冲突的时候，才会使用到base基数，修改值直接累加到base上   
一旦出现了并发冲突，之后所有的操作都只针对Cell[]数组中的单元Cell,各个线程将值累加到各自的槽中，sum时会对槽中所有值进行累加。     

#### 源码分析  
LongAdder 在无竞争的情况下，跟AtomicLong一样，对同一个base进行累加操作，当出现竞争的时候就采用`分散热点`的方法，用空间换时间，用一个数组Cell 将多个线程的累加值放在不同的数组元素中，可以对线程id进行hash 根据hash值映射到这个数组Cell的某个下标，再对该下标中的值进行自增操作，当所有线程操作完毕，将数组Cell中的所有值和base都加起来作为最终结果`sum()`

参考资料中   
P92-P98 暂略




## CompletableFuture
JDK8加入了`CompletableFuture`,提供了一种观察者模式类似的机制，可以让任务执行完成后通知监听的一方    
`CompletableFuture` 实现了 `Future`, `CompletionStage` 接口，实现了 Future接口就可以兼容现在有线程池框架，而 `CompletionStage` 接口才是异步编程的接口抽象，里面定义多种异步方法(`CompletableFuture `大部分方法来自`CompletionStage`接口)，通过这两者集合，从而打造出了强大的 `CompletableFuture` 类

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/CompletableFuture依赖图.png)

### 构造 CompletableFuture

通过两组四种静态方法构造一个`CompletableFuture`，不推荐使用new 构造  

```java
//runAsync 不带返回值
public static CompletableFuture<Void> runAsync(Runnable runnable) {
    return asyncRunStage(asyncPool, runnable);
}
//指定线程池
public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor) {
     return asyncRunStage(screenExecutor(executor), runnable);
}


//supplyAsync 带返回值
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier) {
    return asyncSupplyStage(asyncPool, supplier);
}
//指定线程池
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier,Executor executor) {
    return asyncSupplyStage(screenExecutor(executor), supplier);
}
```

### 简单调用

```java
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(4);

        CompletableFuture<Void> voidCompletableFuture = CompletableFuture.runAsync(() -> {
            System.out.println(Thread.currentThread().getName());
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },executorService);
        System.out.println(voidCompletableFuture.get());//null

        CompletableFuture<String> voidCompletableFuture1 = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName());
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "ddd";
        },executorService);
        System.out.println(voidCompletableFuture1.get());//ddd
    }
```

###  CompletableFuture常用方法  

#### join get
join()和get()方法都是用来获取`CompletableFuture`异步之后的返回值    

`join()`方法抛出的是uncheck异常（即RuntimeException),不会强制开发者抛出，会将异常包装成`CompletionException`异常 `CancellationException`异常，不需要调用者抛出或trycatch异常

`get()`方法抛出的是经过检查的异常，ExecutionException, InterruptedException 需要用户(调用者)手动处理（抛出或者 try catch）


#### 获得结果和触发计算
```java

public T get() 
public T get(long timeout,TimeUnit unit) //带超时时间的get
public T join()
public T getNow(T valuelfAbsent)//调用时立刻获取，如果没有值就返回默认值，即设定的valuelfAbsent值
public boolean complete(T value) // 返回true 打算了get过程(理解为超时)，返回value值，返回false成功get 返回get的值

```

#### 对计算结果进行处理

**thenApply**把上一个任务返回结果当做入参，执行结束将会返回结果  
**thenRun**不需要上一步的结果，执行结束无返回结果  

```java
//在之前的基础上继续处理，把上一个任务返回结果当做入参，执行结束将会返回结果
public <U> CompletionStage<U> thenApply(Function<? super T,? extends U> fn);
//方法中带有 Async ，代表可以异步执行
public <U> CompletionStage<U> thenApplyAsync(Function<? super T,? extends U> fn);
//指定线程池
public <U> CompletionStage<U> thenApplyAsync
        (Function<? super T,? extends U> fn,
         Executor executor);

//thenRun同理 

```

`CompletableFuture` 方法执行过程若产生异常，当调用 `get，join`获取任务结果才会抛出异常   
显示使用 `try..catch` 处理异常。不过这种方式不太优雅，CompletionStage 提供几个方法，可以优雅处理异常。  

```java
// 同步
CompletableFuture<T> exceptionally(Function<Throwable, ? extends T> fn)
CompletableFuture<T> whenComplete(BiConsumer<T, ? super Throwable> action)
CompletableFuture<T> handle(BiFunction<T, Throwable, ? extends U> fn)
 
// 异步
CompletableFuture<T> whenCompleteAsync(BiConsumer<T, ? super Throwable> action)
CompletableFuture<T> handleAsync(BiFunction<T, Throwable, ? extends U> fn)
```

`exceptionally` 使用方式类似于 `try..catch` 中 `catch`代码块中异常处理。
`whenComplete` 与 `handle` 方法就类似于 `try..catch..finanlly` 中 `finally` 代码块。无论是否发生异常，都将会执行的。这两个方法区别在于`handle`支持返回结果。   

`whenComplete` 与 `handle`同时可以用于中间值的处理

```java
// 同步
CompletableFuture<T> exceptionally(Function<Throwable, ? extends T> fn)
CompletableFuture<T> whenComplete(BiConsumer<T, ? super Throwable> action)
CompletableFuture<T> handle(BiFunction<T, Throwable, ? extends U> fn)
 
// 异步
CompletableFuture<T> whenCompleteAsync(BiConsumer<T, ? super Throwable> action)
CompletableFuture<T> handleAsync(BiFunction<T, Throwable, ? extends U> fn)
```

```java
public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
        ExecutorService executorService = Executors.newFixedThreadPool(2);
        CompletableFuture.supplyAsync(() -> {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return 6;
        }, executorService).handle((r, e) -> {
            System.out.println(r);
            int i = 2 / 0;//异常
            System.out.println(r);//出异常后 这里的r就是null了
            return r * 5;
        }).handle((r, e) -> {
            //出异常后这里也是null了
            System.out.println(r);
            return r - 2;
        }).whenComplete((v, e) -> {
            System.out.println("计算结果：" + v);
        }).whenComplete((r, e) -> {
            System.out.println(r);
            System.out.println(r+"12312");
        }).exceptionally(e -> {
            System.out.println(e.getMessage());
            System.out.println(e);
            return null;
        });
        System.out.println("============主线程==========");
        executorService.shutdown();


    }
```

#### 对计算结果进行消费
```java
    public static void main(String[] args) {
        CompletableFuture.supplyAsync(() -> {
            return 3;
        }).thenApply(r -> {
            return r * 8;
        }).thenApply(r -> {
            return r / 2;
        }).thenAccept(r -> System.out.println(r));// 12

        System.out.println(CompletableFuture.supplyAsync(() -> "6666").thenRun(() -> {}).join()); //null

        System.out.println(CompletableFuture.supplyAsync(() -> "6666").thenAccept(r -> System.out.println(r)).join());//6666

        System.out.println(CompletableFuture.supplyAsync(() -> "6666").thenApply(r -> r + "9999").join());//66669999
    }
```

- `thenRun`A执行完执行B，B不需要A的结果
- `thenAccept` A执行完执行B，B需要A的结果，但是任务B无返回值
- `thenApply` A执行完执行B，B需要A的结果，同时任务B有返回值


#### Aysnc的说明
若没有传入自定义线程池，都用默认线程池`ForkJoinPool`;  

传入了一个自定义线程池:
- 如果你执行第一个任务的时候，传入了一个自定义线程池:
- 调用`thenRun`方法执行第二个任务时，则第二个任务和第一个任务是共用同一个线程池。
- 调用`thenRunAsync`执行第二个任务时，则第一个任务使用的是你自己传入的线程池，第二个任务使用的是`ForkJoin线程池`
  - 有可能处理太快，系统优化切换原则，直接使用main线程处理
其它如:`thenAccept`和`thenAcceptAsync`，`thenApply`和`thenApplyAsync`等，它们之间的区别也是同理


#### 对计算速度进行选用、合并
两个任务必须都完成，触发该任务。每个种类都有三个重载方法,分别是当前线程,异步(默认线程池),异步(指定线程池):  
`runAfterBoth(Async)` ：组合两个 future，不需要获取 future 的结果，只需两个 future 处理完任务后，处理该任务  
`thenAcceptBoth(Async)` ：组合两个 future，获取两个 future 任务的返回结果，然后处理任务，没有返回值。  
`thenCombine(Async)` ：组合两个 future，获取两个 future 的返回结果，并返回当前任务的返回值  


当两个任务中，任意一个 future 任务完成的时候，执行任务。每个种类都有三个重载方法,分别是当前线程,异步(默认线程池),异步(指定线程池):  
`applyToEither(Async)` ：两个任务有一个执行完成，获取它的返回值，处理任务并有新的返回值。  
`acceptEither(Async)` ：两个任务有一个执行完成，获取它的返回值，处理任务，没有新的返回值。   
`runAfterEither(Async)` ：两个任务有一个执行完成，不需要获取 future 的结果，处理任务，也没有返回值。  


```java
class CompletableFutureTest4 {
    public static void main(String[] args) {
        CompletableFuture<String> first = CompletableFuture.supplyAsync(() -> {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "1号选手";
        });
        CompletableFuture<String> second = CompletableFuture.supplyAsync(() -> {
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "2号选手";
        });
        //需要参数 需要返回
        CompletableFuture<String> result = first.applyToEither(second, r ->r);
        first.acceptEither(second, r ->{
            //需要参数 不会返回
            System.out.println(r);
        });
        first.runAfterEither(second, () ->{
            System.out.println("不需要参数 不返回");
        });

        CompletableFuture<String> res = first.thenCombine(second, (x, y) -> x + y);
        first.runAfterBoth(second,()->{
            System.out.println("不需要参数 也不返回");
        });
        first.thenAcceptBoth(second,(x,y)->{
            //需要参数 不会返回
            System.out.println(x+y);
        });

        System.out.println("结果"+result.join());
        System.out.println("结果"+res.join());
    }
}
```

## ThreadLocal
ThreadLocal提供了线程局部变量，一个线程局部变量在多个线程中，分别有独立的值（副本），从而避免了线程安全问题

- `ThreadLocal`是一个本地线程副本变量工具类。内部是一个弱引用的Map来维护   
- `ThreadLocal`使得每一个线程都有自己的专属本地变量,线程之间互不影响。它为每个线程都创建一个副本，每个线程可以访问自己内部的副本变量    

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/ThreadLocal的方法.png)

[上图文档地址](https://www.runoob.com/manual/jdk11api/java.base/java/lang/ThreadLocal.html)  

构造函数是一个泛型的，传入的类型是你要使用的局部变量变量的类型。初始化`initialValue()` 用于如果你没有调用`set()`方法的时候，调用 `get()`方法返回的默认值。如果不重载初始化方法，会返回 `null`。 如果调用了`set()`方法，再调用`get()`方法，就不会调用`initialValue()`方法。 如果调用了`set()`，再调用`remove()`，再调用`get()`，是会调用`initialValue()`的  

`withInitial()`在1.8版本新增，创建一个线程局部变量。 通过调用get上的`Supplier`方法确定变量的初始值  
`ThreadLocal<Integer> sal = ThreadLocal.withInitial(() -> 0);` 初始值为0


### 简单案例

```java
public class ThreadLocalDemo {
    static class ThreadA implements Runnable {
        private ThreadLocal<String> threadLocal;

        public ThreadA(ThreadLocal<String> threadLocal) {
            this.threadLocal = threadLocal;
        }

        @Override
        public void run() {
            threadLocal.set("A");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("ThreadA输出：" + threadLocal.get());
        }

        static class ThreadB implements Runnable {
            private ThreadLocal<String> threadLocal;

            public ThreadB(ThreadLocal<String> threadLocal) {
                this.threadLocal = threadLocal;
            }

            @Override
            public void run() {
                threadLocal.set("B");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("ThreadB输出：" + threadLocal.get());
            }
        }

        public static void main(String[] args) {
            ThreadLocal<String> threadLocal = new ThreadLocal<>();
            new Thread(new ThreadA(threadLocal)).start();
            new Thread(new ThreadB(threadLocal)).start();
        }
    }
}

// 输出：
ThreadA输出：A
ThreadB输出：B
```

**虽然两个线程使用的同一个ThreadLocal实例（通过构造方法传入），但是它们各自可以存取自己当前线程的一个值**

### 一般案例  

房产销售买房，用多个线程模拟多个销售员，统计每个销售分别卖出多少 和共计卖出多少  

```java
class House {

    int saleCount = 0;

    public synchronized void saleHouse() {
        ++saleCount;
    }
    //初始化值
    /*ThreadLocal<Integer> sal = new ThreadLocal<Integer>() {
        @Override
        protected Integer initialValue() {
            return 0;
        }
    };*/
    //初始化值 1.8新增静态方法
    ThreadLocal<Integer> sal1 = ThreadLocal.withInitial(() -> 0);
    //threadlocal增加销售量
    public void saleHouseLocal() {
        sal1.set(sal1.get() + 1);
    }

    public static void main(String[] args) {
        House house = new House();
        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                int size = new Random().nextInt(5) + 1;
                try {
                    for (int j = 0; j < size; j++) {
                        house.saleHouse();
                        house.saleHouseLocal();
                    }
                    System.out.println(Thread.currentThread().getName()+"号卖出 "+house.sal1.get());
                }finally {
                    //清除threadlocal
                    house.sal1.remove();
                }
            },String.valueOf(i)).start();
        }
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("共计："+house.saleCount);
    }
}
0号卖出 1
1号卖出 4
4号卖出 4
2号卖出 4
3号卖出 5
共计：18

```

**为什么要清除(回收)ThreadLocal？**   
> **必须回收自定义的ThreadLocal变量**，尤其在`线程池场景`下，线程经常会被复用，如果不清理自定义的ThreadLocal变量，可能会影响后续业务逻辑和造成内存泄露等问题。尽量在代理中使用try-finally块进行回收。 


> ThreadLocal有什么作用呢?   
> 最常见的ThreadLocal使用场景为用来解决数据库连接、Session管理等。数据库连接和Session管理涉及多个复杂对象的初始化和关闭。
> 如果在每个线程中声明一些私有变量来进行操作，那这个线程就变得不那么“轻量”了，需要频繁的创建和关闭连接  

### 源码解析
`ThreadLocal`是一个壳子，真正的存储结构是`ThreadLocal`里有`ThreadLocalMap`静态内部类    
每个`Thread`对象维护着一个`ThreadLocalMap`的引用，`ThreadLocalMap`是`ThreadLocal`的内部类，用`Entry`来进行存储。

- 调用`ThreadLocal`的`set()`方法时， 实际上就是往`ThreadLocalMap`设置值，`key`是`ThreadLocal`对象，值Value是传递进来的对象
- 调用`ThreadLocal`的`get()`方法时， 实际上就是往`ThreadLocalMap`获取值，`key`是`ThreadLocal`对象


```java
public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }

public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }

```

> 每个线程中都有一个 `ThreadLocalMap`数据结构，当执行set方法时，其值是保存在当前线程的 threadLocals变量中，当执行get方法中，是从当前线程的 threadLocals变量获取。 (ThreadLocalMap的key值是ThreadLocal类型)

所以在线程1中set的值，对线程2来说是摸不到的，而且在线程2中重新set的话，也不会影响到线程1中的值，保证了线程之间不会相互干扰。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/ThreadLocal三者关系.png)

```java
public class Thread implements Runnable {

    ...
    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null; 
    ...
}
```

Thread类有属性变量threadLocals （类型是ThreadLocal.ThreadLocalMap），也就是说每个线程有一个自己的ThreadLocalMap ，所以每个线程往这个ThreadLocal中读写隔离的，并且是互相不会影响的。一个ThreadLocal只能存储一个Object对象，如果需要存储多个Object对象那么就需要多个ThreadLocal

### threadlocal的问题

```java
//部分源码
static class ThreadLocalMap {
        //静态内部类entry 继承弱引用 
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                //将k 构造成弱引用
                super(k);
                value = v;
            }
        }
}
```

#### 为什么key使用弱引用  
ThreadlocalMap是和线程绑定在一起的，如果这样线程没有被销毁，而我们又已经不会再使用某个threadlocal引用，那么key-value的键值对就会一直在map中存在，key指向的是ThreadLocal对象(强引用)，导致整个Entry对象不能被回收，就出现了内存泄漏

将key设置为弱引用，那么当发生GC的时候，就会自动将弱引用给清理掉，也就是说：假如某个用户A执行方法时产生了一份threadlocalA，然后在很长一段时间都用不到threadlocalA时，作为弱引用，它会在下次垃圾回收时被清理掉,此时的Entry的key指向null(这里会导致别的问题下面说)  

ThreadLocalMap在内部的set，get和扩容或 remove方法时都会清理掉泄漏的Entry，可以释放value对象所占用的内存

#### 为什么value不适用弱引用
若仍在使用某个threadlocal 此时的key指向threadlocal对象，仍然是强引用，但是value一直是弱引用，此间发生gc会将value直接回收成为null，导致get出来的对象是null

#### ThreadLocal 为什么会内存泄露
之前说过如果不用弱引用可能导致内存泄漏，但是有些情况下在使用了弱引用仍然可能导致内存泄漏   

```java
class Test5{
    public static void main(String[] args) {
                ExecutorService executorService = Executors.newFixedThreadPool(1);
                executorService.execute(() -> {
                    ThreadLocal<Test5> threadLocal = new ThreadLocal<>();
                    Test5 redSpider = new Test5();
                    threadLocal.set(redSpider);
                    //threadLocal=null          //将threadLocal 引用赋值为空
                });
    }
}
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/ThreadLocal内存泄漏.png)

我们在前面介绍`ThreadLocalMap`时已知：`ThreadLocalMap`中，`Entry`的key为 `WeakReference`，当我们给`threadLocal=null`时，逻辑图中`强引用②`会消失，这样 `ThreadLocal`对象实例只有一个 `Entry`中的 `key` 的一个 `弱引用③`。因为弱引用的性质，在下一次 GC 时就会回收 ThreadLocal对象实例。  

这时逻辑图中的引用只剩下 `强引用①` 和 `强引用④`。如果当前线程迟迟不断掉的话(线程不结束)就会一直存在一条强引用链：`thread(ref)->Thread->ThreadLocalMap->Entry->redSpider(ref)`。此时就是ThreadLocal 内存泄露

原因也就找到了：   
- 堆中有一个强引用指向`RedSpider`对象实例，该实例没法被 GC。
- 因为`Entry`的`key`为null ,所以没有任何途径能够接触到`redSpider(ref)`，因此也不能访问到`RedSpider`对象实例


> 虽然弱引用，保证了`key`指向的`ThreadLocal对象`能被及时回收，但是`v`指向的`value对象`是需要ThreadLocalMap调用get、set发现key is null才会去回收整个entry、value    
> **因此弱引用不能100%保证内存不泄露**。我们要在不使用某个ThreadLocal对象后，手动调`remove`方法来删除它。    
> 尤其是在线程池中，不仅仅是内存泄露的问题，因为**线程池中的线租是重复使用的，意味着这个线程的ThreadLocalMap对象也是重复使用的，如果我们不手动调用remove方法，那么后面的线程就有可能获取到上个线程遗留下来的value值**，造成奇怪的bug

#### 回收entry实现
在threadLocal的生命周期里，针对threadLocal存在的内存泄漏的问题，在调用`getEntry` `set` `remove`方法时都会判断key是否为空，若为空就会通过`expungeStaleEntry`，`cleanSomeSlots`,`replaceStaleEntry`这三个方法清理掉key为null的脏entry

```java
//以getEntry方法为例  
        private Entry getEntry(ThreadLocal<?> key) {
            int i = key.threadLocalHashCode & (table.length - 1);
            Entry e = table[i];
            if (e != null && e.get() == key)
                return e;
            else
                return getEntryAfterMiss(key, i, e);
        }

        private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
            Entry[] tab = table;
            int len = tab.length;

            while (e != null) {
                ThreadLocal<?> k = e.get();
                if (k == key)
                    return e;
                //若为空就调用清理方法清除key为null 的entry
                if (k == null)
                    expungeStaleEntry(i);
                else
                    i = nextIndex(i, len);
                e = tab[i];
            }
            return null;
        }
```

#### 解决问题
针对ThreadLocal 内存泄露的原因，我们可以从两方面去考虑：

- 删除无用 Entry 对象，断掉指向ThreadLocal实例的弱引用。即 用完ThreadLocal后手动调用remove()方法。
- **可以让ThreadLocal 的强引用一直存在**，保证任何时候都可以通过 ThreadLocal 的弱引用访问到 Entry的 value值。即 **将ThreadLocal 变量定义为 private static**

### 使用注意事项

- 初始化使用`ThreadLocal.withInitial(() -> 0);`
- 将ThreadLocal 变量定义为 private static,静态变量
- 使用完要进行remove，尤其是使用线程池

### ThreadLocal的使用场景
1. 资源持有：比如我们有三个不同的类。在一次Web请求中，会在不同的地方，不同的时候，调用这三个类的实例。但用户是同一个，用户数据可以保存在一个线程里，我们可以在程序1把用户数据放进ThreadLocalMap里，然后在程序2和程序3里面去用它。 这样做的优势在于：持有线程资源供线程的各个部分使用，全局获取，降低编程难度

2. 线程一致： 我们每次对数据库操作，都会走JDBC getConnection，JDBC保证只要你是同一个线程过来的请求，不管是在哪个part，都返回的是同一个连接。这个就是使用ThreadLocal来做的。 当一个part过来的时候，JDBC会去看ThreadLocal里是不是已经有这个线程的连接了，如果有，就直接返回；如果没有，就从连接池请求分配一个连接，然后放进ThreadLocal里。 这样就可以保证一个事务的所有part都在一个连接里  

3. 线程安全： 假设我们一个线程的调用链路比较长。在中途中出现异常怎么做？我们可以在出错的时候，把错误信息放到ThreadLocal里面，然后在后续的链路去使用这个值。使用TheadLocal可以保证多个线程在处理这个场景的时候保证线程安全

4. 并发计算：和热点分散很像，如果我们有一个大的任务，可以把它拆分成很多小任务，分别计算，然后最终把结果汇总起来，把每个线程的计算结果放进ThreadLocal里面，最后取出来汇总


### 总结 
- ThreadLocal并不解决线程间共享数据的问题
- ThreadLocal适用于变量在线程间隔离且在方法间共享的场景
- ThreadLocal通过隐式的在不同线程内创建独立实例副本避免了实例线程安全的问题每个线程持有一个只属于自己的专属Map并维护了ThreadLocal对象与具体实例的映射，该Map由于只被持有它的线程访问，故不存在线程安全以及锁的问题
- ThreadLocalMap的Entry对ThreadLocal的引用为弱引用，避免了ThreadLocal对象无法被回收的问题都会通过expungeStaleEntry，cleanSomeSlots,replaceStaleEntry这三个方法回收键为null 的Entry对象的值（即为具体实例）以及Entry对象本身从而防止内存泄漏，属于安全加固的方法


## 同步工具  

### CountDownLatch

CountDownLatch 是JDK并发包中提供的一个同步工具类,基于AQS实现

**示例：**5个人到齐后开饭  

```java
public class CountDownLatchDemo {

    private static final int PERSON_COUNT = 5;

    private static final CountDownLatch c = new CountDownLatch(PERSON_COUNT);

    public static void main(String[] args) throws InterruptedException {
        System.out.println("l am master, waiting guests...");
        for (int i = 0; i < PERSON_COUNT; i++) {
            int finalI = i;
            new Thread(new Runnable() {
                @SneakyThrows
                @Override
                public void run() {
                    System.out.println(Thread.currentThread().getName()+" l am person["+ finalI +"]");
                    TimeUnit.MILLISECONDS.sleep(500);
                    //System.out.println(Thread.currentThread().getName()+" count:"+c.getCount());
                    c.countDown();
                }
            }).start();
        }
        c.await();
        System.out.println("all guests get, begin dinner...");
    }

}

```

多个线程并发调用`countDown()`方法`await()`方法等待至CountDownLatch为0时结束等待恢复运行

#### 方法
- `getCount()`：获取当前count的值。
- `wait()`：让当前线程在此CountDownLatch对象上等待，可以中断。与notify()、notifyAll()方法对应。
- `await()`：让当前线程等待此CountDownLatch对象的count变为0，可以中断。
- `await(timeout,TimeUnit)`：让当前线程等待此CountDownLatch对象的count变为0，可以超时、可以中断。
- `countDown()`：使此CountDownLatch对象的count值减1(无论执行多少次，count最小值为0)。

#### 使用场景
- 场景一：将任务分割成多个子任务，每个子任务由单个线程去完成，等所有线程完成后再将结果汇总。（MapReduce）这种场景下，CountDoenLatch作为一个完成信号来使用。
- 场景二：多个线程等待，一直等到某个条件发生。比如多个赛跑运动员都做好了准备，就等待裁判手中的发令枪响。这种场景下，就可以将CountdownLatch的初始值设置成1。


#### 简单总结
`CountDownLatch`的初始值不能重置，只能减少不能增加，最多减少到0；    
当`CountDownLatch`计数值没减少到0之前，调用await方法可能会让调用线程进入一个阻塞队列，直到计数值减小到0；
调用`countDown`方法会让计数值每次都减小1，但是最多减少到0。当`CountDownLatch`的计数值减少到0的时候，会唤醒所有在阻塞队列中的线程

### CyclicBarrier
`CyclicBarrier`也是JDK并发包中提供的一个辅助并发工具类。`CyclicBarrier`的作用是让一组线程互相等待，直到这组线程中所有的线程都到达同步点（完成某个动作，体现到API上就是调用`CyclicBarrier`的`await`方法），这些线程才会继续往下工作。

在相互等待的线程被释放后，`CyclicBarrier`可以被循环使用。这个从这个类的名字中的Cyclic就可以看出

- `await()`等待所有 parties在此障碍上调用 await 。
- `await​(long timeout, TimeUnit unit)`等待所有 parties在此屏障上调用 await ，或者指定的等待时间过去。
- `isBroken()`查询此屏障是否处于损坏状态。
- `reset()`将屏障重置为其初始状态。

```java

//集齐7颗龙珠就可以召唤神龙
public class CyclicBarrierDemo {

    //创建固定值
    private static final int NUMBER = 7;

    public static void main(String[] args) {
        //创建CyclicBarrier
        CyclicBarrier cyclicBarrier =
                new CyclicBarrier(NUMBER,()->{
                    System.out.println("*****集齐7颗龙珠就可以召唤神龙");
                });

        //集齐七颗龙珠过程
        for (int i = 1; i <=7; i++) {
            new Thread(()->{
                try {
                    System.out.println(Thread.currentThread().getName()+" 星龙被收集到了");
                    //等待 没到7 就等待
                    cyclicBarrier.await();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            },String.valueOf(i)).start();
        }
    }
}

```


### Semaphore
Semaphore的主要作用是控制线程并发的数量。我们可以将Semaphore想象成景区的一个门卫，这个门卫负责发放景区入园的许可证    

Semaphore是一个计数信号量，从概念上将，信号量维护了一个许可集，如有必要，在许可可用前会阻塞每一个`acquire()`，然后在获取该许可。每个`release()`添加一个许可，从而可能释放一个正在阻塞的获取者。但是，不使用实际的许可对象，Semaphore只对可用许可的号码进行计数，并采取相应的行动

构造方法: `Semaphore(int permits)` 创建具有给定的**许可数**和非公平的公平设置的Semapore

常用方法:
- `acquire()`从此信号量获取一个许可，在提供一个许可前一直将线程阻塞，否则线程被中断
- `release()`释放一个许可，将其返回给信号量  

> 参考案例:三个车位,六辆车来抢  

```java
public class SemaphoreDemo {

    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(3);

        for (int i = 0; i < 6; i++) {
            new Thread(()->{
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName()+"抢到了");

                    TimeUnit.SECONDS.sleep(new Random().nextInt(5));
                    System.out.println(Thread.currentThread().getName()+"挺五秒后离开");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    semaphore.release();
                }


            }, String.valueOf(i)).start();
        }


    }
}
```

### Exchanger简介
Exchanger——交换器，是JDK1.5时引入的一个同步器，从字面上就可以看出，这个类的主要作用是交换数据。    
Exchanger有点类似于CyclicBarrier，我们知道CyclicBarrier是一个栅栏，到达栅栏的线程需要等待其它一定数量的线程到达后，才能通过栅栏。可以看作是一个双向栅栏   

Thread1线程到达栅栏后，会首先观察有没其它线程已经到达栅栏，如果没有就会等待，如果已经有其它线程（Thread2）已经到达了，就会以成对的方式交换各自携带的信息，因此Exchanger非常适合用于两个线程之间的数据交换





# 面试题
1.Synchronized 用过吗，其原理是什么?  
2.你刚才提到获取对象的锁，这个“锁”到底是什么?如何确定对象的领?  
3.什么是可重入性，为什么说Synchronized是可重入锁?    
4.JVM对Java的原生锁做了哪些优化?   
5.为什么说Synchronized是非公平锁?  
6.什么是锁消除和锁粗化?   
7.为什么说Synchronized是—个悲观领?乐观锁的实现原理又是什么?什么是CAS？  
8.乐观锁—定就是好的吗?   
9、synchronized实现原理，monitor对象什么时候生成的?知道monitor的monitorenter和monitorexit这两个是怎么保证同步的吗，或者说，这两个操作计算机底层是如何执行的     
10.刚刚你提到了synchronized的优化过程，详细说一下吧。偏向锁和轻量级锁有什么区别?   
  
二、可重入锁ReentrantLock及其他显式锁相关问题    
1.跟Synchronized相比，可重入锁ReentrantLock 其续现原理有什么不同?  
2那么请谈谈AQS框架是怎么回事儿?  
3.请尽可能详尽地对比下Synchronized和 ReentrantLock的异同。   
4.ReentrantLock是如何实现可重入性的?   

三、其他
1.你怎么理解iava多线程的?怎么处理并发?线程池有那几个核心参数?你们项目中如何根据实际场景设置参数的?  
2.Java加锁有哪几种锁？   
3.简单说说lock ?   
4.hashmap的实现原理? hash冲突怎么解决?为什么使用红黑树?  
5.spring里面都使用了那些设计模式?循环依赖怎么解决?    
6.项目中那个地方用了countdownlanch，怎么使用的?  
7、从集合开始吧，介绍一下常用的集合类，哪些是有序的，哪些是无序的   
8、hashmap是如何寻址的，哈希碰撞后是如何存储数据的，1.8后什么时候变成红黑树,红黑树有什么好处   
9、concurrrenthashmap怎么实现线程安全，一个里面会有几个段 segment，jdk1.8后有优化concurrenthashmap吗?分段锁有什么坏处   


如何中断线程    
- 通过循环读取volatile 变量
- 通过循环读取原子类 atomicBoolean 
- 通过终端方法  
当前线程中断标识为true 是不是线程就立刻停止      
静态方法`Thread.interrupted()` 谈谈理解    


ThreadLocal中ThreadLocalMap的数据结构和关系?  
ThreadLocal的key是弱引用，这是为什么?   
ThreadLocal内存泄露问题你知道吗?    
ThreadLocal中最后为什么要加remove方法?   
ThreadLocal用在什么地方？
ThreadLocal一些细节！
ThreadLocal的最佳实践！



用过Java那些锁
读写锁是什么  锁饥饿是什么
有没有比读写锁更快的锁
StampedLock是什么
ReentrantReadWriteLock 锁降级机制是什么  


# 参考资料
> - [尚硅谷JUC](https://www.bilibili.com/video/BV1ar4y1x727)
> - [深入浅出java多线程](https://redspider.gitbook.io/concurrent/di-yi-pian-ji-chu-pian/1)   
> - [深入浅出java多线程](http://concurrent.redspider.group/article/02/6.html)
> - [CompletionStage](https://blog.csdn.net/listeningsea/article/details/123156227l)
> - [原子类型累加器(这个系列值得全看一遍)](https://www.cnblogs.com/54chensongxia/p/12191042.html)