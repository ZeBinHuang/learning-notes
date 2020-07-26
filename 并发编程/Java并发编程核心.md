# Java并发编程核心

## 线程基础

### 第01讲：为何说只有一种实现线程的方法？

线程实现方式

**实现Runnable接口**

```java
public class RunnableThread implements Runnable{
    @Override
    public void run() {
        System.out.println("用实现Runnable接口实现线程");
    }
}
```

第1种方式首先创建一个类实现Runnable接口，然后重写Runnable接口的run方法，实例化创建的这个类，之后把这个实例传到Thread类中就可以实现多线程



**继承Thread类**

```java
public class ExtendsThread extends Thread{
    public void run(){
        System.out.println("用Thread类实现多线程");
    }
}
```

第2种方式继承Thread类，重写Thread类的run()方法



**线程池创建线程**

```java
static class DefaultThreadFactory implements ThreadFactory {
    private static final AtomicInteger poolNumber = new AtomicInteger(1);
    private final ThreadGroup group;
    private final AtomicInteger threadNumber = new AtomicInteger(1);
    private final String namePrefix;

    DefaultThreadFactory() {
        SecurityManager s = System.getSecurityManager();
        group = (s != null) ? s.getThreadGroup() :
        Thread.currentThread().getThreadGroup();
        namePrefix = "pool-" +
            poolNumber.getAndIncrement() +
            "-thread-";
    }

    public Thread newThread(Runnable r) {
        Thread t = new Thread(group, r,
                              namePrefix + threadNumber.getAndIncrement(),
                              0);
        if (t.isDaemon())
            t.setDaemon(false);
        if (t.getPriority() != Thread.NORM_PRIORITY)
            t.setPriority(Thread.NORM_PRIORITY);
        return t;
    }
}
```

线程池本质上是通过线程工厂创建线程的，默认采用DefaultThreadFactory，本质上还是通过new Thread()实现



**有返回值的Callable创建线程**

```java
public class CallableTask implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        return new Random().nextInt();
    }
}
```

无论是Callable还是FutureTask，它们首先和Runnable一样，都是一个任务，是需要被执行的，而不是说他们本身就是线程。



**实现线程只有一种方式**

```java
@Override
public void run() {
    if (target != null) {
        target.run();
    }
}
```

首先，启动线程需要调用start()方法，而start()方法最终还会调用run()方法

上面的代码是Thread类中的run()方法，可以看到，如果target不等于null，就执行target.run()，而target实际上就是一个Runnable，即实现线程时传给Thread类的对象

Thread类本身实现了Runnable接口，继承Thread类的方式会把上述run()方法重写，当线程启动后，最终会调用这个被重写的run()方法

**本质上，实现线程只有一种方式，就是构造一个Thread类，而要想实现线程执行的内容，有两种方式，就是通过实现Runnable接口的方式或继承Thread类重写run()方法**



**实现Runnable接口比继承Thread类实现线程要好**

一，从代码的架构考虑，Runnable只有一个run()方法，定义了需要执行的内容，实现了Runnable与Thread类的解耦，Thread类负责线程启动和属性设置等内容

二，在某些情况下可以提高性能，使用继承Thread类方式，每执行一次任务，都需要创建一个独立的线程，执行完任务后线程走到生命周期的尽头被销毁，如果还想执行这个任务，就必须再新建一个继承了Thread类的类。如果run()方法的内容比较简单，我们使用Runnable接口的方式，就可以把任务直接传入线程池，使用一些固定的线程来完成任务，不需要每次创建销毁线程，大大降低性能开销

三，Java语言不支持双继承，继承Thread类会限制代码的扩展性



### 第02讲：如何正确停止线程？为什么volatile标记位的停止方法是错误的？

对于Java而言，最正确的停止线程的方式是使用interrupt，但interrupt仅仅起到通知被停止线程的作用。

Java程序希望程序间能够相互通知、相互协作地管理线程，因为如果不了解对方正在做的事情，贸然强制停止线程就可能会造成一些安全问题，为了避免造成问题就需要给对方一定的的时间来整理收尾工作。



**如何用interrupt停止线程**

一旦调用线程的interrupt()之后，这个线程的中断标记位就会被设置为true，每个线程都有这样的标记位，当线程执行时，应该定期检查这个标记位，如果标记位被设置成true，就说明有程序想终止该线程。

```java
public class StopThread extends Thread{
    public void run(){
        int count = 0;
        while(!currentThread().isInterrupted()&&count<10000){
            System.out.println("count="+count++);
        }
    }

    public static void main(String[] args) throws InterruptedException {
        StopThread thread = new StopThread();
        thread.start();
        Thread.sleep(3);
        thread.interrupt();
    }
}
```



**sleep期间能否感受到中断**

```java
public class StopDuringSleep {
    public static void main(String[] args) throws InterruptedException {
        Runnable runnable = () -> {
            int num = 0;
            try {
                while (!Thread.currentThread().isInterrupted() && num < 1000) {
                    System.out.println("num=" + num++);
                    Thread.sleep(1000);
                }
            }catch (InterruptedException e) {
                e.printStackTrace();//此处打印出异常信息
            }
        };
        Thread thread = new Thread(runnable);
        thread.start();
        Thread.sleep(3);
        thread.interrupt();
    }
}
```

如果sleep、wait等可以让线程进入阻塞的方法使线程休眠了，而处于休眠中的线程被中断，那么线程是可以感受到中断信号的，并且会抛出一个InterruptedException异常，同时清除中断信号，将中断标记位设置成false



**两种最佳处理方式**

**方法签名抛异常，强制try/catch**

调用方有义务处理异常，要不使用try/catch并在catch中正确处理异常，要不将异常声明到方法签名中

层层传递异常的逻辑保障了异常不被遗漏，而对run()方法而言，就可以根据不同的业务逻辑来进行相应的处理



**再次中断**

```java
private void reInterrupt(){
    try{
        Thread.sleep(3);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
        e.printStackTrace();
    }
}
```

线程在休眠期间被中断，会自动清除中断信号。如果这时手动添加中断信号，中断信号依然可以被捕捉到，这样后续执行的方法依然可以检测到这里发生过中断，可以做出相应的处理



**我们需要注意，在实际开发中不能盲目吞掉中断，如果不在方法签名中声明，也不在catch语句块中再次恢复中断，而是在catch中不作处理，我们称这种行为是“屏蔽了中断请求”。如果我们盲目屏蔽了中断请求，会导致中断信号被完全忽略，最终导致线程无法正确停止**



### 第03讲：线程是如何在6种状态之间转换的？

**线程的6种状态**

![](..\笔记图片\线程六种状态.jpg)

**New(新创建)**

线程被创建但尚未启动的状态

**Runnable(可运行)**

对应操作系统线程状态中的两种状态，分别是Running和Ready，也就是说，Java中处于Runnable状态的线程有可能正在执行，也有可能没有正在执行，正在等待被分配CPU资源

**Blocked(阻塞)**

进入synchronized保护的代码时没有抢到monitor锁

**waiting(等待)**

线程进入waiting状态有三种可能性：

没有设置Timeout参数的Object.wait()方法

没有设置Timeout参数的Thread.join()方法

LockSupport.park()方法

**Timed Waiting(计时等待)**

线程进入waiting状态有以下可能性：

设置了Timeout参数的Thread.sleep(long millis)方法

设置Timeout参数的Object.wait(long millis)方法

设置Timeout参数的Thread.join(long millis)方法

设置Timeout参数的LockSupport.parkNanos(long timeout)方法和LockSupport.parkUntil(long deadline)方法

**Terminated(被终止)**

run()方法执行完毕，线程正常退出

出现一个没有捕获的异常，终止了run()方法，最终导致意外终止



### 第04讲：wait/notify/notifyAll方法的使用注意事项

**wait必须在synchronized保护的同步代码中使用**

假设我们可以随意使用wait()方法，在没有受到synchronized保护的代码中使用wait()方法，有可能出现以下场景：

线程在调用wait()方法之前，就被调度器暂停了，此时还没来得及执行wait方法；

在give方法执行了notify()并没有任何效果，因为wait()方法还没来得及执行；

此时，刚才被调度器暂停的线程回来继续执行wait()方法并进入了等待

假设这时没有更多的线程执行notify()方法，那么等待的线程就可能陷入无穷无尽的等待

在以上场景中，“判断-执行”不是一个原子操作，在中间被打断了，是线程不安全的

```java
Queue<String> buffer = new LinkedList<>();

public void give(String data){
    buffer.add(data);
    notify();
}

public String take() throws InterruptedException {
    while(buffer.isEmpty()){
        wait();
    }
    return buffer.remove();
}
```

修改成被synchronized保护的同步代码块的形式

```java
Queue<String> buffer = new LinkedList<>();

public void give(String data){
    synchronized (this){
        buffer.add(data);
        notify();
    }
}

public String take() throws InterruptedException {
    synchronized (this){
        while(buffer.isEmpty()){
            wait();
        }
        return buffer.remove();
    }
}
```

这里还存在一个“虚假唤醒”的问题，使用while循环的结构来保证正确性



**为什么wait/notify/notifyAll被定义在Object类中，而sleep定义在Thread类中**

主要有以下原因：

因为Java中每个对象都有一把称之为monitor监视器的锁，由于每个对象都可以上锁，这就要求在对象头中有一个用来保存锁信息的位置。这个锁是对象级别的，不是线程级别的，wait/notify/notifyAll都是锁级别的操作，他们的锁属于对象，所以把它们定义在Object类中是最合适的，因为Object类是所有对象的父类



### 第05讲：有哪几种实现生产者消费者模式的方法？

**使用BlockingQueue实现生产者消费者模式**

```java
public class ConsumerAndProducerSolution01 {
    public static void main(String[] args) {
        BlockingQueue<Object> queue = new ArrayBlockingQueue<Object>(10);

        Runnable producer = ()->{
          while(true){
              try {
                  queue.put(new Object());
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
          }
        };

        new Thread(producer).start();
        new Thread(producer).start();

        Runnable consumer = ()->{
          while(true){
              try {
                  System.out.println(Thread.currentThread().getName()+":"+queue.take());
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
          }
        };

        new Thread(consumer).start();
        new Thread(consumer).start();
    }
}
```



**使用Condition实现生产者消费者模式**

```java
public class MyBlockingQueueForCondition {
    private Queue queue;
    private int max = 16;
    private ReentrantLock lock = new ReentrantLock();
    private Condition notEmpty = lock.newCondition();
    private Condition notFull = lock.newCondition();
    
    public MyBlockingQueueForCondition(int size){
        this.max = size;
        queue = new LinkedList();
    }
    
    public void put(Object o) throws InterruptedException {
        lock.lock();
        try{
            while(queue.size()==max){
                notFull.await();
            }
            queue.add(o);
            notEmpty.notifyAll();
        }finally {
            lock.unlock();
        }
    }
    
    public void take() throws InterruptedException {
        lock.lock();
        try{
            while(queue.isEmpty()){
                notEmpty.await();
            }
            queue.remove();
            notFull.notifyAll();
        }finally {
            lock.unlock();
        }
    }
}
```

在take()方法中使用while(queue.isEmpty())检查队列状态，而不是if(queue.isEmpty())

这是为了防止虚假唤醒，当存在多个消费者时，用if判断导致线程在队列为空时获取到锁，可能会抛出NoSuchElementException异常



**使用wait/notify实现生产者消费者模式**

```java
public class MyBlockingQueue {
    private int maxSize;
    private LinkedList<Object> storage;

    public MyBlockingQueue(int size){
        this.maxSize = size;
        storage = new LinkedList<>();
    }

    public synchronized void put() throws InterruptedException {
        while(storage.size()==maxSize){
            wait();
        }
        storage.add(new Object());
        notifyAll();
    }

    public synchronized void take() throws InterruptedException {
        while(storage.isEmpty()){
            wait();
        }
        System.out.println(storage.remove());
        notifyAll();
    }
}
```

使用MyBlockingQueue实现

```java
public class WaitStyle {
    public static void main(String[] args) {
        MyBlockingQueue storage = new MyBlockingQueue(10);
        Producer producer = new Producer(storage);
        Consumer consumer = new Consumer(storage);
        new Thread(producer).start();
        new Thread(consumer).start();
    }
}

class Producer implements Runnable{
    private MyBlockingQueue storage;
    public Producer(MyBlockingQueue storage){
        this.storage = storage;
    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            try {
                storage.put();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
class Consumer implements Runnable{
    private MyBlockingQueue storage;
    public Consumer(MyBlockingQueue storage){
        this.storage = storage;
    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            try {
                storage.take();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```



## 线程池

**使用线程池的好处**

第一，线程池可以解决线程生命周期的系统开销问题，同时可以加快响应速度。

第二，线程池可以统筹内存和CPU的使用，避免资源使用不当。

第三，线程池可以统一管理资源。



**线程池的参数**

| 参数名                 | 含义                     |
| ---------------------- | ------------------------ |
| corePoolSize           | 核心线程数               |
| maxPoolSize            | 最大线程数               |
| keepAliveTime+时间单位 | 空闲线程的存活时间       |
| ThreadFactory          | 线程工厂、用来创建新线程 |
| workQueue              | 用于存放任务的队列       |
| Handler                | 处理被拒绝的任务         |



**线程池有哪四种拒绝策略？**

| 拒绝策略            | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| AbortPolicy         | 直接抛出一个类型为RejectedExecutionException的RuntimeException |
| DiscardPolicy       | 直接丢弃新任务                                               |
| DiscardOldestPolicy | 丢弃任务队列的头结点                                         |
| CallerRunsPolicy    | 把任务交给提交任务的线程执行                                 |



**6种常见的线程池**

| 方法                          | 解释                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| FixedThreadPool               | 创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程池达到最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行而异常结束，那么线程池会补充一个新的线程。 |
| CachedThreadPool              | 创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，就会回收部分空闲的线程(60秒没有执行的线程)，当任务数量增加时，以可以智能的添加新的线程来处理任务。这个线程池不会对线程池的大小进行限制，线程池的大小完全依赖操作系统(JVM)能够创建的最大线程大小。有一个用于存储提交任务的队列，但这个队列是SynchronousQueue，队列的容量为0，实际不存储任何任务，只负责对任务进行中转和传递，所以效率比较高 |
| SingleThreadExecutor          | 创建一个单线程的线程池。这个线程池只有一个工作线程，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。些线程池保证所有任务的执行顺序会按照任务的提交顺序执行。 |
| ScheduledThreadPool           | 创建一个大小无限的线程池。线程池运行定时以及周期性执行任务的需求。 |
| SingleThreadScheduledExecutor | 创建一个单线程用于定时以及周期性执行任务的需求。             |
| ForkJoinPool                  | 可以执行产生子任务的任务；每个线程都有自己独立的任务队列；“work-stealing”中任务繁重的线程获取任务的逻辑是后进先出，空闲的线程偷任务的逻辑是先进先出 |



**线程池内部结构**

线程管理器、工作线程、任务队列、任务



**阻塞队列**

| FixedThreadPool               | LinkedBlockingQueue |
| ----------------------------- | ------------------- |
| SingleThreadExecutor          | LinkedBlockingQueue |
| CachedThreadPool              | SynchronousQueue    |
| ScheduledThreadPool           | DelayedWorkQueue    |
| SingleThreadScheduledExecutor | DelayedWorkQueue    |

LinkedBlockingQueue：没有容量限制

SynchronousQueue：容量为0

DelayedWorkQueue：按照延迟的时间长短对任务进行排序



**合适的线程数量是多少？CPU核心数和线程数的关系**

**CPU密集型任务**

最佳的线程数为CPU核心数的1~2倍

**耗时IO型任务**

最大线程数一般会大于CPU核心数很多倍

**线程数=CPU核心数*（1+平均等待时间/平均工作时间）**



**定制自己的线程池**

核心线程数

阻塞队列

ArrayBlockingQueue

线程工厂

可以使用默认的defaultThreadFactory，也可以通过com.google.common.util.concurrent.ThreadFactoryBuilder来实现

```java
ThreadFactoryBuilder builder = new ThreadFactoryBuilder();
//生成的线程的名字是有固定格式的
ThreadFactory rpcFactory = builder.setNameFormat("rpc-pool-%d").build();
```

拒绝策略

可以选择四种拒绝策略之一，也可以通过实现RejectedExecutionHandler接口来实现自己的拒绝策略



**如何正确关闭线程池？**

shutdown()

isShutdown()

isTerminated()

awaitTermination()

shutdownNow()



**线程池实现“线程复用”的原理**

核心原理在于线程池对Thread进行了封装，并不是每次执行任务都会调用Thread.start()来创建新线程，而是让每个线程去执行一个“循环任务”，在这个“循环任务”中，不停的检查是否还有任务等待被执行，如果有则直接去执行这个任务，也就是调用任务的run方法。

查看ThreadPoolExecutor的execute方法源码

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    int c = ctl.get();
    //判断当前线程数是否小于核心线程数
    if (workerCountOf(c) < corePoolSize) {
        //布尔值传入true代表增加线程时判断当前线程是否少于corePoolSize，小于则新增新线程
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    
    //检查线程池状态
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        //线程池被关闭，移除刚刚添加的任务
        if (! isRunning(recheck) && remove(command))
            reject(command);
        //检查当前线程数为0
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    
    //布尔值传入true代表增加线程时判断当前线程是否少于maxPoolSize，小于则新增新线程
    else if (!addWorker(command, false))
        reject(command);
}
```

Worker可以理解为是对Thread的包装

```java
Worker(Runnable firstTask) {
    setState(-1); // inhibit interrupts until runWorker
    this.firstTask = firstTask;
    this.thread = getThreadFactory().newThread(this);
}
```

线程复用的逻辑实现主要在Worker类中的run方法里执行的runWorker方法中

```java
final void runWorker(Worker w) {
    Runnable task = w.firstTask;

    while (task != null || (task = getTask()) != null) {
        try{
            task.run();
        } finally {
            task = null;
        }
    }
}
```





## 各种各样的“锁”

**锁的7大分类**

**偏向锁/轻量级锁/重量级锁**

这三种锁特指synchronized锁的状态

偏向锁：锁不存在竞争

轻量级锁：锁原来是偏向锁的时候，被另一个锁访问，说明存在竞争，那么偏向锁就会升级为轻量级锁，线程会通过自旋的形式去尝试获取锁

重量级锁：是互斥锁，多个线程直接有实际竞争，且锁竞争时间长，拿不到锁的线程会进入阻塞状态

**可重入锁/非可重入锁**

可重入锁：能在不释放这把锁的情况下，再次获取这把锁

**共享锁/独占锁**

共享锁：同一把锁可以被多个线程同时获得

独占锁：只能被一个线程获得

**公平锁/非公平锁**

**悲观锁/乐观锁**

**自旋锁/非自旋锁**

**可中断锁/不可中断锁**

不可中断锁：一旦线程申请了锁，就没有回头路了，只能等到拿到锁以后才能进行其他的逻辑处理



**悲观锁和乐观锁的本质**

悲观锁适合用于并发写入多、临界区代码复杂、竞争激烈等场景，这种场景下悲观锁可以避免大量的无用的反复尝试等消耗

乐观锁适用于大部分是读取，少部分是修改的场景，也适合虽然读写都很多，但是并发并不激烈的场景。



**synchronized背后的“monitor”锁**

同步代码块：反编译后的代码中多了monitorenter和monitorexit指令

同步方法：反编译后的代码中有一个叫作ACC_SYNCHRONIZED的flag修饰符



**Lock的常用方法**

lock()

tryLock()

tryLock(long time,TimeUnit unit)

lockInterruptibly()

unlock()



**读写锁的获取规则**

读读共享，其他都互斥



**讲一讲公平锁和非公平锁**

**为什么要非公平？**

因为唤醒一个等待的线程是需要很大开销的，很有可能在唤醒之前，插队的线程已经拿到了这把锁并且执行完任务释放了这把锁，

**读锁插队策略**

第一种：允许插队

缺陷：如果等待队列中有一个线程等待获取写锁，这时不断有读取线程增加，会导致等待的线程长时间内拿不到写锁，陷入“饥饿”状态

第二种：不允许插队

即便是非公平锁，只要等待队列的头结点是尝试获取写锁的过程，那么读锁依然是不能插队的，目的是避免“饥饿”



**锁的升降级**

读写锁降级























