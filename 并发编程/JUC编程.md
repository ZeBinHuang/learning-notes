### 线程和进程



### 并发和并行



### Lock锁

传统synchronized与Lock接口

两者区别：

1、synchronized是在JVM层面上实现的，不但可以通过一些监控工具监控synchronized的锁定，而且在代码执行时出现异常，JVM会自动释放锁定，但是使用Lock则不行，lock是通过代码实现的，要保证锁定一定会被释放，就必须将unLock()放到finally{}中

2、ReentrantLock 支持中断等待获取锁的线程：当持有锁的线程长期不释放锁时，正在等待的线程可以选择放弃等待，改为处理其他事情，它对处理执行时间非常上的同步块很有帮助。而在等待由synchronized产生的互斥锁时，会一直阻塞，是不能被中断的。

3、synchronized 必须在获取锁的代码块中释放锁，但是ReentrantLock 可以更加灵活的选择何时去释放锁。

4、ReentrantLock 可实现公平锁和不公平锁：多个线程在等待同一个锁时，必须按照申请锁的时间顺序排队等待，而非公平锁则不保证这点，在锁释放时，任何一个等待锁的线程都有机会获得锁。synchronized中的锁为非公平锁，ReentrantLock默认情况下也是非公平锁，但可以通过构造方法ReentrantLock（ture）来要求使用公平锁。  

 5、ReentrantLock 可以绑定多个条件：一个ReentrantLock对象可以同时绑定多个Condition对象（条件变量或条件队列），而在synchronized中，锁对象的wait()和notify()或notifyAll()方法可以实现一个隐含条件，但如果要和多于一个的条件关联的时候，就不得不额外地添加一个锁，而ReentrantLock则无需这么做，只需要多次调用newCondition()（此为Lock接口中规定创建Condition监视器的方法）即可。而且我们还可以通过绑定Condition对象来判断当前线程通知的是哪些线程（即与Condition对象绑定在一起的相关线程）。synchronized使用object类中的监视器方法wait和notify，而且只能创建一个监视器队列，但是Lock接口可以创建多个Condition实例，然后调用await和singal来操作。

乐观锁

悲观锁

### 生产者和消费者问题

线程通信问题

注意：防止出现虚假唤醒，等待应该总是出现在循环中！！！

Condition可实现精准的通知和唤醒线程：

一个ReentrantLock对象可以同时绑定多个Condition对象，只需要多次调用newCondition()（此为Lock接口中规定创建**Condition监视器**的方法）即可，可以通过绑定Condition对象来判断当前线程通知的是哪些线程（即与Condition对象绑定在一起的相关线程），调用await()和singal()方法来操作

### 8锁现象

关于锁的八个问题

如何判断锁的是谁？

1、普通同步方法，synchronized 锁的对象是方法的调用者，谁先拿到锁谁先执行

2、静态同步方法，类一加载就有了！锁的是Class。

​	  同一个类的两个对象的Class类模板只有一个，static，锁的是Class

### 集合类不安全

1、List

```java
//并发下 ArrayList 不安全的吗，Synchronized；        
/** 解决方案；        
* 1、List<String> list = new Vector<>();         
* 2、List<String> list = Collections.synchronizedList(new ArrayList<> ());         
* 3、List<String> list = new CopyOnWriteArrayList<>()；         
*/        
// CopyOnWrite 写入时复制   COW  计算机程序设计领域的一种优化策略；        
// 多个线程调用的时候，list，读取的时候，固定的，写入（覆盖）       
// 在写入的时候避免覆盖，造成数据问题！        
// 读写分离
    
//CopyOnWriteArrayList底层add()方法
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```

2、Set

```java
//1、Set<String> set = Collections.synchronizedSet(new HashSet<>()); 
//2、Set<String> set = new CopyOnWriteArraySet<>(); 
```

3、Map

```java
// Map<String, String> map = new ConcurrentHashMap<>();
//ConcurrentHashMap的原理
```

### Callable

```java
//可以有返回值，可以抛出被检查异常
public class CallableTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        MyThread thread = new MyThread();
        FutureTask<Integer> tack = new FutureTask<>(thread);//适配类
        new Thread(tack,"A").start();
        new Thread(tack,"B").start();//结果会被缓存
        Integer i = (Integer)tack.get();//可能造成阻塞
        System.out.println(i);
    }
}
class MyThread implements Callable<Integer>{
    public Integer call(){
        System.out.println("call...");
        return 1024;
    }
}
```

### 常用的辅助类

1、CountDownLatch

```java
//计数器
public static void main(String[] args) throws InterruptedException {
    CountDownLatch count = new CountDownLatch(5);
    for (int i = 0; i < 5; i++) {
        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+"go go go");
            count.countDown();//数量-1
        },String.valueOf(i)).start();
    }
    count.await();// // 等待计数器归零，然后再向下执行
    System.out.println("finished...");
}
```

2、CyclicBarrier

```java
//加法计数器
public static void main(String[] args) {
     CyclicBarrier cyclic = new CyclicBarrier(4,()->{
         System.out.println("集齐四个");
     });
     for (int i = 0; i < 4; i++) {
         final int temp = i;
         new Thread(()->{
             System.out.println(Thread.currentThread().getName()+" "+temp);
             try {
                 cyclic.await();
             } catch (InterruptedException e) {
                 e.printStackTrace();
             } catch (BrokenBarrierException e) {
                 e.printStackTrace();
             }
         }).start();
     }
 }
```

3、Semaphore：信号量

```java
//作用： 多个共享资源互斥的使用！并发限流，控制大的线程数！
public static void main(String[] args) {
    Semaphore sema = new Semaphore(3);
    for (int i = 0; i < 7; i++) {
        new Thread(()->{
            try {
                sema.acquire();
                System.out.println(Thread.currentThread().getName()+"抢到车位");
                TimeUnit.SECONDS.sleep(2);
                System.out.println(Thread.currentThread().getName()+"离开车位");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }finally {
                sema.release();
            }
        }).start();
    }
}
```

### 读写锁

ReentrantReadWriteLock

- 读写锁：分为读锁和写锁，多个读锁不互斥，读锁与写锁互斥，这是由JVM自己控制的。
- ReentrantReadWriteLock会使用两把锁来解决问题，一个读锁（多个线程可以同时读），一个是写锁（单个线程写）。
- ReadLock可以被多个线程持有并且在作用时排斥任何的WriteLock，而WriteLock则是完全的互斥。这一特性最为重要，因为对于高读取频率而相对较低写入的数据结构，使用此类锁同步机制则可以提高并发量。

```java
public class ReadWriteLockTest {
    public static void main(String[] args) {
        MyCache cache = new MyCache();
        for (int i = 0; i < 3; i++) {
            final int temp = i;
            new Thread(()->{
                cache.get(temp+"");
            }).start();
        }
        for (int i = 0; i < 3; i++) {
            final int temp = i;
            new Thread(()->{
                cache.put(temp+"",temp+"");
            }).start();
        }
    }
}
class MyCache{
    private volatile Map<String,Object> map = new HashMap<>();
    // 读写锁： 更加细粒度的控制
    private ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    public void put(String key,Object value){
        readWriteLock.writeLock().lock();
        try{
            System.out.println(Thread.currentThread().getName()+"写入");
            map.put(key,value);
            System.out.println(Thread.currentThread().getName()+"写入ok");
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            readWriteLock.writeLock().unlock();
        }
    }

    public void get(String key){
        readWriteLock.readLock().lock();
        try{
            System.out.println(Thread.currentThread().getName()+"读取");
            map.get(key);
            System.out.println(Thread.currentThread().getName()+"读取ok");
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            readWriteLock.readLock().unlock();
        }
    }
}
```

### 阻塞队列

**四组API**

| 方式         | 抛出异常 | 有返回值，不抛出异常 | 阻塞 等待 | 超时等待 |
| :----------- | -------- | -------------------- | --------- | -------- |
| 添加         | add      | offer                | put       | offer    |
| 移除         | remove   | poll                 | take      | poll     |
| 检测队首元素 | element  | peek                 |           |          |

SynchronousQueue 同步队列

没有容量，进去一个元素，必须等待取出来之后，才能再往里面放一个元素！

### 线程池

> 三大方法

```java
 public static void main(String[] args) {
      ExecutorService threadPool = Executors.newSingleThreadExecutor();// 单个线 程 
     // ExecutorService threadPool = Executors.newFixedThreadPool(5); // 创建一个固定的线程池的大小 
     // ExecutorService threadPool = Executors.newCachedThreadPool(); // 可伸缩的，遇强则强，遇弱则弱
     try {
         for (int i = 0; i < 3; i++) {
             pool.execute(()->{
                 System.out.println(Thread.currentThread().getName());
             });
         }
     }catch (Exception e){
         e.printStackTrace();
     }finally {
         pool.shutdown();
     }
 }
```

> 七大参数

```java
// 本质ThreadPoolExecutor（）    
public ThreadPoolExecutor(int corePoolSize, // 核心线程池大小                          
                          int maximumPoolSize, // 大核心线程池大小                          
                          long keepAliveTime, // 超时了没有人调用就会释放                          
                          TimeUnit unit, // 超时单位                          
                          BlockingQueue<Runnable> workQueue, // 阻塞队列  
                          ThreadFactory threadFactory, // 线程工厂：创建线程的，一般 不用动                       
                          RejectedExecutionHandler handle // 拒绝策略)    
使用自定义线程池
public static void main(String[] args) {
    ExecutorService pool = new ThreadPoolExecutor(
        2,5,
        3, TimeUnit.SECONDS,new LinkedBlockingDeque<>(3),
        Executors.defaultThreadFactory(),
        new ThreadPoolExecutor.DiscardOldestPolicy()
    );
    try {
        // 最大承载：Deque + max
        // 超过 RejectedExecutionException
        for (int i = 0; i < 9; i++) {
            pool.execute(()->{
                System.out.println(Thread.currentThread().getName());
            });
        }
    }catch (Exception e){
        e.printStackTrace();
    }finally {
        pool.shutdown();
    }
}                          
```

> 四种拒绝策略

```java
/*new ThreadPoolExecutor.AbortPolicy() // 银行满了，还有人进来，不处理这个人的，抛出异 常 
* new ThreadPoolExecutor.CallerRunsPolicy() // 哪来的去哪里！ 
* new ThreadPoolExecutor.DiscardPolicy() //队列满了，丢掉任务，不会抛出异常！ 
* new ThreadPoolExecutor.DiscardOldestPolicy() //队列满了，尝试去和早的竞争，也不会抛出异常！ 
*/
```

池的大的大小如何去设置！
了解：IO密集型，CPU密集型：（调优）

```java
// 大线程到底该如何定义        
// 1、CPU 密集型，几核，就是几，可以保持CPu的效率高！        
// 2、IO  密集型   > 判断你程序中十分耗IO的线程，        
// 程序   15个大型任务  io十分占用资源！
```

### 函数式接口

只有一个方法的接口

> Function函数式接口

```java
public interface Function<T, R> {
    R apply(T t);
}   
```

> Predicate断定型接口

```java
public interface Predicate<T> {
    boolean test(T t);
}    
```

> Consumer消费型接口

```java
public interface Consumer<T> {
    void accept(T t);
}    
```

> Supplier供给型接口

```java
public interface Supplier<T> {
    T get();
}
```

### Stream流式计算

```java
/** * 题目要求：一分钟内完成此题，只能用一行代码实现！ 
* 现在有5个用户！筛选： 
* 1、ID 必须是偶数 
* 2、年龄必须大于23岁 
* 3、用户名转为大写字母
* 4、用户名字母倒着排序
* 5、只输出一个用户！ 
*/ 
public class Test {    
    public static void main(String[] args) {       
    User u1 = new User(1,"a",21);        
    User u2 = new User(2,"b",22);        
    User u3 = new User(3,"c",23);        
    User u4 = new User(4,"d",24);        
    User u5 = new User(6,"e",25);        
    // 集合就是存储        
    List<User> list = Arrays.asList(u1, u2, u3, u4, u5);
    // 计算交给Stream流        
    // lambda表达式、链式编程、函数式接口、Stream流式计算        
    list.stream()                
        .filter(u->{return u.getId()%2==0;})                
        .filter(u->{return u.getAge()>23;})                
        .map(u->{return u.getName().toUpperCase();})               
        .sorted((uu1,uu2)->{return uu2.compareTo(uu1);})            
        .limit(1)                
        .forEach(System.out::println);    
	}
}
```

### ForkJoin

```java
// 如何使用 forkjoin 
// 1、forkjoinPool 通过它来执行 
// 2、计算任务 forkjoinPool.execute(ForkJoinTask task) 
// 3. 计算类要继承 ForkJoinTask
```



### JMM

java内存模型

关于JMM的一些同步的约定： 

1、线程解锁前，必须把共享变量立刻刷回主存。

2、线程加锁前，必须读取主存中的新值到工作内存中！ 

3、加锁和解锁是同一把锁 

线程  工作内存  、主内存

八种操作：

![image-20200529113812390](C:\Users\64816\AppData\Roaming\Typora\typora-user-images\image-20200529113812390.png).

### Volatile

> 1、保证可见性



> 2、不保证原子性



> 3、禁止指令重排



### 彻底玩转单例模式

> 饿汉式

```java
public class Hungry {
    private Hungry(){
    }

    private final static Hungry HUNGRY = new Hungry();

    public static Hungry getInstance(){
        return HUNGRY;
    }
}
```

> DCL懒汉式

```java
public class LazyMan {
    private static boolean flag = false;
    private LazyMan(){
//        System.out.println(Thread.currentThread().getName());
        //解决用一次反射破坏
//        if(man != null){
//            throw new RuntimeException();
//        }
        //解决使用两次反射构造
        if(!flag){
            flag = true;
        }else{
            throw new RuntimeException();
        }
    }

    private volatile static LazyMan man;
    // 双重检测锁模式的 懒汉式单例  DCL懒汉式
    public static LazyMan getInstance(){
        if(man == null){
            synchronized (LazyMan.class){
                if(man == null){
                    man = new LazyMan();//不是一个原子性操作
                    /**
                     * 1. 分配内存空间
                     * 2、执行构造方法，初始化对象
                     * 3、把这个对象指向这个空间
                     *  123
                     *  132 A
                     *      B // 此时lazyMan还没有完成构造
                     **/
                }
            }
        }
        return man;
    }

    public static void main(String[] args) throws IllegalAccessException, InvocationTargetException,
            InstantiationException, NoSuchMethodException {
        //LazyMan man = LazyMan.getInstance();
        //使用双重锁检测模式解决
//        for (int i = 0; i < 5; i++) {
//            new Thread(()->{
//                LazyMan.getInstance();
//            }).start();
//        }
        Constructor<LazyMan> constructor = LazyMan.class.getDeclaredConstructor(null);
        constructor.setAccessible(true);
        LazyMan man1 = constructor.newInstance();
        LazyMan man2 = constructor.newInstance();
        System.out.println(man2);
        System.out.println(man1);
    }
}
```

> 静态内部类

```java
public class Holder {
    private Holder(){

    }
    public static Holder getInstance(){
        return InnerClass.HOLDER;
    }
    private static class InnerClass{
        private static final Holder HOLDER = new Holder();
    }
}
```

> 枚举

```java
public enum MyEnum {
    INSTANCE;
    public static MyEnum getInstance(){
        return INSTANCE;
    }

    public static void main(String[] args) throws IllegalAccessException, InvocationTargetException,
            InstantiationException, NoSuchMethodException {
        MyEnum myEnum = MyEnum.getInstance();
        System.out.println(myEnum);
        // NoSuchMethodException: Single.MyEnum.<init>()
        //使用jad反编译可以看到源码中并没有空参的构造方法
        //Constructor<MyEnum> constructor = MyEnum.class.getDeclaredConstructor(null);
        //java.lang.IllegalArgumentException: Cannot reflectively create enum objects
        Constructor<MyEnum> constructor = MyEnum.class.getDeclaredConstructor(String.class,int.class);
        constructor.setAccessible(true);
        MyEnum myEnum1 = constructor.newInstance();
        System.out.println(myEnum1);
    }
}
```

使用jad反编译枚举类型的源码

```java
public final class MyEnum extends Enum
{

    public static MyEnum[] values()
    {
        return (MyEnum[])$VALUES.clone();
    }

    public static MyEnum valueOf(String name)
    {
        return (MyEnum)Enum.valueOf(Single/MyEnum, name);
    }

    private MyEnum(String s, int i)
    {
        super(s, i);
    }

    public static MyEnum getInstance()
    {
        return INSTANCE;
    }

    public static void main(String args[])
        throws IllegalAccessException, InvocationTargetException, InstantiationException, NoSuchMethodException
    {
        MyEnum myEnum = getInstance();
        System.out.println(myEnum);
        Constructor constructor = Single/MyEnum.getDeclaredConstructor(new Class[] {
            java/lang/String, Integer.TYPE
        });
        constructor.setAccessible(true);
        MyEnum myEnum1 = (MyEnum)constructor.newInstance(new Object[0]);
        System.out.println(myEnum1);
    }

    public static final MyEnum INSTANCE;
    private static final MyEnum $VALUES[];

    static 
    {
        INSTANCE = new MyEnum("INSTANCE", 0);
        $VALUES = (new MyEnum[] {
            INSTANCE
        });
    }
}
```

























