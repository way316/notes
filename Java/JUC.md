



# JUC

## Synchronized详细介绍

### Sunchronized 简介

作用：能够保证在同一时刻最多只有一个线程执行该段代码。

对象锁：包括方法锁和同步代码块锁

类锁：修饰静态方法或者指定锁为Class对象。

一个类只会有一个Class对象。

一句话总结：JVM会自动通过使用monitor来加锁和解锁，保证了同时只有一个线程可以执行指定代码，从而保证了线程安全，同时具有可重入和不可中断的性质。

### 多线程访问同步方法的七种情况

1. 两个线程同时访问一个对象的同步方法：阻塞
2. 两个线程访问的是两个对象的同步方法：不阻塞
3. 两个线程访问的是Synchronized的静态方法:阻塞
4. 同时访问同步方法与非同步方法：不阻塞
5. 访问同一个对象的不同的非静态方法：阻塞
6. 同时访问静态和非静态方法：不阻塞
7. 方法抛出异常之后是否会抛出锁：会



### 可重入锁

可以获取锁里面的锁，常用在递归方法中，Synchronized是支持可重入锁的

### 底层

通过字节码可以看到最后原理就是调用了monitorenter和monitorexit指令来实现锁



可重入锁原理：JVM会记录被加锁的次数，第一次加锁时候次数会从0变为1，之后再次加锁，就从1变2.退出一层同步代码块计数就会减一，当计数为0的时候，代表锁释放了。

可见性问题：使用Synchronized加的锁可以保证代码的可见性，确保不同线程的操作能同步到所有线程。

### 注意点

1. 锁的范围不应该特别大，防止效率过低
2. 避免锁的嵌套，容易产生死锁。

## JUC基础概念

多线程设计步骤：

![image-20211009113943856](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211009113943856.png)

### 1. 线程状态：

```
NEW				创建
RUNNABLE		可执行
BLOCKED			阻塞
WAITING			不见不散
TIMED_WAITING	过时不候
TERMINATED		终结
```

### 2. Sleep 和 Wait 的区别

（1）sleep是Thread的静态方法，wait是Object的方法，任何对象实例都能调用。
（2）sleep不会释放锁，它也不需要占用锁。wait会释放锁，但调用它的前提是当前线程占有锁(即代码要在synchronized中)。
（3）它们都可以被interrupted方法中断。

### 3.并行与并发

并发：同一时刻多个线程在访问同一个资源，多个线程对一个点
例子：春运抢票 电商秒杀...
并行：多项工作一起执行，之后再汇总
例子：泡方便面，电水壶烧水，一边撕调料倒入桶中

### 4.管程

保证了同一时刻只有一个进程在管程内活动,即管程内定义的操作在同一时刻只被一个进程调用.

执行线程首先要持有管程对象，然后才能执行方法，当方法完成之后会释放管程，方法在执行时候会持有管程，其他线程无法再获取同一个管程



## Lock接口

### Synchronized

1. 修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的代码，作用的对象是调用这个代码块的对象；
2. 修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象；
3. 修改一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个类的所有对象；
4. 修改一个类，其作用的范围是synchronized后面括号括起来的部分，作用的对象是这个类的所有对象。

不能被继承。

![image-20211009103911995](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211009103911995.png)

### Lock

Lock与的Synchronized区别

• Lock不是Java语言内置的，synchronized是Java语言的关键字。Lock是一个类，通过这个类可以实现同步访问；
• 采用synchronized不需要用户去手动释放锁，当synchronized方法或者synchronized代码块执行完之后，系统会自动让线程释放对锁的占用；而Lock则必须要用户去手动释放锁，如果没有主动释放锁，就有可能导致出现死锁现象。



#### newCondition：

​    //定义锁
​    static Lock lock = new ReentrantLock();
​    //获得Condtion对象
​    static Condition condition = lock.newCondition();

Condition比较常用的两个方法：
• await()会使当前线程等待,同时会释放锁,当其他线程调用signal()时,线程会重新获得锁并继续执行。
• signal()用于唤醒一个等待的线程。



#### ReentrantLock:

可重入锁：



#### ReadWriteLock:

两个方法，一个用来获取读锁，一个用来获取写锁。



#### Lock和synchronized的不同：

1. Lock是一个接口，而synchronized是Java中的关键字，synchronized是内置的语言实现；
2. synchronized在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而Lock在发生异常时，如果没有主动通过unLock()去释放锁，则很可能造成死锁现象，因此使用Lock时需要在finally块中释放锁；
3. Lock可以让等待锁的线程响应中断，而synchronized却不行，使用synchronized时，等待的线程会一直等待下去，不能够响应中断；
4. 通过Lock可以知道有没有成功获取锁，而synchronized却无法办到。
5. Lock可以提高多个线程进行读操作的效率。
6. 应该优先使用Synchronized因为lock需要手动操作可能会出错



## 线程之间通信

使用synchronized会有虚假唤醒问题。

synchronized方案 (必须把wait放在while里而不是if里，因为if执行一次之后就离开了等待状态。

)

![image-20211009120806727](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211009120806727.png)

Lock方案

![image-20211009120836127](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211009120836127.png)

![image-20211009120844679](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211009120844679.png)



线程间定制化通信：

创建多个condition,每个condition控制一把锁，让不同线程之间调用锁。





### 集合线程安全

Arraylist线程不安全问题：

![image-20211009121826330](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211009121826330.png)

报错：java.util.ConcurrentModificationException

原因：

![image-20211009121849743](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211009121849743.png)

三种解决方案：

1. Vector

   ![image-20211009123946877](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211009123946877.png)

2. Collections提供了方法synchronizedList

   ![image-20211009123933934](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211009123933934.png)

3. CopyOnWriteArrayList (重点)

   和Arraylist的区别：

   1. 它最适合于具有以下特征的应用程序：List 大小通常保持很小，只读操作远多于可变操作，需要在遍历期间防止线程间的冲突。
   2. 它是线程安全的。
   3. 因为通常需要复制整个基础数组，所以可变操作（add()、set() 和 remove() 等等）的开销很大。
   4. 迭代器支持hasNext(), next()等不可变操作，但不支持可变 remove()等操作。
   5. 使用迭代器进行遍历的速度很快，并且不会与其他线程发生冲突。在构造迭代器时，迭代器依赖于不变的数组快照。

   特点：

   1. 独占锁效率低：采用读写分离思想解决
   2. 写线程获取到锁，其他写线程阻塞
   3. 复制思想

   添加时候，先进行一次复制，复制之后的list添加数据，然后再修改原来的数据，在开始之前会使用lock，在结束的时候释放lock，保证每次只有一个线程在添加。





HashSet线程不安全：使用CopyOnWriteArraySet

HashMap线程不安全：使用ConcurrentHashMap



## 多线程锁

### synchronized:

对于普通同步方法，锁是当前实例对象。
对于静态同步方法，锁是当前类的Class对象。（这个时候如果用了实体类调用其他方法是不会被锁的）
对于同步方法块，锁是Synchonized括号里配置的对象

### 公平锁，非公平锁：

非公平锁：ReentrantLock(false)，会出现一个线程把所有事情都干了，其他线程饿死，执行效率比较高,默认为非公平锁。

公平锁：ReentrantLock(trye)，会尽量让所有线程都干活，但是执行效率比较低。



### 可重入锁：

synchronized和lock都是可重入锁



![image-20211009142435781](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211009142435781.png)

### 死锁

两个或者两个以上的进程，因为争夺资源而造成一种互相等待的现象，如果没有外界干涉就无法继续执行下去。

产生原因：

1. 系统资源不足
2. 进程运行推进顺序不合适
3. 资源分配不当

![image-20211009150414391](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211009150414391.png)

验证死锁：

jps，这个命令类似linux ps -ef,显示jvm里的类

jstack 类号码 	jvm自带追踪堆栈的跟踪工具



## Callable&Future 接口

Callable接口改进了Runnable，允许线程返回结果。

• 为了实现 Runnable，需要实现不返回任何内容的run（）方法，而对于
Callable，需要实现在完成时返回结果的call（）方法。
• call（）方法可以引发异常，而run（）则不能。
• 为实现 Callable 而必须重写call 方法
• 不能直接替换 runnable,因为Thread 类的构造方法根本没有Callable

![image-20211009152139918](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211009152139918.png)



![image-20211009152720278](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211009152720278.png)

通过FutureTask使用callable：

![image-20211009154336883](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211009154336883.png)

Future Task 重点

• 当主线程将来需要时，就可以通过Future 对象获得后台作业的计算结果或者执行状态

• 一般FutureTask 多用于耗时的计算，主线程可以在完成自己的任务后，再去
获取结果。
• 仅在计算完成时才能检索结果；如果计算尚未完成，则阻塞 get 方法。一旦计
算完成，就不能再重新开始或取消计算。get 方法而获取结果只有在计算完成
时获取，否则会一直阻塞直到任务转入完成状态，然后会返回结果或者抛出异
常。
• get 只计算一次,因此get 方法放到最后



## JUC 三大辅助类

• CountDownLatch: 减少计数
• CyclicBarrier: 循环栅栏
• Semaphore: 信号灯

### CountDownLatch：

保证线程结束之后再运行后面的,如果CountDown没有变为0，是不会执行countdown后面的方法的。

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



### CyclicBarrier：

可以将CyclicBarrier 理解为加1 操作,达到指定条件才能执行

```Java
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

### Semaphore：

Semaphore 的构造方法中传入的第一个参数是最大信号量（可以看成最大线程池），每个信号量初始化为一个最多只能分发一个许可证。使用.acquire()获取许可证，使用.relsease()扔掉许可证

```java
//6辆汽车，停3个车位
public class SemaphoreDemo {
    public static void main(String[] args) {
        //创建Semaphore，设置许可数量
        Semaphore semaphore = new Semaphore(3);

        //模拟6辆汽车
        for (int i = 1; i <=6; i++) {
            new Thread(()->{
                try {
                    //抢占
                    semaphore.acquire();

                    System.out.println(Thread.currentThread().getName()+" 抢到了车位");

                    //设置随机停车时间
                    TimeUnit.SECONDS.sleep(new Random().nextInt(5));

                    System.out.println(Thread.currentThread().getName()+" ------离开了车位");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    //释放
                    semaphore.release();
                }
            },String.valueOf(i)).start();
        }
    }
}
```

## 读写锁

悲观锁与乐观锁的区别：

悲观锁不允许多线程操作，乐观锁允许，但是通过版本号控制，如果版本号不对，就不允许提交事务。

![image-20211010100330943](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211010100330943.png)

表锁和行锁:

锁定整个表以及锁定其中一行，行锁更好的支持并发。



读写锁：

读锁死锁：

![image-20211010104053637](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211010104053637.png)

写锁死锁：

![image-20211010104204121](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211010104204121.png)



1. 线程进入读锁的前提条件：
• 没有其他线程的写锁
• 没有写请求, 或者有写请求，但调用线程和持有写锁的线程是同一个(可重入锁)。

2. 线程进入写锁的前提条件：
• 没有其他线程的读锁
• 没有其他线程的写锁

重要特性：
（1）公平选择性：支持非公平（默认）和公平的锁获取方式，吞吐量还是非公平优于公平。
（2）重进入：读锁和写锁都支持线程重进入。
（3）锁降级：遵循获取写锁、获取读锁再释放写锁的次序，写锁能够降级成为读锁。

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

![image-20211010110750995](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211010110750995.png)

锁降级：

![image-20211010110917176](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211010110917176.png)

原因：获取读锁的时候可能有其他线程占有写锁所以不能升级，有写锁的线程肯定独占写锁所以获取读锁也没有关系。



## 阻塞队列

• 当队列中没有数据的情况下，消费者端的所有线程都会被自动阻塞（挂起），直到有数据放入队列
• 当队列中填满数据的情况下，生产者端的所有线程都会被自动阻塞（挂起），直到队列中有空的位置，线程被自动唤醒

主要有两种：

• 先进先出（FIFO）：先插入的队列的元素也最先出队列，类似于排队的功能。从某种程度上来说这种队列也体现了一种公平性
• 后进先出（LIFO）：后插入队列的元素最先出队列，这种队列优先处理最近发生的事件(栈)

ArrayBlockingQueue和LinkedBlockingQueue，由数组组成的阻塞队列和链表组成的阻塞队列，ArrayBlockingQUEUE没有使用分离锁，LinkedBlockingQueue采用了分离式锁。

DelayQueue:使用优先级队列实现的延迟无界阻塞队列,是一个没有大小限制的队列，因此往队列中插入数据的操作（生产者）永远不会被阻塞

PriorityBlockingQueue：支持优先级排序的无界阻塞队列，不会阻塞数据生产者

SynchronousQueue: 不存储元素的阻塞队列，也即单个元素的队列，和不用队列类似。

• 公平模式：SynchronousQueue会采用公平锁，并配合一个FIFO队列来阻塞多余的生产者和消费者，从而体系整体的公平策略；
• 非公平模式（SynchronousQueue默认）：SynchronousQueue采用非公平锁，同时配合一个LIFO队列来管理多余的生产者和消费者，而后一种模式，如果生产者和消费者的处理速度有差距，则很容易出现饥渴的情况，即可能有某些生产者或者是消费者的数据永远都得不到处理。

LinkedTransferQueue: 由链表组成的无界阻塞队列。

LinkedBlockingDeque: 由链表组成的双向阻塞队列

![image-20211010141318797](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211010141318797.png)

## 线程池

线程池做的工作只要是控制运行的线程数量，处理过程中将任务放入队列，然后在线程创建后启动这些任务，如果线程数量超过了最大数量，超出数量的线程排队等候，等其他线程执行完毕，再从队列中取出任务来执行。

• 降低资源消耗: 通过重复利用已创建的线程降低线程创建和销毁造成的销耗。
• 提高响应速度: 当任务到达时，任务可以不需要等待线程创建就能立即执行。
• 提高线程的可管理性: 线程是稀缺资源，如果无限制的创建，不仅会销耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。                                                                                                                                                                               • Java中的线程池是通过Executor框架实现的，该框架中用到了Executor，Executors，ExecutorService，ThreadPoolExecutor这几个类

![image-20211010141847396](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211010141847396.png)

### 线程池种类

1. newCachedThreadPool: 创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程.

• 线程池中数量没有固定，可达到最大值（Interger. MAX_VALUE）
• 线程池中的线程可进行缓存重复利用和回收（回收默认时间为1分钟）
• 当线程池中，没有可用线程，会重新创建一个线程

适用于创建一个可无限扩大的线程池，服务器负载压力较轻，执行时间较短，任务多的场景

2. newFixedThreadPool(常用):创建一个可重用固定线程数的线程池，以共享的无界队列方式来运行这些线程。

   • 线程池中的线程处于一定的量，可以很好的控制线程的并发量
   • 线程可以重复被使用，在显示关闭之前，都将一直存在
   • 超出一定量的线程被提交时候需在队列中等待

   适用于可以预测线程数量的业务中，或者服务器负载较重，对线程数有严格限制的场景

3. newSingleThreadExecutor：创建一个使用单个 worker 线程的 Executor，以无界队列方式来运行该线程。

   • 线程池中最多执行1个线程，之后提交的线程活动将会排在队列中以此执行

   适用于需要保证顺序执行各个任务，并且在任意时间点，不会同时有多个线程的场景

4. newScheduleThreadPool：线程池支持定时以及周期性执行任务，创建一个corePoolSize为传入参数，最大线程数为整形的最大数的线程池

   • 线程池中具有指定数量的线程，即便是空线程也将保留

   • 可定时或者延迟执行线程活动

   适用于需要多个后台线程执行周期任务的场景

5. newWorkStealingPool:创建一个拥有多个任务队列的线程池，可以减少连接数，创建当前可用cpu核数的线程来并行执行任务

   适用于大耗时，可并行执行的场景

   ```java
   //演示线程池三种常用分类
   public class ThreadPoolDemo1 {
       public static void main(String[] args) {
           //一池五线程
           ExecutorService threadPool1 = Executors.newFixedThreadPool(5); //5个窗口
   
           //一池一线程
           ExecutorService threadPool2 = Executors.newSingleThreadExecutor(); //一个窗口
   
           //一池可扩容线程
           ExecutorService threadPool3 = Executors.newCachedThreadPool();
           //10个顾客请求
           try {
               for (int i = 1; i <=10; i++) {
                   //执行
                   threadPool3.execute(()->{
                       System.out.println(Thread.currentThread().getName()+" 办理业务");
                   });
               }
           }catch (Exception e) {
               e.printStackTrace();
           }finally {
               //关闭
               threadPool3.shutdown();
           }
   
       }
   
   }
   ```

   线程池都是调用同一个方法，只是参数不一样

   ![image-20211010143904380](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211010143904380.png)

### 线程池七大参数

• corePoolSize线程池的常驻线程数
• maximumPoolSize能容纳的最大线程数
• keepAliveTime空闲线程存活时间
• unit 存活的时间单位
• workQueue 存放提交但未执行任务的队列
• threadFactory 创建线程的工厂类
• handler 等待队列满后的拒绝策略

当提交的任务数大于（workQueue.size() + maximumPoolSize ），就会触发线程池的拒绝策略。

### 线程池的工作流程

1. 在创建了线程池后，线程池中的线程数为零

2. 当调用execute()方法添加一个请求任务时，线程池会做出如下判断： 

   2.1 如果正在运行的线程数量小于corePoolSize，那么马上创建线程运行这个任务； 

   2.2 如果正在运行的线程数量大于或等于corePoolSize，那么将这个任务放入队列；

   2.3 如果这个时候队列满了且正在运行的线程数量还小于maximumPoolSize，那么还是要创建非核心线程立刻运行这个任务；

   2.4 如果队列满了且正在运行的线程数量大于或等于maximumPoolSize，那么线程池会启动饱和拒绝策略来执行。

3. 当一个线程完成任务时，它会从队列中取下一个任务来执行

4. 当一个线程无事可做超过一定的时间（keepAliveTime）时，线程会判断： 4.1 如果当前运行的线程数大于corePoolSize，那么这个线程就被停掉。 4.2 所以线程池的所有任务完成后，它最终会收缩到corePoolSize的大小。

### 拒绝策略

![image-20211010145406516](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211010145406516.png)

### 注意事项

1. 项目中创建多线程时，使用常见的三种线程池创建方式，单一、可变、定长都有一定问题，原因是FixedThreadPool和SingleThreadExecutor底层都是用LinkedBlockingQueue实现的，这个队列最大长度为Integer.MAX_VALUE，容易导致OOM。
2. 所以实际生产一般自己通过ThreadPoolExecutor的7个参数，自定义线程池

![image-20211010145735003](C:\Users\Wang\OneDrive\Java全家桶\JUC.assets\image-20211010145735003.png)

```java
        ExecutorService threadPool = new ThreadPoolExecutor(
                2,
                5,
                2L,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.AbortPolicy()
        );
```

## Fork/Join 分支合并框架

将一个大的任务拆分成多个子任务进行并行处理，最后将子任务结果合并成最后的计算结果，并进行输出

```java
class MyTask extends RecursiveTask<Integer> {

    //拆分差值不能超过10，计算10以内运算
    private static final Integer VALUE = 10;
    private int begin ;//拆分开始值
    private int end;//拆分结束值
    private int result ; //返回结果

    //创建有参数构造
    public MyTask(int begin,int end) {
        this.begin = begin;
        this.end = end;
    }

    //拆分和合并过程
    @Override
    protected Integer compute() {
        //判断相加两个数值是否大于10
        if((end-begin)<=VALUE) {
            //相加操作
            for (int i = begin; i <=end; i++) {
                result = result+i;
            }
        } else {//进一步拆分
            //获取中间值
            int middle = (begin+end)/2;
            //拆分左边
            MyTask task01 = new MyTask(begin,middle);
            //拆分右边
            MyTask task02 = new MyTask(middle+1,end);
            //调用方法拆分
            task01.fork();
            task02.fork();
            //合并结果
            result = task01.join()+task02.join();
        }
        return result;
    }
}

public class ForkJoinDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //创建MyTask对象
        MyTask myTask = new MyTask(0,100);
        //创建分支合并池对象
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Integer> forkJoinTask = forkJoinPool.submit(myTask);
        //获取最终合并之后结果
        Integer result = forkJoinTask.get();
        System.out.println(result);
        //关闭池对象
        forkJoinPool.shutdown();
    }
}
```

## CompletableFuture异步回调

CompletableFuture在Java里面被用于异步编程，异步通常意味着非阻塞，可以使得我们的任务单独运行在与主线程分离的其他线程中，并且通过回调可以在主线程中得到异步任务的执行状态，是否完成，和是否异常等信息。

Future的缺点：

（1）不支持手动完成
我提交了一个任务，但是执行太慢了，我通过其他路径已经获取到了任务结果，现在没法把这个任务结果通知到正在执行的线程，所以必须主动取消或者一直等待它执行完成
（2）不支持进一步的非阻塞调用
通过Future的get方法会一直阻塞到任务完成，但是想在获取任务之后执行额外的任务，因为Future不支持回调函数，所以无法实现这个功能
（3）不支持链式调用
对于Future的执行结果，我们想继续传到下一个Future处理使用，从而形成一个链式的pipline调用，这在Future中是没法实现的。
（4）不支持多个Future合并

比如我们有10个Future并行执行，我们想在所有的Future运行完毕之后，执行某些函数，是没法通过Future实现的。
（5）不支持异常处理
Future的API没有任何的异常处理的api，所以在异步运行时，如果出了问题是不好定位的。



```java
//异步调用和同步调用
public class CompletableFutureDemo {
    public static void main(String[] args) throws Exception {
        //同步调用
        CompletableFuture<Void> completableFuture1 = CompletableFuture.runAsync(()->{
            System.out.println(Thread.currentThread().getName()+" : CompletableFuture1");
        });
        completableFuture1.get();

        //mq消息队列
        //异步调用
        CompletableFuture<Integer> completableFuture2 = CompletableFuture.supplyAsync(()->{
            System.out.println(Thread.currentThread().getName()+" : CompletableFuture2");
            //模拟异常
            int i = 10/0;
            return 1024;
        });
        completableFuture2.whenComplete((t,u)->{
            System.out.println("------t="+t);
            System.out.println("------u="+u);
        }).get();
    }
}
```

