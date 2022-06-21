
# JUC概述 
## 什么是JUC
`java.util.concurrent` 包 简称JUC
## 进程和线程的概念 

- 进程与线程
- 线程的状态
- wait 和sleep
- 并发和并行 
- 管程
- 用户线程和守护线程


## Lock和synchronized
- `Lock` 是一个接口,而 `synchronized` 是java中的关键字,`synchronized` 是内置的语言实现
- `synchronized` 在发生异常时,会自动释放锁,不会导致死锁, `Lock` 在发生异常时,如果没有主动通过 unLock() 释放,很可能造成死锁,因此使用 `Lock` 时需要在 finally 块中释放锁
- `Lock` 可能让等待锁的线程响应中断,`synchronized` 等待的线程会一致等待,不能响应中断
- 通过 `Lock` 可以知道有没有成功获取锁, 而 `synchronized` 无法实现
- `Lock` 可以提高多个线程进行读操作的效率,在竞争资源激烈的情况下,`Lock` 的性能要远优于`synchronized`
  



## 多线程编程步骤
- 1. 创建资源类,在资源类创建属性和操作方法
- 2. 在资源类操作方法 
  - 1. 判断
  - 2. 干活
  - 3. 通知
- 3. 创建多个线程,调用资源类操作方法
- 4. 防止虚假唤醒问题

## 线程间通信 

### Synchronizaed 案例
> 操作线程时,等待线程使用 `wait()`
> 通知线程 使用 `notify(),notifyAll()`方法

```java
class Share{
    private int i =0;

    public synchronized void incr() throws InterruptedException {
        while (i!=0){
            this.wait();
        }
        i++;
        System.out.println(Thread.currentThread().getName()+"::"+i);
        //通知
        this.notifyAll();
    }

    public synchronized  void decr() throws InterruptedException {
        while (i!=1){
            this.wait();
        }
        i--;
        System.out.println(Thread.currentThread().getName()+"::"+i);
        this.notifyAll();
    }
}
public class ThreadDemo1 {
    public static void main(String[] args) {
        Share share = new Share();
        new Thread(()->{
            for (int i = 1; i < 10; i++) {
                try {
                    share.incr();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        },"AA").start();

        new Thread(()->{
            for (int i = 1; i < 10; i++) {
                try {
                    share.decr();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        },"BB").start();
        new Thread(()->{
            for (int i = 1; i < 10; i++) {
                try {
                    share.incr();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        },"CC").start();

        new Thread(()->{
            for (int i = 1; i < 10; i++) {
                try {
                    share.decr();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        },"DD").start();
    }
}
```
### 虚假唤醒问题
> 如果一个线程执行完毕后，通知其他线程，该线程又进入等待睡眠，可能会因为某些原因被唤醒后，if结构的语句就不会判断了，一直往下执行，所以需要将if换成while结构，每次都判断。因为wait在哪里睡眠就在哪里被唤醒，结果被某个异常唤醒了后回不去了，if结构不会在判断了，需要更改为while

### Lock案例

```java
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();
```
> 上锁`lock.lock()`
> 解锁 `lock.unlock()`
> 唤醒所有等待线程 `condition.signalAll()`
> 唤醒一个等待线程 `condition.signal()`

```java
class Share{
    private int number =0;

    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();


    public void incr() throws InterruptedException {
        lock.lock();

        try {
            while (number!=0){
                condition.await();
            }
            number++;
            System.out.println(Thread.currentThread().getName()+"::"+number);
            condition.signalAll();

        }finally {
            lock.unlock();
        }
    }

    public void decr() throws InterruptedException {

        lock.lock();
        try {
            while (number!=1){
                condition.await();
            }
            number --;
            System.out.println(Thread.currentThread().getName()+"::"+number);
            condition.signalAll();
        }finally {
            lock.unlock();
        }
    }

}
public class ThreadDemo2 {

    public static void main(String[] args) {

        Share share = new Share();
        new Thread(()->{
            for (int i = 1; i < 10; i++) {
                try {
                    share.incr();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"AA").start();

        new Thread(()->{
            for (int i = 1; i < 10; i++) {
                try {
                    share.decr();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"BB").start();
        new Thread(()->{
            for (int i = 1; i < 10; i++) {
                try {
                    share.incr();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"CC").start();

        new Thread(()->{
            for (int i = 1; i < 10; i++) {
                try {
                    share.decr();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"DD").start();
    }

}
```

## 线程间的定制化通信 
> 上面的 AABBCC 输出顺序不确定,所谓定制化通信，需要让线程进行**一定的顺序**操作
> 具体思路:每个线程添加一个标志位(flag)，是该标志位则执行操作，并且修改为下一个标志位，通知下一个标志位的线程

流程
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/juc1.png)

```java
class ShareRec{
    //标志位
    private int flag = 1; //1 AA 2 BB 3 CC
    private Lock lock = new ReentrantLock();
    private Condition c1 = lock.newCondition();
    private Condition c2 = lock.newCondition();
    private Condition c3 = lock.newCondition();

    //打印5次
    public void print5(int loop) throws InterruptedException {
        lock.lock();
        try {
            while (flag!=1){
                c1.await();
            }
            for (int i = 0; i < 4; i++) {
                System.out.println(Thread.currentThread().getName()+"::"+i+"::"+loop);
            }
            flag = 2;
            c2.signal();//修改标志位,在通知BB线程
        }finally {
            lock.unlock();
        }
    }

    //打印10次
    public void print10(int loop) throws InterruptedException {
        lock.lock();
        try {
            while (flag!=2){
                c2.await();
            }
            for (int i = 0; i < 9; i++) {
                System.out.println(Thread.currentThread().getName()+"::"+i+"::"+loop);
            }
            flag = 3;
            c3.signal();//修改标志位,在通知CC线程
        }finally {
            lock.unlock();
        }
    }
    //打印15次
    public void print15(int loop) throws InterruptedException {
        lock.lock();
        try {
            while (flag!=3){
                c3.await();
            }
            for (int i = 0; i < 14; i++) {
                System.out.println(Thread.currentThread().getName()+"::"+i+"::"+loop);
            }
            flag = 1;
            c1.signal();//修改标志位,在通知AA线程
        }finally {
            lock.unlock();
        }
    }


}
public class ThreadDemo3 {
    public static void main(String[] args) {
        ShareRec shareRec = new ShareRec();
        new Thread(()->{
            for (int i = 0; i < 9; i++) {
                try {
                    shareRec.print5(i);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"AA").start();
        new Thread(()->{
            for (int i = 0; i < 9; i++) {
                try {
                    shareRec.print10(i);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"BB").start();
        new Thread(()->{
            for (int i = 0; i < 9; i++) {
                try {
                    shareRec.print15(i);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"CC").start();

    }
}

```

## 集合的线程安全

### ArrayList 集合线程不安全演示 
```java
/**
 * list集合线程不安全演示
 */
public class ThreadDemo4 {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();

        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                list.add(UUID.randomUUID().toString().substring(0,8));

                System.out.println(list);
            },String.valueOf(i)).start();
        }
    }
}
``` 

`java.util.ConcurrentModificationException` 并发修改异常


#### 解决方案 Vector 
通过list下的实现类 `Vector`
因为在 `Vector` 下的add普遍都是线程安全
` List<String> list = new Vector<>();`

#### 解决方案 Collections.synchronizedList

Collections类中的很多方法都是static静态
其中有一个方法是返回指定列表支持的同步（线程安全的）列表为`synchronizedList(List <T> list)`
`List<String> list = Collections.synchronizedList(new ArrayList<>());`

#### 解决方案 JUC CopyOnWriteArrayList

`List<String> list = new CopyOnWriteArrayList<>();`
写时复制技术  
- 读取时并发读取
- 写入时,先复制一份,在复制的那份上进行操作,写入后,将复制的和原来的进行合并覆盖原来的内容.

```java
//源码参考
    public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            //原数组
            Object[] elements = getArray();
            int len = elements.length;
            //复制数组
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            //操作复制的数组
            newElements[len] = e;
            //将复制的数组合并(覆盖)到原数组
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```
### HashSet,HashMap线程不安全
HashSet 解决方案 `CopyOnWriteArraySet`
HashSet 解决方案 `ConcurrentHashMap`




## 多线程锁 

> `synchronized` 实现同步的基础: java中的每个对象都可以作为锁,具体表现为以下三种形式
- 对于普通同步方法,锁是当前实例对象 this
- 对于静态同步方法,锁是当前类的Class对象
- 对于同步方法块,锁是`synchronized` 括号里的配置对象


### 公平锁和非公平锁
> `FairSync` 公平锁,效率相对低
> `NonfairSync` 非公平锁  **(默认)** ,效率高，但是线程容易饿死(某个线程不会执)
```java
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

### 可重入锁 
> `synchronized` (隐式) 和 `Lock` (显式) 都是可重入锁 
> 可重入锁也叫递归锁
> `synchronized` 不需要手动上锁,解锁,所以是隐式
> `Lock` 需要手动上锁,解锁,所以是显式 


### 死锁 
> 两个或两个以上的进程,因为争夺资源而造成的互相等待的现象 


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/juc2.png)


产生死锁的原因:
- 系统资源不足
- 进程运行推进顺序不合适
- 资源分配不当

验证是否为死锁
- `jps` 
- `jstack` jvm自带堆栈跟踪工具 

> `jsp -l` 然后使用 `jstack pid` 查看

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/juc3.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/juc4.png)



## Callable 接口 
创建线程的四种方式
-  继承Thread类
-  实现Runnable 接口
-  实现Callable接口
-  线程池

`Callable` 接口和 `Runnable` 接口的区别

- Callable中的call()计算结果，如果无法计算结果，会抛出异常
- Runnable中的run()使用实现接口Runnable的对象创建一个线程时，启动该线程将导致在独立执行的线程中调用该对象的run方法
- 总的来说：**run()没有返回值，不会抛出异常。而call()有返回值，会抛出异常**


### FutureTask

> 所谓的FutureTask是在不影响主任务的同时，开启单线程完成某个特别的任务，之后主线程续上单线程的结果即可（该单线程汇总给主线程只需要一次即可）
> 如果之后主线程在开启该单线程，可以直接获得结果，因为之前已经执行过一次了

FutureTask的构造方法有
- `FutureTask(Callable<> callable)` 创建一个FutureTask，一旦运行就执行给定的Callable
- `FutureTask(Runnable runnable,V result)` 创建一个FutureTask，一旦运行就执行给定的Ru你那边了，并安排成功完成时get返回给定的结果  


```java
//比较两个接口
//实现Runnable接口
class MyThread1 implements Runnable {
    @Override
    public void run() {

    }
}

//实现Callable接口
class MyThread2 implements Callable {

    @Override
    public Integer call() throws Exception {
        System.out.println(Thread.currentThread().getName()+" come in callable");
        return 200;
    }
}

public class Demo1 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //Runnable接口创建线程
        new Thread(new MyThread1(),"AA").start();

        //Callable接口,报错
       // new Thread(new MyThread2(),"BB").start();

        //FutureTask
        FutureTask<Integer> futureTask1 = new FutureTask<>(new MyThread2());

        //lam表达式
        FutureTask<Integer> futureTask2 = new FutureTask<>(()->{
            System.out.println(Thread.currentThread().getName()+" come in callable");
            return 1024;
        });

        //创建一个线程
        new Thread(futureTask2,"lucy").start();
        new Thread(futureTask1,"mary").start();

//        while(!futureTask2.isDone()) {
//            System.out.println("wait.....");
//        }
        //调用FutureTask的get方法
        System.out.println(futureTask2.get());

        System.out.println(futureTask1.get());

        System.out.println(Thread.currentThread().getName()+" come over");
       }
}

```


## JUC的辅助类

### 减少计数 CouncDownLatch  
> `CountDownLatch` 类可以设置一个计数器，通过 `countDown` 方法进行减一操作，使用 `await` 方法等待计数器不大于0 ，然后继续执行 `awati` 方法之后的语句   



`CountDownLatch` 主要有两个方法:
- 当一个线程或多个线程调用 `await` 方法时,这些线程会阻塞
- 其他线程调用 `countDown` 方法会将计数器减一(不会阻塞)
- 当计数器值变为0时,因调用 `await` 方法而阻塞的线程会被唤醒,继续执行 


```java
//演示 CountDownLatch
public class CountDownLatchDemo {
    //6个同学陆续离开教室之后，班长锁门
    public static void main(String[] args) throws InterruptedException {

        //创建CountDownLatch对象，设置初始值
        CountDownLatch countDownLatch = new CountDownLatch(6);

        //6个同学陆续离开教室之后
        for (int i = 1; i <=6; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+" 号同学离开了教室");

                //计数  -1
                countDownLatch.countDown();

            },String.valueOf(i)).start();
        }

        //等待
        countDownLatch.await();

        System.out.println(Thread.currentThread().getName()+" 班长锁门走人了");
    }
}

```

### 循环栅栏 CyclicBarrier
> 该类是一个同步辅助类，允许一组线程互相等待，直到到达某个公共屏障点 `common barrier point`，在设计一组固定大小的线程的程序中，这些线程必须互相等待，这个类很有用，因为barrier在释放等待线程后可以重用，所以称为循环 `barrier` 
> `CyclicBarrier` 的构造方法第一个参数是目标障碍数,每执行 `CyclicBarrier` 一次,障碍数就会加一,如果达到了目标障碍数,才会执行 `cyclicBarrier.await()` 之后的语句


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
                    //等待
                    cyclicBarrier.await();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            },String.valueOf(i)).start();
        }
    }
}

```

### 信号灯 Semaphore 

> 一个计数信号量，从概念上将，信号量维护了一个许可集，如有必要，在许可可用前会阻塞每一个`acquire()`，然后在获取该许可。每个`release()`添加一个许可，从而可能释放一个正在阻塞的获取者。但是，不使用实际的许可对象，Semaphore只对可用许可的号码进行计数，并采取相应的行动

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

## 读写锁 

> **悲观锁:** 悲观锁是基于一种悲观的态度类来防止一切数据冲突，它是以一种预防的姿态在修改数据之前把数据锁住，然后再对数据进行读写，在它释放锁之前任何人都不能对其数据进行操作，直到前面一个人把锁释放后下一个人数据加锁才可对数据进行加锁，然后才可以对数据进行操作，一般数据库本身锁的机制都是基于悲观锁的机制实现的;  
> **特点:** 可以完全保证数据的独占性和正确性，因为每次请求都会先对数据进行加锁， 然后进行数据操作，最后再解锁，而加锁释放锁的过程会造成消耗，所以性能不高;

> **乐观锁:** 操作数据时不会对操作的数据进行加锁（这使得多个任务可以并行的对数据进行操作），只有到数据提交的时候才通过一种机制来验证数据是否存在冲突(一般实现方式是通过加版本号然后进行版本号的对比方式实现);  
> **特点:** 乐观锁是一种并发类型的锁，其本身不对数据进行加锁通而是通过业务实现锁的功能，不对数据进行加锁就意味着允许多个请求同时访问数据，同时也省掉了对数据加锁和解锁的过程，这种方式因为节省了悲观锁加锁的操作，所以可以一定程度的的提高操作的性能，不过在并发非常高的情况下，会导致大量的请求冲突，冲突导致大部分操作无功而返而浪费资源，所以在高并发的场景下，乐观锁的性能却反而不如悲观锁。

- 表锁：整个表操作，不会发生死锁
- 行锁：每个表中的单独一行进行加锁，会发生死锁  
- 读锁：共享锁（可以有多个人读），会发生死锁  
- 写锁：独占锁（只能有一个人写），会发生死锁  

读写锁`ReentrantReadWriteLock`  
读锁为`ReentrantReadWriteLock.ReadLock.readLock()`方法  
写锁为`ReentrantReadWriteLock.WriteLock.writeLock()`方法   

创建读写锁对象`private ReadWriteLock rwLock = new ReentrantReadWriteLock();`   
写锁 加锁 `rwLock.writeLock().lock();`，解锁为`rwLock.writeLock().unlock();`   
读锁 加锁`rwLock.readLock().lock();`，解锁为`rwLock.readLock().unlock();` 

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

> 读写锁: 一个资源可以被多个读线程访问,或者一个写线程访问,但是不能同时存在读写线程,读写互斥,读读共享

读写锁的演变过程: 
- 无锁情况:多线程抢占资源
- 添加锁 `synchronized` `reentrantLock` 这种锁都是独占锁,无论读写都是不能共享,每次只能有一个读或者一个写操作
- 读写锁 `ReentrantReadWriteLock` 读取可以共享(写还是不行),缺点:容易一直读,不写;读时不能写,写时可以读

锁降级: 将**写入锁**降级为**读锁** ,但是读锁不能升级为写锁 
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/juc5.png)

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

```

## 阻塞队列 BlockingQueue
> 队列是空的, 从队列中获取元素的操作将被阻塞
> 队列是满的,从队列中添加元素的操作将被阻塞 


> 试图从空的队列中获取元素的线程将会被阻塞,直到其他线程往空队列插入新的元素
> 试图向已满的队列中添加新的元素的线程将会被阻塞,指导其他线程从队列中移除一个或多个元素或者完全清空

> 阻塞: 某些情况下线程会被挂起,一旦满足条件线程就会被自动唤醒 


### ArrayBlcokingQueue
由数组结构组成的有界的阻塞队列


### LinkedBlcokingQueue
由链表组成的有界阻塞队列

### 核心方法 
方法类型 | 抛出异常 | 特殊值| 阻塞| 超时
---------|----------|---------|---------|---------
 插入 | `add(e)` | `offer(e)`| `put(e)`| `offer(e,time,unit)`
 移除 | `remove()` | `poll()`| `take`| `poll(time,unit)`
 检查 | `element()` | `peek()`| 不可用| 不可用

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/juc6.png)


## 线程池
> ThreadPool 线程池维护着多个线程，一种线程使用模式

特点：
- 降低资源消耗，线程复用
- 控制并发数量
- 可以对线程做统一管理

### 线程池使用方式
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

### ThreadPoolExecutor七个参数解读
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


#### 自定义线程池
> 实际应用中,不建议使用 `Executors` 创建线程池,而是通过 `ThreadPoolExecutor` 方式 ,可能会造成OOM

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

## Fork/Join
> `Fork/Join`框架是一个实现了`ExecutorService` 接口的多线程处理器，它专为那些可以通过递归分解成更细小的任务而设计，最大化的利用多核处理器来提高应用程序的性能。
> 与其他 `ExecutorService` 相关的实现相同的是，`Fork/Join`框架会将任务分配给线程池中的线程。而与之不同的是，`Fork/Join` 框架在执行任务时使用了工作窃取算法。

fork在英文里有分叉的意思，join在英文里连接、结合的意思。顾名思义，fork就是要使一个大任务分解成若干个小任务，而join就是最后将各个小任务的结果结合起来得到大任务的结果。

- `ForkJoinTask` 我们要使用 Fork/Join 框架，首先需要创建一个 ForkJoin 任务。该类提供了在任务中执行 fork 和 join 的机制。通常情况下我们不需要直接集成 `ForkJoinTask` 类，只需要继承它的子类，Fork/Join 框架提供了两个子类:
  - RecursiveAction：用于没有返回结果的任务
  - RecursiveTask:用于有返回结果的任务
- `ForkJoinTask` 需要通过 `ForkJoinPool` 来执行
- `RecursiveTask` 继承后可以实现递归调用的任务

```java
class MyTask extends RecursiveTask<Integer> {
    //拆分差值不能超过10
    private static final Integer val = 10;

    private int begin;
    private int end;
    private int result;

    public MyTask(int begin,int end) {
        this.begin = begin;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        //判断相加的两个数是否差超过10
        if((end-begin)<=val){
            for (int i = begin; i <=end ; i++) {
                result +=i;
            }

        }else{
            //进一步拆分
            int middle = (begin+end)/2;
            MyTask myTask1 = new MyTask(begin, middle);
            MyTask myTask2 = new MyTask(middle+1, end);
            //调用方法拆分
            myTask1.fork();
            myTask2.fork();

            result = myTask1.join()+myTask2.join();
        }


        return result;
    }
}

public class ForkJoinDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        MyTask myTask = new MyTask(0,100);
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Integer> submit = forkJoinPool.submit(myTask);
        Integer integer = submit.get();
        System.out.println(integer);
        forkJoinPool.shutdown();
    }

}

```

## CompletableFuture 异步回调 
> `CompletableFuture` 是java8 中新引入的类
> `CompletableFuture` 在 Java 里面被用于异步编程，异步通常意味着非阻塞，可以使得我们的任务单独运行在与主线程分离的其他线程中，并且通过回调可以在主线程中得到异步任务的执行状态，是否完成，和是否异常等信息
> `CompletableFuture` 实现了 `Future`, `CompletionStage` 接口，实现了 Future接口就可以兼容现在有线程池框架，而 `CompletionStage` 接口才是异步编程的接口抽象，里面定义多种异步方法，通过这两者集合，从而打造出了强大的 `CompletableFuture` 类

```java
public class CompletableDemo {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //异步调用没有返回值的方法
        CompletableFuture<Void> future = CompletableFuture.runAsync(()->{
            System.out.println(Thread.currentThread().getName()+"11");
        });
        future.get();

        //异步调用有返回值
        CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(()->{
            System.out.println(Thread.currentThread().getName());
            return 1220;
        });
        future1.whenComplete((a,b)->{
            System.out.println(a);//返回值
            System.out.println(b);//异常
        });
        future1.get();
    }
}
```



































## 参考资料
> - [尚硅谷视频](https://www.bilibili.com/video/BV1Kw411Z7dF)
> - [csdn博客](https://blog.csdn.net/weixin_47872288/article/details/119453092)
