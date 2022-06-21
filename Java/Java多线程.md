对《深入浅出java多线程》 的学习总结  
https://redspider.gitbook.io/concurrent/di-yi-pian-ji-chu-pian/1   

http://concurrent.redspider.group/article/02/6.html

# 基础篇
## 进程与线程的基本概念
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/基础1.png) 

--- 
> 上下文切换（有时也称做进程切换或任务切换）是指 CPU 从一个进程（或线程）切换到另一个进程（或线程）。上下文是指某一时间点 CPU 寄存器和程序计数器的内容。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/基础2.png)

> 进程和线程的区别与联系

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/基础3.png)

## 多线程入门类和接口

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

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/基础4.png)
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
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/基础5.png)

### Thread类的常用方法
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/基础6.png)

 currentThread() 当前线程对象引用,start(),yield()线程让步,sleep()睡眠,join()等待

 ### Thread和Runnable 接口比较
 ![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/基础7.png)

 ### Callable\ Future\ FutureTask
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/基础8.png)  

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
```
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/基础9.png)


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/基础10.png)

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
        // 使用
        ExecutorService executor = Executors.newCachedThreadPool();
        FutureTask<Integer> futureTask = new FutureTask<>(new Task());
        executor.submit(futureTask);
        System.out.println(futureTask.get());
    }
}
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/基础11.png)  

## 线程组和线程优先级  

### 线程组ThreadGroup  
- 每一个Thread必然存在于一个ThreadGroup中,Thread 不能独立于ThreadGroup存在
- 如果在new Thread时没有显式指定，那么默认将父线程（当前执行new Thread的线程）线程组设置为自己的线程组 
- ThreadGroup管理下属Thread，TreadGroup是一个标准的向下引用树状结构，防止**上级线程被下级线程引用而无法有效的被GC回收** 

### 线程优先级
- 优先级范围1-10
- 默认优先级为5(NORM_PRIORITY),最小为1(MIN_PRIORITY), 最大值为10(MAX_PRIORITY)  
- 更高优先级比低优先级有**更高几率**得到执行,优先级高度依赖于系统  
- Java优先级不可靠,真正的调用顺序由**操作系统的线程调度算法决定**  
- 如果某个线程**优先级大于线程所在线程组的最大优先级**，那么该线程的优先级将会失效，取而代之的是线程组的最大优先级  

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
> 线程可以看作是轻量级进程

操作系统线程主要有以下三个状态
- 就绪状态 ready 
- 执行状态 running
- 等待状态 waiting  

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
> NEW状态的线程此时尚未启动,并未调用线程的start方法

1. 重复调用start方法会报错 IllegalThreadStateException 
2. 一个线程执行完毕(此时处于TERMINATED状态),再次调用会抛出同样的错误 
3. 在调用一次start()之后，threadStatus的值会改变（threadStatus !=0）

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
阻塞状态。处于BLOCKED状态的线程正等待锁的释放以进入同步区。  


#### WAITING 
等待状态, 处于等待状态的线程变成RUNNABLE状态需要其他线程唤醒  
调用下面3个方法会使线程进入等待状态
1. `Object.wait()` 
2. `Thread.join()` 底层为wait方法
3. `LocakSupport.part()`  除非获得调用许可,否则禁用当前线程进行线程调度  

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

#### WAITING状态与RUNNABLE状态的转换  
**Object.wait()**
> 调用wait方法前线程必须持有对象锁
> 调用wait方法会释放当前锁,直到有其他线程调用notify()或者notifyAll() 方法唤醒等待锁的线程
> notify 和notifyAll 方法不一定会唤醒之前调用wait方法的线程, notifyAll 唤醒所有等待线程,也不一定会马上执行 之前的线程,具体调用顺序依赖系统调度

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
#### TIMED_WAITING与RUNNABLE状态转换
- TIMED_WAITING与WAITING状态类似，只是TIMED_WAITING状态等待的时间是指定的。
1. `Thread.sleep(long)`
2. `Object.wait(long)`
> wait(long)方法使线程进入TIMED_WAITING状态。这里的wait(long)方法与无参方法wait()相同的地方是，都可以通过其他线程调用notify()或notifyAll()方法来唤醒。
> 不同的地方是，有参方法wait(long)就算其他线程不来唤醒它，经过指定时间long之后它会自动唤醒，拥有去争夺锁的资格。
3. `Thread.join(long)`

#### 线程中断
- `Thread.interrupt()` 中断线程,不会立刻终止线程,而是将线程的中断状态设置为true(默认为false),如果该线程被sleep调用阻塞,会抛出InterruptedException
- `Thread.interrupted()` 静态方法,测试当前线程是否被中断,会改线程中断状态,调用一次使线程中断状态设置为**false**
- `Thread.isInterrupted()` 测试当前线程是否被中断。与上面方法不同的是调用这个方法并不会影响线程的中断状态。
- `Thread.currentThread().isInterrupted()`：测试当前线程是否被中断。线程的中断状态受这个方法的影响，意思是调用一次使线程中断状态设置为**true**，连续调用两次会使得这个线程的中断状态重新转为false；

```java
//注意上面两个区别
//第2个
public static boolean interrupted() {
        return currentThread().isInterrupted(true);
}

//第4个
public boolean isInterrupted() {
        return isInterrupted(false);
}

//上面两个方法最终调用的同一个本地方法,但是传入的参数不同 
//ClearInterrupted 的值决定状态是否重置  不是设置状态值 
private native boolean isInterrupted(boolean ClearInterrupted);
```

## 线程间通信

### 锁与同步 
```java
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
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/基础13.png)  

### 等待/通知机制

> 通过Object类的wait() notify() notifyAll() 方法来实现  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/基础14.png)  

```java
//将上述方法修改为如下即可 
//完成打印后 notify 唤醒再将当前线程进入等待 释放对象锁
            synchronized (lock){
                for (int i = 0; i < 100; i++) {
                    System.out.println("ThreadAAAAAA"+i);
                    lock.notify();
                    try {
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
```

### 信号量
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/基础15.png)

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

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/基础16.png)  

### 管道 
> JDK提供了 `PipedWriter`、 `PipedReader`、 `PipedOutputStream`、 `PipedInputStream`   
> 前面两个是基于字符的，后面两个是基于字节流的  


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/基础17.png)  

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
### sleep方法
sleep 方法是Thread 类的**静态**方法 :
- `Thread.sleep(long)`
- `Thread.sleep(long,int)` 后面的int是nano 毫秒数,但是并不精确到毫秒,第二个参数nanos大于0.5毫秒时或者小于1毫秒，进一位。

> sleep方法不会释放当前锁,只是释放cpu资源,而wait 方法会释放锁  

- sleep 和 wait 的区别   
  - wait可以指定时间，也可以不指定；而sleep**必须指定时间**。
  - wait**释放cpu资源，同时释放锁**；sleep**释放cpu资源，但是不释放锁**，所以易死锁。
  - wait必须放在同步块或同步方法中，而sleep可以在任意位置。


### ThreadLocal

> ThreadLocal是一个本地线程副本变量工具类。内部是一个弱引用的Map来维护  
> 使得 每一个线程都有自己的专属本地变量,线程之间互不影响。它为每个线程都创建一个副本，每个线程可以访问自己内部的副本变量  

> 最常见的ThreadLocal使用场景为用来解决数据库连接、Session管理等。数据库连接和Session管理涉及多个复杂对象的初始化和关闭。
> 如果在每个线程中声明一些私有变量来进行操作，那这个线程就变得不那么“轻量”了，需要频繁的创建和关闭连接  


# 原理篇

## Java 内存模型 JMM

两种并发模型  
- 消息传递并发模型
- 共享内存并发模型 


> Java中，使用的是共享内存并发模型。

### 运行时内存的划分 
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/Java运行时数据区.png)

对于每一个线程来说，**栈都是私有的，而堆是共有的**。  

也就是说在栈中的变量（局部变量、方法定义参数、异常处理器参数）不会在线程之间共享，也就不会有内存可见性（下文会说到）的问题，也不受内存模型的影响。  
而在堆中的变量是共享的，本文称为共享变量。  

所以，**内存可见性是针对的共享变量**


> 线程之间的共享变量存在主内存中，每个线程都有一个私有的本地内存，存储了该线程已读、写共享变量的副本。
> 本地内存是Java内存模型的一个抽象概念，**并不真实存在**。它涵盖了缓存、写缓冲区、寄存器等。

> Java线程之间的通信由Java内存模型（简称JMM）控制，从抽象的角度来说，**JMM定义了线程和主内存之间的抽象关系**  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/JMM抽象示意图.jpg)

上图总结: 
- 所有的共享变量都保存在主内存中
- 每个线程都保存了一份该线程使用到的共享变量的副本
- 如果线程A和线程B之间进行通信的话,必须经过两个步骤
  - 线程A 将本地内存的共享变量更新到主内存中
  - 线程B 读取主内存中的共享变量,保存到本地内存

> 线程A无法直接访问线程B的工作内存,线程间通信必须经过主内存  
> JMM规定,线程对共享变量的所有操作必须在自己的本地内存中进行,不能直接冲主内存中读取  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/多线程JMM.png)


## 重排序

> 计算机在执行程序时，为了提高性能，编译器和处理器常常会对指令做重排

```java
a = b + c;
d = e - f ;
```
> 先加载b、c（注意，即有可能先加载b，也有可能先加载c），但是在执行add(b,c)的时候，**需要等待b、c装载结束才能继续执行**，
> 也就是增加了停顿，那么后面的指令也会依次有停顿,这降低了计算机的执行效率。

> 为了减少这个停顿，我们可以先加载e和f,然后再去加载add(b,c),这样做对程序（串行）是没有影响的,但却减少了停顿。既然add(b,c)需要停顿，那还不如去做一些有意义的事情。

指令重排一般分为一下三种:  
- 编译器优化重排
- 指令并行重排
- 内存系统重排

> 指令重排可以保证串行语义一致，但是没有义务保证多线程间的语义也一致。所以在多线程下，指令重排序可能会导致一些问题。


### 顺序一致性模型和JMM的保证

#### 数据竞争
当程序未正确同步的时候，就可能存在数据竞争。  
如果程序中包含了数据竞争，那么运行的结果往往充满了不确定性，比如读发生在了写之前，可能就会读到错误的值；如果一个线程程序能够正确同步，那么就不存在数据竞争。

Java内存模型（JMM）对于正确同步多线程程序的内存一致性做了以下保证：  
> 如果程序是正确同步的，程序的执行将具有顺序一致性。 即程序的执行结果和该程序在顺序一致性模型中执行的结果相同。

这里的同步包括了使用`volatile、final、synchronized`等关键字来实现多线程下的同步。


#### 顺序一致性模型

顺序一致性内存模型是一个理想化的**理论参考模型**  

顺序一致性模型的两大特性  
- 一个线程的所有操作必须按照程序的顺序(java 代码的顺序)来执行
- 不管程序是否同步,所有线程都只能看到一个单一的操作执行顺序,在顺序一致性模型中,每个操作必须**是原子性的,且立刻对所有线程可见**  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/顺序一致性模型.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/JMM顺序一致性.png)


## happens-before

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/happensbefore.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/happensbefore示例.png)


## volatile 

- 保证变量的内存可见性
- 禁止volatile变量与普通变量重排序


从volatile的内存语义上来看，volatile可以保证内存可见性且禁止重排序。  

> 在保证内存可见性这一点上，volatile有着与锁相同的内存语义，所以可以作为一个“轻量级”的锁来使用。  
> 但由于volatile仅仅保证对**单个volatile变量的读/写具有原子性**，而锁可以保证整个临界区代码的执行具有原子性。所以在功能上，锁比volatile更强大；在性能上，volatile更有优势。


## synchronized 与锁

> Java多线程的锁都是基于对象的，Java中的每一个对象都可以作为一个锁

synchronized 三种形式

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
> 某一块代码区域，它同一时刻只能由一个线程执行，如果synchronized关键字在方法上，那临界区就是整个方法内部。而如果是使用synchronized代码块，那临界区就指的是代码块内部的区域

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

### 几种锁

> Java 6 为了减少获得锁和释放锁带来的性能消耗，引入了“偏向锁”和“轻量级锁“。在Java 6 以前，所有的锁都是”重量级“锁。
> 所以在Java 6 及其以后，一个对象其实有四种锁状态，它们级别由低到高依次是: 

- 无锁状态
- 偏向锁
- 轻量级锁
- 重量级锁

#### 偏向锁

> 大多数情况下,锁不仅不存在多线程竞争，而且总是由同一线程多次获得，于是引入了偏向锁。 

偏向锁会**偏向于第一个访问锁的线程**，如果在接下来的运行过程中，该锁没有被其他的线程访问，则持有偏向锁的线程将**永远不需要触发同步**。
也就是说，**偏向锁在资源无竞争情况下消除了同步语句，连CAS操作都不做了，提高了程序的运行性能**。

撤销(关闭)偏向锁:  
如果应用程序里所有的锁通常处于竞争状态，那么偏向锁就会是一种累赘，对于这种情况，我们可以一开始就把偏向锁这个默认功能给关闭

```
-XX:UseBiasedLocking=false
```

#### 轻量级锁
多个线程在**不同时段**获取同一把锁，即不存在锁竞争的情况，也就没有线程阻塞。针对这种情况，JVM采用轻量级锁来避免线程的阻塞与唤醒。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/自旋.png)

#### 重量级锁  


#### 锁对比
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/锁对比.png)




## CAS与原子操作

### 乐观锁与悲观锁

> 悲观锁就是我们常说的锁。对于悲观锁来说，它总是认为每次访问共享资源时会发生冲突，所以必须对每次数据操作加上锁，以保证临界区的程序同一时间只能有一个线程在执行

> 乐观锁又称为“无锁”，顾名思义，它是乐观派。乐观锁总是假设对共享资源的访问没有冲突，线程可以不停地执行，无需加锁也无需等待。  
> 而一旦多个线程发生冲突，乐观锁通常是使用一种称为CAS的技术来保证线程执行的安全性  
> 乐观锁天生免疫死锁。


> 乐观锁多用于“**读多写少**“的环境，避免频繁加锁影响性能；而悲观锁多用于”**写多读少**“的环境，避免频繁失败和重试影响性能。

### CAS的概念
CAS的全称是：比较并交换(Compare And Swap)

> 比较并交换的过程如下：   
> 判断V是否等于E，如果等于，将V的值设置为N；如果不等，说明已经有其它线程更新了V，则当前线程放弃更新，什么都不做。

例  
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/cas案例.png)

> CAS是一种原子操作，它是一种系统原语，是一条CPU的原子指令，从CPU层面保证它的原子性


### Java实现CAS的原理 - Unsafe类
Java中，有一个 `Unsafe` 类，它在`sun.misc`包中。它里面是一些`native`方法，其中就有几个关于CAS的:

```java
boolean compareAndSwapObject(Object o, long offset,Object expected, Object x);
boolean compareAndSwapInt(Object o, long offset,int expected,int x);
boolean compareAndSwapLong(Object o, long offset,long expected,long x);
```

### 原子操作-AtomicInteger
> JDK提供了一些用于原子操作的类，在`java.util.concurrent.atomic`包下面

> `AtomicInteger` 类的 `getAndAdd(int delta)` 方法是调用 `Unsafe` 类的方法来实现的

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/atomicInteger.png)


###  CAS实现原子操作的三大问题

#### ABA问题

> 所谓ABA问题，就是一个值原来是A，变成了B，又变回了A。这个时候使用CAS是检查不出变化的，但实际上却被更新了两次

> ABA问题的解决思路是在变量前面追加上**版本号或者时间戳**。从JDK 1.5开始，JDK的atomic包里提供了一个类`AtomicStampedReference`类来解决ABA问题。

#### 循环时间长开销大
CAS多与自旋结合。如果自旋CAS长时间不成功，会占用大量的CPU资源。 

解决思路是让JVM支持处理器提供的pause指令。  

pause指令能让自旋失败时cpu睡眠一小段时间再继续自旋，从而使得读操作的频率低很多,为解决内存顺序冲突而导致的CPU流水线重排的代价也会小很多。  


#### 只能保证一个共享变量的原子操作

- 使用JDK 1.5开始就提供的`AtomicReference`类保证对象之间的原子性，把多个变量放到一个对象里面进行CAS操作  
- 使用锁。锁内的临界区代码可以保证只有当前线程能操作。

## AQS  
> AQS是 `AbstractQueuedSynchronizer` 的简称，即**抽象队列同步器**





































