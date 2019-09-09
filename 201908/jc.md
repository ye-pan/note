# Java并发

## 并发

一个可以同时处理多个任务的程序，就是并发程序。一个JVM和其底层操作系统提供了从表象的同时性到物理的并行性的映射。

## Java线程

线程是一个独立于其它线程执行的调用序列，共享进程资源。Java用Thread表示线程对象。

Java程序至少包括一个线程，运行main方法。

线程调度：

* 协同式线程调度，线程执行时间由线程本身控制。好处，实现简单，没有什么线程同步问题，缺点，线程执行时间不可控，一直不告知系统线程切换，那么程序就会一直阻塞在那里。
* 抢占式线程调度，由系统来分配线程的执行时间，线程切换由系统决定。Java使用抢占式来实现多线程的调度的。

Java线程状态，任意时间点，一个线程只能有一种状态：

* 新建，NEW，创建尚未启动的线程
* 运行，RUNNABLE，可能正在执行，也可能正在等待分配CPU执行时间
* 无限期等待，WAITING，不会被分配执行时间，要等待被其它线程显示唤醒。Object.wait，Thread.join，LockSupport.park等方法
* 限期等待，TIME_WAITING，不会被分配执行时间，在等待指定时间后由系统自动唤醒。Object.wait(time)，Thread.join(time)，LockSupport.parkNanos(time)，LockSupport.parkUntil(time)
* 阻塞，BLOCKED，发生在同步等待锁的时候。
* 结束

Thread的一些接口：

wait：有InterruptedException，释放目标对象的锁，将该线程放在目标对象等待集合中

wait(time)：定时版本的wait，超时后自动唤醒，wait(0)和wait(0,0)类似wait

notify：JVM从目标对象等待集合中随意唤醒一个线程

notifyAll：和notify类似，但是唤醒的时所有等待集合中的线程，但因为这些线程需要获取同一个锁，最总也只有一个线程能成功

interrupt：请求结束执行Thread对象。不会真正的立即结束线程，而是发出中断申请，然后由线程在下一个合适的时刻中断自己。如：wait，sleep，join等会抛出InterruptedException的方法。

interrupted：interrupted将清除中断标志的值

isInterrupted：isInterrupted和interrupted一样，只是不清除标志值

sleep：

join：暂定调用线程的执行，直到调用该方法的线程执行结束未知。例A等待B执行结束，在A中B.join就行了

setUncaughtExceptionHandler：当线程出现未校验异常时，该方法用于建立未校验异常的控制器

start：启动线程，重复调用抛出InvalidThreadStateException

setPriority/getPriority：当活动的线程数目多于系统所能够使用的CPU数目时，调度程序通常会偏向运行那些有着更高优先级的进程。



## Java内存模型

Java内存模型用于屏蔽掉各种硬件和操作系统的内存访问差异，以实现让程序在各种平台下都能达到一致的内存访问效果。主要定义程序中各个变量的访问规则，即再虚拟机中将变量存储到内存和从内存中取出变量这样的底层细节。包括，实例字段，静态字段和构成数组对象的元素，不包括局部变量与方法参数，这是线程私有的。

Java内存模型规定了所有变量都存储在主内存中，每条线程还有自己的工作内存，线程的工作内存中保存了被该线程使用到的变量的主内存副本拷贝，线程对变量的所有操作都必须在工作内存中进行，而不能直接读写主存中的变量。不同的线程之间也无法直接访问对方工作内存中的变量，线程间变量值的传递需要通过主内存来完成。

JMM定义的8种操作，用于变量在主内存和工作内存之间同步：

* lock：作用于主内存，把一个变量标识为一条线程独占
* unlock：作用于主内存，把处于锁定状态的变量释放出来
* read：作用于主内存，把一个变量从主内存传输到线程的工作内存中
* load：作用于工作内存，把read操作得到的变量值放入工作内存的变量副本中
* use：作用于工作内存，把工作内存中一个变量值传递给执行引擎
* assign：作用于工作内存，把从执行引擎接收到的值赋给工作内存的变量
* store：作用于工作内存，把工作内存中一个变量值传送到主内存中
* write：作用于主内存，把store操作从工作内存中得到的变量值放入主内存中



读：主内存->read->load->工作内存->use->执行引擎

写：执行引擎->assign->工作内存->store->write->主内存

指令重排：在编译器中生成的指令顺序，可以与源代码中的顺序不同，此外编译器还会把变量保存在寄存器而不是内存中，处理器可以采用乱序或并行等方式来执行指令，缓存可能会改变将写入变量提交到主内存的次序，保存在处理器本地缓存中的值对其它处理器是不可见的。Java语言规范要求JVM在线程中维护一种类似串行的语义，只要程序最终结果与在严格串行环境中执行的结果相同，那么上述操作都是允许的。

Java内存模型是通过各种操作来定义的，包括对变量的读/写，监视器的加锁和释放，以及线程的启动/合并。JMM为程序中所有的操作定义了一个偏序关系，称之为Happens-Before。要想保证执行操作B的线程看到操作A的结果（无论A和B是否在同一个线程中执行），那么在A和B之间必须满足Happens-Before关系，如果两个操作之间缺乏Happends-Before关系，那么JVM可以对它们任意地重排序。

Happens-Before关系：

* 程序顺序规则，如果程序中操作A在操作B之前，那么在线程中A操作将在B操作之前执行
* 监视器锁规则，在监视器锁上的解锁操作必须在同一个监视器锁上的加锁操作之前执行
* volatile变量规则，对volatile变量的写入操作必须在对该变量的读操作之前执行。原子变量也有相同语义
* 线程启动规则，在线程上对Thread.start的调用必须在该线程中执行任何操作之前执行
* 线程结束规则，在线程中的任何操作都必须在其他线程检测到该线程已经结束之前执行，或者从Thread.join中成功返回，或者在调用Thread.isAlive时返回false
* 中断规则，当一个线程在另一个线程上调用interrupt时，必须被中断线程检测到interrupt调用之前执行
* 终结器规则，对象的构造函数必须在启动该对象的终结器之前执行
* 传递性，如果操作A在操作B之前执行，并且操作B在操作C之前执行，那么操作A必须在操作C之前执行

## 线程安全

当多个线程访问某个类时，不管运行时环境采用何种调度方式或者这些线程将如何交替执行，并且在主调代码中不需要任何额外的同步或协调，这个类都能表现出正确的行为，那么就称这个类时线程安全的。

>  无状态对象一定是线程安全的

竞态条件：由于不恰当的执行时序而出现不正确的结果。特别是多线程程序中很容易造成这个问题，常见的类型有“先检查后执行”，“读取-修改-写入”等操作类型。要保证状态一致性，就需要在单个原子操作中更新所有相关的状态变量。

### 活跃性

死锁：每个人都拥有其他人需要的资源，同时又等待其他人已经拥有的资源，并且每个人在获得所需要的资源前都不会放弃已经拥有的资源

顺序死锁：两个线程试图以不同的顺序来获得相同的锁，如果按照相同的顺序请求锁，那么就不会产生这种情况

资源死锁：和顺序死锁类似，是两个资源池之间的请求资源顺序不同造成的死锁。

死锁：

* 通过定时锁避免

饥饿：某个线程长时间无法访问他所需要的资源而不能继续执行造成的，资源主要指CPU时钟周期。一个主要的原因可能是调整了线程优先级造成的某些低优先级线程长时间没有获得CPU始终周期

活锁：某个线程没有阻塞，但也不能继续执行，因为线程将不断重复执行相同的操作，而且总会失败。

### 发布/逸出

发布就是对象能够在当前作用域之外的代码中使用，而当某个不应该发布的对象被发布就称之为逸出。

### 主要修复手段

当多线程程序共享的访问可变的状态变量时，没有使用合适的同步，那么就有线程问题，主要修复手段：

* 不在线程间共享该状态，即让该变量线程封闭
* 将状态变量修改为不可变的变量
* 在访问状态变量时使用同步

线程封闭：让数据仅在线程内访问，不共享。比如Spring的事务管理里面就大量使用了ThreadLocal确保同一线程使用的是一个资源。

* Ad-hoc线程封闭，指维护线程封闭的子这完全由程序实现来承担
* 栈封闭，
* ThreadLocal类，

## 同步

### Java内置的同步机制

volatile是Java虚拟机提供的轻量级同步机制：

- 影响修饰的变量的可见性，即任何线程对变量的改动都会立即反映到其它所有线程中
- 禁止指令重排序优化
- 没加锁，所以不会阻塞线程

synchronized：

* 影响所修饰代码块的内存可见性
* 禁止所修饰代码块的指令重排
* 以互斥的方式访问所修饰的代码块，会造成阻塞

### Java同步集合

### JVM引入的锁优化

自旋锁：共享数据的锁定状态只会持续很短的一段时间，为了这段时间去挂起和恢复线程并不值得，如果物理机器有一个以上的处理器，能让两个或以上的线程同时并行执行，我们就可以让后面请求锁的哪个线程“稍等一下”，但不放弃处理器的执行，看看持有锁的线程是否很快就会释放锁，为了让线程等待，只需要让线程执行一个忙循环。一般自旋默认值是10次。

自适应自旋锁：JDK1.6引入的，自旋时间不再固定，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定的。就是JVM根据实际 — 对象上锁的获取情况来决定是自旋还是挂起等待。

锁消除：JVM即时编译器在运行时，对一些代码上要求同步，但是被检测到不可能存在共享数据竞争的锁进行消除

锁粗化：如果一些列的连续操作都对同一个对象反复加锁和解锁，甚至加锁操作出现在循环体内，那即使没有线程竞争，频繁地进行互斥同步操作也会导致不必要地性能损耗，JVM探测到这样一串零碎地操作都对同一个对象加锁，将会把锁同步地范围扩展到整个操作序列外部。

轻量级锁：

偏向锁：

## Java并发库

### Lock

lock和内置的synchronized一样，只是提供了除基本lock/unlock功能之外的其它一些功能，比如tryLock/tryLock(time)，lockInterruptibly，newCondition等功能。显示锁Lock并不是替代内置锁的一种方式，而是当内置加锁机制不适用时，作为一种可选择的高级功能。

```java
public interface Lock {    
    void lock();
    void lockInterruptibly() throws InterruptedException;
    boolean tryLock();
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    void unlock();
    Condition newCondition();
}
```

读写锁：

```java
public interface ReadWriteLock {
    Lock readLock();
    Lock writeLock();
}
```

### 原子对象



### 线程池

Executor，ExecutorService，ThreadPoolExecutor

多线程程序中大量创建线程问题：

* 线程生命周期的开销很高，创建和销毁
* 资源消耗，活跃的线程会消耗系统资源，尤其是内存。
* 稳定性，一个进程可创建的线程的数量有效

executor解耦任务的提交与执行，基于生产-消费者模式，提交任务的操作相当于生产者，执行任务的线程则相当于消费者。

Executors的几个工厂方法：

newFixedThreadPool：创建一个固定长度的线程池，代码

```java
new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>())
```

newCachedThreadPool：创建一个缓存线程池，随着任务数增长创建对应线程去执行，代码

```java
new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
```

newSingleThreadExecotr：创建一个单线程的executor，代码

```java
new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
```

newScheduledThreadPool：创建了一个固定长度的线程池，而且用于执行定时任务，代码

```java
return new DelegatedScheduledExecutorService
            (new ScheduledThreadPoolExecutor(1));
```

## 参考

* 《Java并发编程实践》
* 《操作系统导论》
* 《精通Java并发编程》
* 《Java并发编程：设计原则与模式》