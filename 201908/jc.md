# Java并发

并发：

并行：

## 线程 Thread

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

## 线程安全

当多个线程访问某个类时，不管运行时环境采用何种调度方式或者这些线程将如何交替执行，并且在主调代码中不需要任何额外的同步或协调，这个类都能表现出正确的行为，那么就称这个类时线程安全的。

无状态对象一定是线程安全的

竞态条件：由于不恰当的执行时许而出现不正确的结果。常见的类型有“先检查后执行”，“读取-修改-写入”等操作类型。

要保证状态一致性，就需要在单个原子操作中更新所有相关的状态变量。

### 死锁

死锁：每个人都拥有其他人需要的资源，同时又等待其他人已经拥有的资源，并且每个人在获得所需要的资源前都不会放弃已经拥有的资源

顺序死锁：两个线程试图以不同的顺序来获得相同的锁，如果按照相同的顺序请求锁，那么就不会产生这种情况

资源死锁：和顺序死锁类似，是两个资源池之间的请求资源顺序不同造成的死锁。

死锁：

* 通过定时锁避免

饥饿：某个线程长时间无法访问他所需要的资源而不能继续执行造成的，资源主要指CPU时钟周期。一个主要的原因可能是调整了线程优先级造成的某些低优先级线程长时间没有获得CPU始终周期

活锁：某个线程没有阻塞，但也不能继续执行，因为线程将不断重复执行相同的操作，而且总会失败。

## 同步

### volatile

volatile是Java虚拟机提供的轻量级同步机制：

- 影响修饰的变量的可见性，即任何线程对变量的改动都会立即反映到其它所有线程中
- 禁止指令重排序优化

### synchronized

wait，notify，notifyAll：wait会自动释放当前锁，并请求操作系统挂起当前线程，从而使其它线程能够获得这个锁并修改对象的状态，当被挂起的线程醒来时，它将在返回之前重新获取锁。

### Lock

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

> 显示锁Lock并不是替代内置锁的一种方式，而是当内置加锁机制不适用时，作为一种可选择的高级功能

读写锁：

```java
public interface ReadWriteLock {
    Lock readLock();
    Lock writeLock();
}
```

## 并发框架

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

