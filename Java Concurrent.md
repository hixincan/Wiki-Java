*书山有路勤为径，学海无涯苦作舟*



参考

* http://mishadoff.com/blog/java-magic-part-4-sun-dot-misc-dot-unsafe/



# 背景

在我们平时开发过程中，为了提高性能，或者程序解耦，会引入多线程；

同时在开发web应用时，每个web容器在处理用户请求时，会把用户请求放到线程中执行，

意味着，即使我们不主动使用多线程，我们的程序还是处在多线程环境下。



在不做同步控制的情况下，代码在多线程下是不安全的！



# 并发体系

为了并发安全：互斥同步、非互斥同步、无同步方案
管理线程、提高效率
线程协作





# 并发编程





程序：程序是指令和数据的有序集合，静态的概念

进程：执行程序的一次过程，动态概念。是操作系统分配资源的单位。例子，视频播放（图像、声音、字幕）

线程：一个进程可以包含若干个线程；是CPU调度和执行的单位。



## 进程

操作系统会分配给进程内存、文件资源。进程的切换会保存进程表。



## 线程

进程是线程的容器，操作系统会分配给线程的只有CPU的资源，线程的切换是轻量级的，只有寄存器数据需要保存起来。





## 同步

### 执行同步

多个参与线程并发执行，在某一点汇合（join），执行后续操作



### 数据同步

多份数据保持数据一致





# 线程安全集合

| 集合                | 线程安全 | 实现线程安全的方式                                           | 效率                                 |
| ------------------- | -------- | ------------------------------------------------------------ | ------------------------------------ |
| HashMap             | 不是     | 没有做任何同步措施                                           |                                      |
| HashTable（JDK1.0） | 是       | 在每个方法上添加 synchronized 内置锁                         | 只有一把锁，多线程效率低             |
| ConcurrentHashMap   | 是       | 分段锁，<br />JDK1.7 对数组进行分段，每一段是一把锁<br />JDK1.8 分段锁的颗粒变小，数组每个元素都有自己独立的锁 | 保证性能：让多线程同时并发的个数变多 |





# 线程池

## 介绍线程池

高并发下如果直接用线程会产生的问题？

问题一：反复创建线程开销大
问题二：过多的线程会占用太多内存



解决以上两个问题的思路

* 用少量的线程——避免内存占用过多

* 让这部分线程都保持工作，且可以反复执行任务——避免生命周期的损耗



好处：

* 加快响应速度
* 合理利用CPU和内存
* 统一管理



使用场景

* 服务器接受到大量请求时，使用线程池技术是非常合适的，它可以大大减少线程的创建和销毁次数，提高服务器的工作效率（如 tomcat）
* 在实际开发中，如果需要创建5个以上的线程，那么就可以使用线程池来管理



## 创建线程池



**线程池构造函数**，6个参数

| 参数名        | 类型                     | 含义                                                         |
| ------------- | ------------------------ | ------------------------------------------------------------ |
| corePoolSize  | int                      | 核心线程数，详解见下文                                       |
| maxPoolSize   | int                      | 最大线程数，详解见下文                                       |
| keepAliveTime | long                     | 保持存活时间                                                 |
| workQueue     | BlockingQueue            | 任务存储队列                                                 |
| threadFactory | ThreadFactory            | 当线程池需要新的线程的时候，<br />会使用threadFactory来<br/>生成新的线程 |
| Handler       | RejectedExecutionHandler | 由于线程池无法接受你所提交的任务的拒绝策略                   |

* corePoolSize指的是核心线程数：线程池在完成初始化后，默认情况下，线程池中并没有任何线程，线程池会等待有任务到来时，再创建新线程去执行任务

* maxPoolSize最大量：由于任务量是不均匀的，线程池有可能会在核心线程数的基础上，额外增加一些线程，但是这些新增加的线程数有一个上限

* keepAliveTime：如果线程池当前的线程数多于corePoolSize，那么如果多余的线程空闲时间超过keepAliveTime，它们就会被终止；也可以通过修改参数回收核心线程，但一般不这么做。

* ThreadFactory 用来创建线程：新的线程是由ThreadFactory创建的，默认使用==Executors.defaultThreadFactory==，创建出来的线程都在同一个线程组，拥有同样的NORM_PRIORITY优先级（5）并且都不是守护线程。如果自己指定ThreadFactory，那么就可以==改变线程名、线程组、优先级、是否是守护线程==等。通常，使用默认的线程工厂就能满足需要。
* workQueue 工作队列：
    有3种最常见的队列类型：
    1）直接交接：SynchronousQueue ：队列没有大小，直接使用 maxPoolSize
    2）无界队列：LinkedBlockingQueue  队列没有上限，优点不会丢弃，缺点是任务过多导致资源浪费和 OOM
    3）有界的队列：ArrayBlockingQueue  队列可设置大小



**添加线程规则**

1. 如果线程数小于corePoolSize，即使其他工作线程处于空闲状态，也会创建一个==新线程来运行新任务==。
2. 如果线程数等于（或大于）corePoolSize但少于maxPoolSize，则将任务放入队列。
3. 如果队列已满，并且线程数小于maxPoolSize，则创建一个新线程来运行任务。
4. 如果队列已满，并且线程数大于或等于maxPoolSize，则拒绝该任务

即判断顺序

1. corePoolSize
2. workQueue
3. maxPoolSize



**增减线程的特点**

1. 通过设置corePoolSize和maximumPoolSize相同，就可以创建==固定大小的线程池==。
2. 线程池==希望保持较少的线程数==，并且只有在负载变得很大时才增加它。
3. 通过设置maximumPoolSize为很高的值，例如Integer.MAX_VALUE，可以允许线程池容纳任意数量的并发任务。
4. 是只有在队列填满时才创建多于corePoolSize的线程，所以如果你使用的是无界队列（例如==LinkedBlockingQueue==），那么线程数就不会超过corePoolSize。





**线程池应该手动创建还是自动创建**

手动创建更好，因为这样可以让我们更加明确线程池的运行规则，避免资源耗尽的风险。



让我们来看看自动创建线程池（也就是直接调用JDK封装好的构造函数，在Executors工具类中）可能带来哪些问题

5自动

> newFixedThreadPool

* coreSizePool 和 maxSizePool  为固定大小
* workQueue 为 LinkedBlockingQueue

由于传进去的LinkedBlockingQueue是没有容量上限的，所以当请求数越来越多，并且无法及时处理完毕的时候，也就是请求堆积的时候，会容易造成占用大量的内存，可能会导致OOM。

> SingleThreadExecutor

可以着出，这里和刚才的newFixedThreadPool的原理基本一样，只不过把线程数直接设置成了1，所以这也会导致同样的问题，也就是当请求堆积的时候，可能会占用大量的内存。

> newCachedThreadPool

* coreSizePool 为 0
* maxSizePool 为 Integer.MAX_VALUE
* keepAliveTime 为 60

这可能会创建数量非常多的线程，甚至导致OOM。

> newScheduledThreadPool

定时执行线程

* coreSizePool 为 参数传递
* maxSizePool 为 Integer.MAX_VALUE

* keepAliveTime 60

> workStealingPool（JDK1.8 ）

这个线程池和之前的都有很大不同，适合子任务、窃取。

特点

* 任务不加锁
* 任务不保证顺序



正确的创建线程池的方法
==根据不同的业务场景==，自己设置线程池参数，比如我们的内存有多大，我们想给线程取什么名字（传入ThreadFactory），任务被拒绝之后的日志打印，并发量多大等等



**线程池里的线程数量设定为多少比较合适**？

等价于如何设置线程数，才能充分利用CPU？

* CPU密集型（加密、计算hash等）：最佳线程数为CPU核心数的==1-2==倍左右。

* 耗时IO型（读写数据库、文件、网络读写等）：最佳线程数一般会大于cpu核心数很多倍，以JVM线程监控显示繁忙情况为依据，保证线程空闲可以衔接上
* 参考Brain Goetz推荐的计算方法：线程数===CPU核心数*（1+平均等待时间/平均工作时间）==
* 当然了，最精准的方式还是通过压测





## 停止线程池

线程池执行完后，要手动关闭 threadPool.shutdown();



**5方法**

> shutdown

* ==存量的任务都执行完毕==——将阻塞、等待的任务执行完再关闭
* 新任务拒绝 —— 抛出异常

> isShutdown

* 一旦执行了 shutdown，该方法返回true

> isTerminated

* 线程清空了才返回 true

> awaitTermination

* 阻塞方法

* 检测线程池是否关闭

> shutdownNow

立刻关闭线程池

* 正在执行的任务：收到中断信号 interrupt
* 在队列等待的任务：通过 shutdownNow 方法的方法的返回值，能拿到 runnableList





## 拒绝任务



**拒绝时机**

1. 当Executor关闭时，提交新任务会被拒绝。
2. 以及当Executor对最大线程和工作队列容量使用有限边界并且已经饱和时



**4策略**

> AbortPolicy

直接抛出异常

> DiscardPolicy

静默丢弃

> DiscardOldestPolicy

丢弃最老的任务

> CallerRunsPolicy

让提交任务的线程执行，好处：

* 避免任务损失
* 降低提交任务的速度：调用线程执行也需要时间，为线程池的执行提供了时间





## 钩子方法

每个任务的前后增加方法处理，如日志方法、暂停任务、恢复任务等

TODO 代码示例后面补充



## 实现原理

**线程池组成部分**

* 线程池管理器
* 工作线程
* 任务列队：存放任务的队列，必须支持并发，选用了线程安全的 blockingQueue
* 任务接口（Task）





**接口、类关系**

executor

executorService

abstractorExecutorService

threadPoolexecutor



executors 工具类，创建内置的线程池等方法



**线程池实现任务复用的原理**
相同线程执行不同任务



executor.addWorker 方法

worker.runWork方法 ，拿到task，执行 run 方法



**线程池状态**

1. RUNNING：接受新任务并处理排队任务
2. SHUTDOWN：不接受新任务，但处理排队任务
3. STOP：不接受新任务，也不处理排队任务，并中断正在进行的任务
4. TIDYING：所有任务都已终止，workerCount为零时，线程会转换到TIDYING状态，并将运行terminate（）钩子方法。
5. TERMINATED：terminate（）运行完成



**源码剖析**

TODO 

threadPool.submit 



**使用线程池的注意店**

* 避免任务堆积
* 避免线程数过度增加
* 排查线程泄漏



# ThreadLocal

线程共享进程内存，ThreadLocal就是通过抽象数据结构来实现的线程本地对象，其他线程无法访问和修改。



## 两大场景

* 典型场景1：每个线程需要一个独享的对象（通常是工具类，典型需要使用的类有SimpleDateFormat和Random—— 这些工具类不是线程安全的）

* 典型场景2：每个线程内需要保存全局变量（例如在拦截器中获取用户信息），可以让不同方法直接使用，避免参数传递的麻烦，且没有锁所以非常高效！





**典型场景1**

SimpleDateFormat的进化之路

1. 2个线程分别用自己的SimpleDateFormat，这没问题
2. 后来延伸出10个，那就有10个线程和10个SimpleDateFormat，这虽然写法不优雅（应该复用对象），但勉强可以接受
3. 但是当需求变成了1000个，那么必然要用线程池（否则==消耗内存==太多）
4. 所有的线程都共用同一个simpleDateFormat对象（发生了==线程不安全==问题！！）
5. 用 synchronized 类锁，加锁 dateFormat.format 语句（保证一次只有一个线程执行，==效率低==了！！）
6. 利用ThreadLocal，给每个线程分配自己的 dateFormat对象，保证了线程安全，高效利用内存有

 

**典型场景2**

实例：当前用户信息需要被线程内所有方法共享

* 一个比较繁琐的解决方案是把user作为参数层层传递，从service-10传到service-20，再从service-20传到service-30，以此类推，
* 但是这样做会导致==代码冗余==（重复校验等）且==不易维护==



进化之路

1. 当多线程同时工作时，我们需要保证线程安全，可以用==synchronized==，也可以用==ConcurrentHashMap==，但无论用什么，都会对==性能==有所影响

2. 更好的办法是使用ThreadLocal，这样无需synchronized，可以在不影响性能的情况下，也无需层层传递参数，就可达到保存当前线程对应的用户信息的目的



方法：

* 用ThreadLocal保存一些业务内容（用户权限信息、从用户系统获取到的用户名、userID等）

* 这些信息在==同一个线程内相同==，但是不同的线程使用的业务内容是不相同的

* 在线程生命周期内，都通过这个静态ThreadLocal实例的get 方法取得自己set过的那个对象，避免了将这个对象（例如user对象）作为参数传递的麻烦
* 强调的是同一个请求内（同一个线程内）不同方法间的共享
* 不需重写initialValue0方法，但是必须手动调用set0方法



*开发约定 holder 理解为存放了对象*



**总结**

ThreadLocal的两个作用

1. 让某个需要用到的对象在线程间隔离（每个线程都有自己的独立的对象）
2. 在任何方法中都可以轻松获取到该对象



◆回顾前面的两个场景，对应到这两个作用

场景一：initialValue在ThreadLocal第一次get的时候把对象给初始化出来，==对象的初始化时机由我们控制==

场景二：set如果需要保存到ThreadLocal里的==对象生成时机不由我们随意控制==，例如拦截器生成的用户信息，用ThreadLocal.set直接放到我们的ThreadLocal中去，以便后续使用。



好处

* 达到线程安全
* 不需要加锁，提高执行效率
* 更高效地利用内存、节省开销：相比于每个任务都新建一个SimpleDateFormat，显然用ThreadLocal可以节省内存和开销
* 免去传参的繁琐：无论是场景一的工具类，还是场景二的用户名，都可以在任何地方直接通过ThreadLocal拿到，再也不需要每次都传同样的参数。ThreadLocal使得代码耦合度更低，更优雅







## 实现原理



**ThreadLocal 结构关系**

Thread

* ThreadLocalMap
    * ThreadLocal 



```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

得到：

* 每个线程都有自己的 ThreadLocalMap 
* ThreadLocalMap 存放当前的 threadLocal 对象（this）作为 key
* 一个线程可以使用多个 ThreadLocal 对象



**Thread Local方法**



> initialValue() 方法，初始化

1. 该方法会返回当前线程对应的“初始值”，这是一个==延迟加载==的方法，只有在==调用get==的时候，才会触发
2. 当线程==第一次使用get方法==访问变量时，将调用此方法，除非线程先前调用了set方法，在这种情况下，不会为线程调用本initialValue方法
3. 通常，每个线程最多调用一次此方法，但如果已经调用了remove()后，再调用get(），则可以再次调用此方法
4. 如果不重写本方法，这个方法会返回null。一般使用匿名内部类的方法来重写initialValue）方法，以便在后续使用中可以初始化副本对象。



> get()：得到这个线程对应的value。

* 如果是首次调用get()，则会调用initialize来得到这个值

* get方法是先取出==当前线程==的ThreadLocalMap，然后调用map.getEntry方法，把==当前的ThreadLocal==的引用（this）作为参数传入，取出map中属于本ThreadLocal的value



> void set（T t）：为这个线程设置一个新值

与 get 原理一致



> void remove()：删除对应这个线程的值，

与 get 原理一致





**ThreadLocalMap 结构关系**

ThreadLocalMap 类，也就是Thread.threadLocals 

ThreadLocalMap 类是每个线程Thread类里面的变量

里面最重要的是一个==键值对数组==Entry[] table，可以==认为是一个map==，键值对：
键：这个ThreadLocal

值：实际需要的成员变量，比如user或者simpleDateFormat对象



**ThreadLocalMap内部实现**

|          | ThreadLocalMap                                               | HashMap                                                      |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Hash冲突 | 线性探测法：如果当前数组位有值，<br />则判断下一个数组位是否有值，如果有值继续向下寻找，<br />直到一个为空的数组位 | 拉链法：hash冲突后，会用一个链表往下链，<br />当超过8个后，会编程红黑树 |
|          |                                                              |                                                              |
|          |                                                              |                                                              |



```java
static class ThreadLocalMap {

        /**
         * The entries in this hash map extend WeakReference, using
         * its main ref field as the key (which is always a
         * ThreadLocal object).  Note that null keys (i.e. entry.get()
         * == null) mean that the key is no longer referenced, so the
         * entry can be expunged from table.  Such entries are referred to
         * as "stale entries" in the code that follows.
         */
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
}
```

Entry 的key 是弱引用。



## 内存泄漏

==threadLocal 使用注意点==：有一些要复用的线程，如线程池中的核心线程可能不会被销毁。ThreadLocal 要及时清理不用的值。

ThreadLocal 可能会造成内存泄漏，我们在使用完后，务必要调用 threadLocal.remove()



**内存泄漏的发生**

*key 的泄漏 和  value 的泄漏*



1. ThreadLocalMap的每个Entry 都是一个对key的弱引用，同时，每个Entry 都包含了一个对value的强引用

2. 正常情况下，当线程终止，保存在ThreadLocal里的value会被垃圾回收，因为没有任何强引用了

3. 但是，如果线程不终止（比如线程需要保持很久），那么key对应的value就不能被回收，因为有以下的调用链：

    thread > threadLocalMap > entry(key = null) > value

4. 因为value和Thread之间还存在这个强引用链路，所以导致value无法回收，就可能会出现OOM

5. JDK已经考虑到了这个问题，所以在set，remove，rehash方法中会扫描key为null的Entry，并把对应的value设置为null（调用`expungeStaleEntry`方法），这样value对象就可以被回收

6. 但是如果一个ThreadLocal不被使用，那么实际上set，remove，rehash方法也不会被调用，如果同时线程又不停止，那么调用链就一直存在，那么就导致了==value的内存泄漏==



**避免内存泄漏（阿里规约）**

调用remove方法，就会删除对应的Entry对象，可以避免内存泄漏，所以使用完ThreadLocal之后，应该调用remove方法

要求我们不再使用时调用 remove() 方法，或者用拦截器调用 remove() 方法



## 注意事项

**空指针异常**
在进行get之前，必须先set，否则可能会报空指针异常？



**共享对象**
如果在每个线程中ThreadLocal.set() 进去的东西本来就是多线程共享的同一个对象，比如static对象，那么多个线程的ThreadLocal.get() 取得的还是这个共享对象本身，还是有并发访问问题



**使用收益**

如果可以不使用ThreadLocal就解决问题，那么不要强行使用例如在任务数很少的时候，在局部变量中可以新建对象就可以解决问题，那么就不需要使用到ThreadLocal



**框架优先**

优先使用框架的支持，而不是自己创造
例如在Spring中，如果可以使用`RequestContextHolder`，那么就不需要自己维护ThreadLocal，因为自己可能会忘记调用remove0方法等，造成内存泄漏

Spring 框架实现中大量用到了 threadLocal，案例：

* DateTimeContextHolder类，看到里面用了ThreadLocal

* 每次HTTP请求都对应一个线程，线程之间相互隔离，这就是ThreadLocal的典型应用场景，用 threadLocal 保存了请求属性



# 锁/同步器

## 认识锁

锁是一种工具，用于控制对==共享资源==的访问。

Lock和synchronized，这两个是最常见的锁，它们都可以达到==线程安全==的目的，但是在使用上和功能上又有较大的不同。

Lock并不是用来代替synchronized的，而是当使用synchronized不合适或不足以满足要求的时候，来提供==高级功能==的。







## 锁分类

这些分类，是从各种不同角度出发去看的

这些分类并不是互斥的，也就是多个类型可以并存：有可能一个锁，同时属于两种类型

比如ReentrantLock既是互斥锁，又是可重入锁



<img src="Java%20Concurrent.assets/image-20210706160517714.png" alt="image-20210706160517714" style="zoom:67%;" />	





### **乐观锁和悲观锁**

> 悲观锁 - ==互斥同步锁==

缺点：

* 阻塞和唤醒带来的性能劣势
* 永久阻塞：如果持有锁的线程被永久阻塞，比如遇到了无限循环、死锁等活跃性问题，那么等待该线程释放锁的那几个悲催的线程，将永远也得不到执行
* 优先级反转



如果我不锁住这个资源，别人就会来争抢，就会造成数据结果错误，所以每次悲观锁为了确保结果的正确性，会在每次获取并修改数据时，把数据锁住，让别人无法访问该数据，这样就可以确保数据内容万无一失

Java中悲观锁的实现就是==synchronized和Lock==相关类



> 乐观锁 - 非互斥同步锁：

认为自己在处理操作的时候不会有其他线程来干扰，所以并==不会锁住==被操作对象

在更新的时候，去对比在我修改的期间数据有没有被其他人改变过：如果没被改变过，就说明真的是只有我自己在操作，那我就正常去修改数据，

如果数据和我==一开始拿到的不一样==了，说明其他人在这段时间内改过数据，那我就不能继续刚才的更新数据过程了，我会选择放弃、报错、重试等策略

乐观锁的实现一般都是利用==CAS==算法来实现的，在Java中就是==原子类、并发容器==等



> 开销对比

* 悲观锁的原始开销要高于乐观锁，但是特点是==一劳永逸==，临界区持锁时间就算很长，也不会对互斥锁的开销造成影响
* 相反，虽然乐观锁一开始的开销比悲观锁小，但是如果自旋时间很长或者不停重试，那么==消耗的资源也会越来越多==



> 适用场景

悲观锁：适合==并发写入多==的情况，适用于==临界区持锁时间比较长==的情况，悲观锁可以避免大量的无用自旋等消耗，典型情况：

* 临界区有==IO==操作
* 临界区==代码复杂==或者循环量大
* 临界区==竞争非常激烈==



乐观锁：适合并发==写入少，大部分是读取的场景==，不加锁的能让读取性能大幅提高。





### 可重入锁和非可重入锁

也叫递归锁



特点

* 多次获取==同一把锁==
* 不用先释放锁再获取锁



好处

* 避免死锁：==一个线程在拿到锁后，可以进入被这个锁，锁住的任何方法==。
* 提升封装性：避免了一次次的加锁和解锁
* 适合需要递归才能将数据处理完的场景



在Java中，synchronized 和 reentrantLock 都是可重入锁



**实现源码**

底层是 AQS，通过 state 变量来维护了加锁次数

![image-20210706181743006](Java%20Concurrent.assets/image-20210706181743006.png)







### 公平锁与非公平锁

在Java中，==ReentrantLock、ReentrantReadWriteLock 默认是非公平的==，可以设置参数变为公平

公平指的是按照线程请求的顺序，来分配锁；非公平指的是，不完全按照请求的顺序，在一定情况下，可以插队。

==注意==：非公平也同样不提倡“插队”行为，这里的非公平，指的是“==在合适的时机==”插队，而不是盲目插队。



**为什么要有非公平锁？**

* 进程被唤醒到执行，需要一定的时间

* 为了避免唤醒带来的空档期，==提高吞吐量==
* 允许在这个空档期内，让其他运行的线程使用下这把锁



**什么是合适的时机呢？**

线下买火车票“被插队”的例子。

排在前面的人买了一张票走了，该轮到我了，因为我排了很久的队伍头脑还是朦朦的，这个时候刚才离开是人又过来，向售票员询问火车的一些信息。



只会在“合适的时机”进行插队，即线程被唤醒的时间段里



**伪码描述**

* 假设有一把非公平锁
* 假设有10个线程，依次执行打印任务
* 打印任务，要打印两页内容，每一页都有加锁
* 线程0，先获取到锁，其他9个线程没有拿到锁，进入等待队列
* 线程0，打印完第一页后，释放锁
* 当锁是非公平锁时，线程0会优先再次拿到锁，因为线程从等待队列唤醒需要一定的时间





**真正的插队方法**

* 针对 ==tryLock()== 方法，它是很猛的，它不遵守设定的公平的规则

* 例如，当有线程执行tryLock()的时候，一旦有线程释放了锁，那么这个正在tryLock的线程就能获取到锁，即使在它之前已经有其他现在在等待队列里了



**优缺点对比**

|          | 优点                                                         | 缺点                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 公平锁   | 各线程公平平等，每个线程在等待一段时间后，总<br/>有执行的机会 | 更慢，吞吐量更小                                             |
| 非公平锁 | 锁更快，吞吐量更大                                           | 可能产生==线程饥饿（致命问题）==，也就是某些线程在长时间内，始终得不到执行 |





**源码分析**

唯一的差别，hasQueuedPredecessors() 

![image-20210706194033449](Java%20Concurrent.assets/image-20210706194033449.png)





### 共享锁和排它锁

排他锁，又称为独占锁、独享锁，如 synchronized、reentrantLock。 

共享锁，又称为==读锁==，获得共享锁之后，可以查看但无法修改和删除数据，其他线程此时也可以获取到共享锁，也可以查看但==无法修改和删除==数据

共享锁和排它锁的典型是读写锁 ==ReentrantReadWriteLock==，其中==读锁是共享锁，写锁是排它锁==



**读写锁的作用**

* 在没有读写锁之前，我们假设使用 ReentrantLock，那么虽然我们保证了线程安全，但是也==浪费了一定的资源：多个读操作同时进行，并没有线程安全问题==，ReentrantLock 是不区分读写的。

* 在读的地方使用读锁，在写的地方使用写锁，灵活控制，如果没有写锁的情况下，==读是无阻塞的（但是不能修改和删除！）==，提高了程序的执行效率



**读写锁规则**

* 要么是一个或多个线程同时有读锁，要么是一个线程有写锁，但是两者不会同时出现（==要么多读，要么一写==）

* 读写锁可以理解为只是一把锁



**入门伪码描述**

* 分别定义出读锁、写锁
* 分别编写读方法、写方法 —— 与 lock 写法类似，先获取锁，然后 finally 释放锁
* 用4个线程模拟，依次是 两读 两写
* 读写锁是阻塞的，没有拿到锁的线程会进入等待队列，锁释放后，依次执行





**读锁插队策略（公平与非公平策略）**

假设线程2和线程4正在同时读取，线程3想要写入，拿不到锁，于是进入等待队列，线程5不在队列里，现在过来想要读取

线程示意描述：

* 进行中 —— 线程2、线程4
* 等待队列 —— 线程3、线程5



策略1 - 非公平：

* 读可以插队，效率高

* 但容易造成饥饿，因为总有人持有读锁，获取写锁的线程始终得不到 —— 产生事故



策略2  - 公平 ：

* 线程2、4 执行完后，先执行线程3写，然后再是线程5读
* 避免饥饿，选择公平锁能避免发生事故













--------------------





|          | 描述                                                         | 实现原理                                          | 是否公平                                     |
| -------- | ------------------------------------------------------------ | ------------------------------------------------- | -------------------------------------------- |
| 自旋锁   | 线程自旋一定次数，如10次，若没有竞争到锁，<br />则会升级重量级锁。<br />但高并发场景下，线程可能都会执行完自旋次数，<br />性能浪费 | java 实现自旋，CAS 尝试                           | 公平                                         |
| 轻量级锁 | 线程先逐个插入EntrySet队列尾部，仅头部的线程自旋，竞争失败，则升级到重量级锁。 | java实现EntrySet                                  | 公平                                         |
| 重量级锁 |                                                              | 操作系统实现WaitSet，操作系统可见的休眠线程范围。 | 公平                                         |
| 偏向锁   | 看 Object的偏向锁位置是否存在owner线程，<br />如果竞争成功，这里会插对优先执行 | java实现的部分，更加轻量                          | 非公平：当前线程插队，避免进程切换，效率更高 |







## Synchronized

synchronized 是最早的同步实现，一切从这里开始

```java
synchronized(obj) {
    // 临界区
}
```

转换成伪码

```java
enterLock;
// 临界区
releaseLock;
```



synchronized 需要满足的能力

* 实现锁/解锁
* 实现自旋锁到休眠的升级逻辑
* API设计：每个对象都可以上锁
* 线程在竞争不到资源时休眠
* 释放资源时唤醒线程





### 线程休眠

访问临界区代码，在未获取到锁时，线程要进入休眠

| 休眠机制                                  | 描述             | 特点                                                         |
| ----------------------------------------- | ---------------- | ------------------------------------------------------------ |
| 休眠少量CPU周期（自旋锁）                 | 以慢速执行指令   | CPU 较少分配给该线程资源<br />CPU 能耗少，发热少，低温状态CPU性能更好 |
| 定时休眠（类似Thread.sleep）              | 休眠固定时间0    | CPU 停止分配线程资源；<br />时间所有浪费，用户体验不好       |
| ==信号休眠、信号唤醒（类似wait/notify）== | 等待其他线程唤醒 | 性能和体验都更好                                             |

> 休眠方案一

先尝试自旋（默认10次），再尝试信号休眠、唤醒



object.wait

object.notify





### 线程唤醒



**JVM 知道哪些线程在哪些对象上休眠？**



### 对象锁







### 底层实现

由一对指令括起来的代码 monitorenter， monitorexit 





### 双重检查锁（DCL）

```java
if() {
    synchronized(this) {
        if() return ;
        ...               
    }
}
```

外层的if判断可以避免加锁竞争效率提高；

内层if判断可能线程已经执行完毕；



> 应用场景





## Lock接口

Lock接口最常见的实现类是ReentrantLock



通常情况下，Lock只允许一个线程来访问这个共享资源。不过有的时候，

一些特殊的实现也可允许==并发访问==，比如ReadWriteLock里面的==ReadLock==。



**可见性**

与 synchronized 一样，Lock 也具有可见性保障（happens-before 特性）



### 锁方法

**为什么需要Lock锁？**

*等价于“为什么synchronized不够用？”*

1. 效率低：
    * 锁的释放情况少：释放条件为 1）执行完毕； 2）发生异常。业务中可能需要线程在等待IO时释放锁的情况。
    * 试图获得锁时不能设定超时
    * 不能中断一个正在试图获得锁的线程
2. 不够灵活（读写锁更灵活）：加锁和释放的时机单一，每个锁仅有单一的条件（某个对象），可能是不够的
3. 无法知道是否成功获取到锁



对应 Lock 具有一些方法

1. ==4个方法获取锁：==
    1. lock()、
    2. tryLock()、
    3. tryLock(long time，TimeUnit unit）
    4. lockInterruptibly()

2.  try ... finally {  lock.unlock();  }  // ==一定要释放锁==



lock() 方法

* 是最普通的获取锁。如果锁已被其他线程获取，则进行等待
* Lock不会像synchronized一样在异常时自动释放锁
* 因此最佳实践是，==必须在finally中释放锁==，以保证发生异常时锁一定被释放

* lock方法不能被中断，这会带来很大的隐患：一旦==陷入死锁==，lock 就会陷入永久等待 —— ==引入 tryLock==



> 死锁：

* 至少有两把锁和两个线程，线程1已经占有了锁1，试图抢占锁2；此时线程2已经占有了锁2，试图抢占锁1
* 两个线程对要抢占的锁，都不释放。



tryLock() 

*避免死锁* 

* tryLock() 用来==尝试获取锁==，如果当前锁没有被其他线程占用，则获取成功，则返回true，否则返回false，代表获取锁失败
* 相比于lock，这样的方法显然功能更强大了，我们可以根据是否能获取到锁来==决定后续程序的行为==
* 该方法会==立即返回==，即便在拿不到锁时不会一直在那等



tryLock(long time，TimeUnit unit）



lockInterruptibly()

* 相当于tryLock（long time，TimeUnit unit）把超时时间设置为无限。

* 在等待锁的过程中，线程可以被中断 —— thread0.interrupt 中断线程





### ReentrantLock

*这是最具代表 Lock 锁的实现类*

可重入锁



**可重入伪码**

lock.getHoldCount() 可以查看这把锁的有效加锁次数

lock.lock(); // 获取锁，holdCount + 1

lock.lock(); // 同一线程，获取同一把锁，无须先解锁，holdCount + 1

lock.unlock(); // 释放锁，holdCount - 1

lock.unlock(); // 锁是可重入的，但锁用完要释放锁，供其他线程使用，holdCount - 1



**常用方法**

isHeldByCurrentThread可以看出锁是否被当前线程持有

getQueueLength可以返回当前正在等待这把锁的队列有多长，一般这两个方法是开发和调试时候使用，上线后用到的不多







可重入锁

可重入锁的定义：当线程请求一个由其他线程持有的==对象锁==时，该线程会阻塞，而当线程请求由自己持有的对象锁是，如果该锁是可重入的，请求就会成功，否则阻塞。

|          | Synchronized | ReentrantLock        |
| -------- | ------------ | -------------------- |
| 锁类型   | 非公平锁     | 默认非公平锁，可配置 |
| 实现方式 | JVM实现      | JDK 实现             |
|          |              |                      |



**验证 ReentrantLock 是可重入锁**

```java
public class ReentrantTest01 {
    private final Lock lock = new ReentrantLock();
    int value = 0;

    public static void main(String[] args) throws InterruptedException {
        ReentrantTest01 test = new ReentrantTest01();
        Thread t1 = new Thread(()->{
            test.add();
        });
        t1.start();

        // 等待t1线程执行完毕
        t1.join();

        System.out.println(test.value);
    }

    public void add() {
        lock.lock();
        try {
            value = 1 + get();
        } finally {
            lock.unlock();
        }
    }

    public int get() {
        lock.lock();
        try {
            return value;
        } finally {
            lock.unlock();
        }
    }
}
```



**验证 Synchronized 是可重入锁**

```java
public class ReentrantTest02 {
    private final Object object = new Object();
    int value = 0;

    public static void main(String[] args) throws InterruptedException {
        ReentrantTest02 test = new ReentrantTest02();
        Thread t1 = new Thread(()->{
            test.add();
        });
        t1.start();
        // 等待t1线程执行完毕
        t1.join();
        System.out.println(test.value);
    }

    public void add() {
        synchronized(object) {
            value = 1 + get();
        }
    }

    public int get() {
        synchronized(object) {
            return value;
        }
    }
}
```

**实现自定义不可重入锁**

```java
public class CustomLock {
    private boolean isLock = false;

    public synchronized void lock() throws InterruptedException {
        while (isLock) {
            wait();
        }
        isLock = true;
    }

    public synchronized void unlock() {
        isLock = false;
        notify();
    }
}
```

**实现自定义可重入锁**

```java
public class CustomReentrantLock {
    private boolean isLock = false;
    private Thread lockThread = null;
    private int lockedCount = 0;
    public synchronized void lock() throws InterruptedException {
        //System.out.println("isLock=" + isLock + " lockThreadId=" + lockThreadId + " currentId=" +currentId  );
        while (isLock && !Objects.equals(lockThread, Thread.currentThread())) {
            wait();
        }
        isLock = true;
        lockThread = Thread.currentThread();
        lockedCount++;
    }

    public synchronized void unlock() {
        if (isLock && Objects.equals(lockThread, Thread.currentThread())) {
            lockedCount--;
            if (lockedCount == 0) {
                isLock = false;
                lockThread = null;
                notify();
            }
        }
    }
}
```



### ReenterantReadWriteLock



















> Synchronized 和 Lock 区别

1. Synchronized 内置的Java关键字， Lock 是一个Java类
2. Synchronized 无法判断获取锁的状态，Lock 可以判断是否获取到了锁（TODO 应用场景）
3. Synchronized 会自动释放锁，lock 必须要手动释放锁！如果不释放锁，死锁（类似开车中的自动挡和手动挡，问开车比赛中为什么都是手动挡，车手要尽可能把赛车的性能发挥到极致，这就需要对车有更精准的控制）
4. Synchronized 线程 1（获得锁，阻塞，获得锁可能又阻塞了，TODO 场景）、线程2（等待，傻傻的等，暗含效率低）；Lock锁就不一定会等待下去（lock. tryLock 不会傻傻等， TODO tryLock 时机）；
5. Synchronized 可重入锁，不可以中断的，非公平；Lock ，可重入锁，可以 判断锁，非公平（可以
    自己设置）；
6. Synchronized 适合锁少量的代码同步问题（TODO 最佳实践吗？），Lock 适合锁大量的同步代码！



> 锁是什么，如何判断锁的是谁！ 

synchronized 若修饰的是方法，那么锁的是该类的实例对象，若修饰的是 static 方法，那么锁就是 Class 对象（模板）







## CAS



### ABA

参考：https://developer.aliyun.com/article/744324?spm=a2c6h.12873639.0.0.1bf75749dEe9BJ



ABA 描述（略）



### volatile 关键字



ABA 在一些特殊场景下，会存在问题，举一个堆栈操作的例子：
![image.png](Java%20Concurrent.assets/e1410ebfcff24e5fa07d221c9ccbdd86.png)

并发1（上）：读取栈顶的元素为“A1”
![image.png](Java%20Concurrent.assets/4ac7953bf7ce49d0b311eec183112c64.png)

并发2：进行了2次出栈
![image.png](Java%20Concurrent.assets/753620f216844ddab8644977c690ba02.png)

并发3：又进行了1次出栈
![image.png](Java%20Concurrent.assets/6e4f196a51dc4a70ab32afb95f5726aa.png)

并发1（下）：实施CAS乐观锁，发现栈顶还是“A1”，于是修改为A2
![image.png](Java%20Concurrent.assets/6522f715f0fa46bdb0b437b8c9dee828.png)

此时会出现系统错误，因为此“A1”非彼“A1” 



再举个例子，有一个链表 head->A->B—>C，线程1要把A都移除掉，线程2发现有B就再B前加一个A。线程1、2 就会频繁加减元素。



**ABA问题可以怎么优化？**

* AtomicMarkableReference（boolean表示版本）
* AtomicStampedReference（int表示版本）

实现原理是增加版本，让 CAS 从整体看，除了看值本身，还看版本是否正确！



## Atmoic类

底层是 CAS 原子指令实现























## 工具对比

|      | Synchronized | Lock | CAS  | Atmoic |
| ---- | ------------ | ---- | ---- | ------ |
| 性能 | 低           | 中   | 高   | 高     |
|      |              |      |      |        |
|      |              |      |      |        |







# AQS

同步器开发框架

**AQS**：AbstractQueuedSynchronizer



用来给程序员自定义同步器



<img src="Java%20Concurrent.assets/image-20210630181020001.png" alt="image-20210630181020001" style="zoom: 67%;" />	





| 休眠+唤醒                      | 是否加锁                 | 备注                   |
| ------------------------------ | ------------------------ | ---------------------- |
| Object.wait/notify             | 需要加锁，因为非原子操作 | 需要锁，否则会竞争条件 |
| Lock.newCondition.await/signal | 需要加锁，因为非原子操作 | 需要锁，否则会竞争条件 |
| LockSupport.park/unpark        | 无须加锁，因为底层方法   |                        |



synchronized的缺点

* 不能让用户实现更底层的数据结构



AQS，Java中的抽象类都是为了简化开发，防止程序犯一些低级的错误。



AQS 提供的能力

* tryLock  - 超时
* 接收线程中断
* tryAcquire 非阻塞版获取锁
* release 释放锁
* acquire 获取锁
* condition 条件变量（操作系统提供，基于整数的生产者/消费者模型）
* int state > CAS（替代 unsafe 的 cas）
* 队列











----------------













# JUC

java.util.concurrent







## 不安全集合





> ArrayLIst 是不安全的

```java
public static void main(String[] args) {
    List<String> list = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        new Thread(()->{
            list.add(UUID.randomUUID().toString().substring(0, 5));
            System.out.println(list);
        }, String.valueOf(i)).start();
    }
}
```

![image-20210603193033103](Java.assets/image-20210603193033103.png)

java.util.ConcurrentModificationException 并发修改异常



解决方案：

* java.util.concurrent.CopyOnWriteArrayList (JUC 方案)
* Collections.synchronizedList(new ArrayList<>())
* new Vector<>() —— JDK1.0就有了，源码实现是用 synchronized 关键字



## CopyOnWrite

读写分离的思想



> 为什么 copyOnWriteArrayList 要比 vector 好

* 前者的源码是通过 lock/unlock 和 复制来实现的，后者是  synchronized，前者的性能更好。（TODO 为什么lock就是好）





> Map 也不安全





> 为什么说工作中不用HashMap











> ConcurrentHashMap

TODO 看源码 jdk 文档





> 创建多线程的几种方式

* Thread
* Runnable
* Callable



## Callable

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    FutureTask futureTask = new FutureTask<>(()->{
        return "everything";
    });

    Thread thread = new Thread(futureTask);
    thread.start();

    System.out.println(futureTask.get());
}

/** 执行结果：
everything
*/
```



与 Runnable 相比

* 有返回值，是泛型
* **可以抛出异常**
* 方法不同，runnable 是 run， callable 是 call

```java
new Thread(new Runnable()).start(); // Thread 构造参数只能接收 Runnable 
new Thread(new FutureTask<V>()).start(); // FutureTask 是Runnable 的实现类
new Thread(new FutureTask<V>( Callable )).start(); // Callable 无法直接关联 Thread，需要适配类 - FutureTask

// 获取返回值
FutureTask futureTask = new MyThreadByCallable(); // 拿到引用
new Thread(futureTask, "A").start();
futureTask.get(); // 这个 get 方法可能会产生阻塞，把它放到最后，或者使用异步通信来处理
```

==注意==

Callable 的返回值

* 可能需要等待，会阻塞
* 有缓存，多次调用 call 方法，只执行一次



> 应用场景



## FutureTask

JDK 1.5

特点：

* 有返回值，但是是阻塞的



## CompletableFuture

JDK 1.8

对 FutureTask 的改进

* 有返回值，且是异步的（不阻塞）

> 应用场景



## 停止线程

先看线程执行的任务类型



> 计算型 cpu 密集型

没有 wait、await，没有等待，没有阻塞操作。

线程执行代码中增加一个标志位（volatile），线程定时判断标志位，是要继续、中断、还是回滚。

思路：通过标志位 + DCL 来结束。







>IO密集型

有等待，有阻塞操作

例子：从网卡读取数据，线程处理 wait 中，程序无法在线程wait中增加标志位代码来检查。

思路：interrupt





---------------





取消线程的方案：

* 做好同步
* 若要支持多次取消，做好幂等（无状态）



> 回滚线程





## 常用辅助类

### CountDownLatch 

减法计数器

原理：

countDownLatch.countDown(); // 数量-1

countDownLatch.await(); // 等待计数器归零，然后再向下执行

所以组合起来使用就是，每次有线程调用 countDown() 数量-1，假设计数器变为0，countDownLatch.await() 就会被唤醒，继续执行！  







### CyclicBarrier

加法计数器



## Semaphore

信号量：限流，保持有序

例子：抢车位，6辆车抢3个停车位。



**原理：**
semaphore.acquire() 获得，假设如果已经满了，等待，等待被释放为止！  

semaphore.release(); 释放，会将当前的信号量释放 + 1，然后唤醒等待的线程！

+1 -1 非常像生产者消费者模型；也隐含了线程间的通信。

**作用**： 多个共享资源互斥的使用！并发限流，控制最大的线程数！  





# 综合应用



## 高可用的一些手段

任务调度系统分布式：elastic-job + zookeeper



主备切换：apache curator+zookeeper分布式锁实现



监控报警机制





## 高并发下如何安全修改同一行数据















## 生产者/消费者

这个模型非常普遍，如发红包抢红包



抽象为  读/计算/写 模型





线程交替执行

线程之间的通信问题，必然提到生产者和消费者问题

例子：

A B 操作同一个变量 num = 0 ，先是 A num +1，再是 B num -1。要保证有序交替。

多线程开发中涉及到共享资源的，必须加锁

> 实现一：Synchronized 版（老版），使用 wait（等待唤醒） 和 notify（通知唤醒） 实现，这三个是一组的。

wait、notify 是 Object 类，所以每个Java实例都有

```java
/**
 * 线程交替
 * 线程 A: num +1
 * 线程 B: num -1
 */
public class ConsumeAndProduceTest {
    public static void main(String[] args) {
        Num num = new Num();
        new Thread(()->{
            for (int i = 0; i < 20; i++) {
                try {
                    num.incr();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();

        new Thread(()->{
            for (int i = 0; i < 20; i++) {
                try {
                    num.decr();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();
    }
}

class Num {
    private int val = 0;

    public synchronized void incr() throws InterruptedException {
        if (val != 0) {
            this.wait();
        }
        this.val++;
        System.out.println(Thread.currentThread().getName() + "=>" + this.val);
        this.notify();
    }

    public synchronized void decr() throws InterruptedException {
        if (val == 0) {
            this.wait();
        }
        this.val--;
        System.out.println(Thread.currentThread().getName() + "=>" + this.val);
        this.notify();
    }
}
```

> 当由两个线程，变为4个线程时，上面的代码会出现问题，如何解决？

问题原因

* notify 没有分组，可能同是 +1 的线程先拿到锁
* 线程业务操作时，没有再次判断条件

上面就是**虚假唤醒**

用 while 替代 if



> JUC 版（新版）生产者消费者

新老版有个对应关系

![image-20210603120135357](Java.assets/image-20210603120135357.png)



```java
// 数字 资源类
public class JUCData {
    private int number = 0;

    Lock lock = new ReentrantLock();
    Condition condition = lock.newCondition();
    public void increment() throws InterruptedException {
        lock.lock();
        try {
            while (number != 0) {
                condition.await();
            }
            number++;
            System.out.println(Thread.currentThread().getName() + "=>" + number);
            condition.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void decrement() throws InterruptedException {
        lock.lock();
        try {
            while (number == 0) {
                condition.await();
            }
            number--;
            System.out.println(Thread.currentThread().getName() + "=>" + number);
            condition.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

实现了 synchronized 版本，一样的效果，但诞生新技术的意义是？

> Condition

Condition 的诞生，解决了线程的有序问题

Condition 精准的通知和唤醒线程



![image-20210603162857703](Java.assets/image-20210603162857703.png)

报错原因是忘记加锁了，没有加锁，直接调用 Condition 或 lock.unlock

```java
public class JUC_02_Order {
    public static void main(String[] args) {
        JUCData02 data = new JUCData02();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data.printA();
            }
        }, "A").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data.printB();
            }
        }, "B").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data.printC();
            }
        }, "C").start();
    }
}


// 数字 资源类
class JUCData02 {
    private Lock lock = new ReentrantLock();
    private Condition condition1 = lock.newCondition();
    private Condition condition2 = lock.newCondition();
    private Condition condition3 = lock.newCondition();

    private int number = 1; // 1A 2B 3C

    public void printA() {
        lock.lock();
        try {
            while (number % 3 != 1) {
                condition1.await();
            }
            System.out.println(Thread.currentThread().getName() + "=>" + number);
            number++;
            condition2.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    public void printB() {
        lock.lock();
        try {
            while (number % 3 != 2) {
                condition2.await();
            }
            System.out.println(Thread.currentThread().getName() + "=>" + number);
            number++;
            condition3.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    public void printC() {
        lock.lock();
        try {
            while (number % 3 != 0) {
                condition3.await();
            }
            System.out.println(Thread.currentThread().getName() + "=>" + number);
            number++;
            condition1.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```







## 实现分布式事务

扼要技术点

* volatile —— 线程可见性
* 双重检查锁
* completableFuture —— 异步且返回线程的执行结果

```
有这样的一个任务T
T由N个子任务构成，每个子任务完成的时长不同
若其中有一个子任务失败
所有任务结束，T 任务失败
请写程序模拟这个过程

要求Fail Fast，结束越快越好
```

```java
public class Test01_DistributedTransaction {
    public static void main(String[] args) throws ExecutionException, InterruptedException, IOException {
        MyTask task1 = new MyTask("task1", 5, true);
        MyTask task2 = new MyTask("task2", 1, false);
        MyTask task3 = new MyTask("task3", 3, true);

        // 异步获取
 CompletableFuture.supplyAsync(task1::call).thenAccept(Test01_DistributedTransaction::callback);      CompletableFuture.supplyAsync(task2::call).thenAccept(Test01_DistributedTransaction::callback);
 CompletableFuture.supplyAsync(task3::call).thenAccept(Test01_DistributedTransaction::callback);
        // 让线程执有时间执行完
        System.in.read();
    }

    private static void callback(Boolean result) {
        if (false == result) {
            System.out.println("错误发生");
        }
    }

    private static class MyTask {
        private String name;
        private int seconds;
        private boolean ret;

        public MyTask(String name, int seconds, boolean ret) {
            this.name = name;
            this.seconds = seconds;
            this.ret = ret;
        }

        public Boolean call() {
            try {
                TimeUnit.SECONDS.sleep(seconds);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(name + " task callback");
            return ret;
        }
    }
}
```





