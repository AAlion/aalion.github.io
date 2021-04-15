3---
layout: post
title: 'AQS'
categories: Java
date: 2021-01-01 00:00:03
tags:
    - Java
    - 源码
---

@[TOC](目录)

#### AQS(AbstractQueuedSynchronizer) ★
* [AQS_zejian](https://blog.csdn.net/javazejian/article/details/75043422)|
* unlock()操作必须在finally代码块中确保即使临界区执行抛出异常，线程最终也能正常释放锁

##### 可重入锁
* Lock为接口，ReentrantLock是Lock的实现类
* 又名递归锁，ReentrantLock/Synchronized就是一个典型的可重入锁。
* 最大作用：避免死锁
* 可重入锁概念：
-- ReentrantLock翻译叫可重入锁。所谓可重入锁，顾名思义，指的是线程可以重复获取同一把锁。
-- 同一个线程外层函数获得锁之后，内层递归函数仍然能够获取该锁的代码，在同一个线程在外层方法获取锁的时候，在进入内层方法会自动获取锁
-- 如下代码，当线程 T1 执行到 ①处时，已经获取到了锁 rtl ，当在 ① 处调用get() 方法时，会在 ② 再次对锁 rtl执行加锁操作。
此时，如果锁 rtl 是可重入的，那么线程T1可以再次加锁成功；如果锁 rtl 是不可重入的，那么线程 T1 此时会被阻塞。
    ```java
    class X {
    	 private final Lock rtl = new ReentrantLock();
    	 int value;
    	 public int get() {
    		 // 获取锁
    		 rtl.lock(); ②
    		 try {
    			return value;
    		 } finally {
    			 // 保证锁能释放
    			 rtl.unlock();
    		 }
    	 }
    	 public void addOne() {
    		 // 获取锁
    		 rtl.lock();
    		 try {
    			value = 1 + get(); ①
    		 } finally {
    			 // 保证锁能释放
    			 rtl.unlock();
    		 }
    	 }
    }
    
    ```
* 可重入函数
指的是多个线程可以同时调用该函数，每个线程都能得到正确结果；同时在一个线程内支持线程切换，无论被切换多少次，结果都是正确的。多线程可以同时执行，还支持线程切换，这意味着什么呢？线程安全啊。所以，可重入函数是线程安全的。

##### AQS
* [RL_zejian](https://blog.csdn.net/javazejian/article/details/75043422)|
* **AQS原理及实现?** 
+ 类在java.util.concurrent.locks包下面
+ 概念
AQS是一个用来***构建锁和同步器的框架***，使用AQS能简单且高效地构造出应用广泛的大量的同步器，比如我们提到的ReentrantLock，Semaphore，其他的诸如ReentrantReadWriteLock，SynchronousQueue，FutureTask等等皆是基于AQS的。当然，我们自己也能利用AQS非常轻松容易地构造出符合我们自己需求的同步器。
+ AQS核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用CLH队列锁实现的，即将暂时获取不到锁的线程加入到队列中。
![AQS原理图](https://img-blog.csdnimg.cn/20200716013846411.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
+ AQS 对资源的共享方式
AQS定义两种资源共享方式
① Exclusive（独占）：只有一个线程能执行，如ReentrantLock。又可分为公平锁和非公平锁：
*公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
*非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的
② Share（共享）：多个线程可同时执行，如Semaphore/CountDownLatch。Semaphore、CountDownLatch、 CyclicBarrier、ReadWriteLock 我们都会在后面讲到。
ReentrantReadWriteLock 可以看成是组合式，因为ReentrantReadWriteLock也就是读写锁允许多个线程同时对某一资源进行读。
不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源 state 的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在顶层实现好了。
+ AQS底层使用了模板方法模式
同步器的设计是基于模板方法模式的，如果需要自定义同步器一般的方式是这样（模板方法模式很经典的一个应用）：
使用者继承AbstractQueuedSynchronizer并重写指定的方法。（这些重写方法很简单，无非是对于共享资源state的获取和释放）
将AQS组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。
这和我们以往通过实现接口的方式有很大区别，这是模板方法模式很经典的一个运用。
AQS使用了模板方法模式，自定义同步器时需要重写下面几个AQS提供的模板方法：
 ```java
isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。
tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。

 ```
默认情况下，每个方法都抛出 UnsupportedOperationException。 这些方法的实现必须是内部线程安全的，并且通常应该简短而不是阻塞。AQS类中的其他方法都是final ，所以无法被其他类使用，只有这几个方法可以被其他类使用。
 *以ReentrantLock为例，state初始化为0，表示未锁定状态。A线程lock()时，会调用tryAcquire()独占该锁并将state+1。此后，其他线程再tryAcquire()时就会失败，直到A线程unlock()到state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A线程自己是可以重复获取此锁的（state会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证state是能回到零态的。
*再以CountDownLatch以例，任务分为N个子线程去执行，state也初始化为N（注意N要与线程个数一致）。这N个子线程是并行执行的，每个子线程执行完后countDown()一次，state会CAS(Compare and Swap)减1。等到所有子线程都执行完后(即state=0)，会unpark()主调用线程，然后主调用线程就会从await()函数返回，继续后余动作。
*一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现tryAcquire-tryRelease、tryAcquireShared-tryReleaseShared中的一种即可。但AQS也支持自定义同步器同时实现独占和共享两种方式，如ReentrantReadWriteLock。
* [⭐原理图示](https://blog.csdn.net/wind_602/article/details/104161960)

* -----------------------------具体-------------------
AQS的原理概要，如下[源码](https://blog.csdn.net/javazejian/article/details/75043422)
* **1 AQS工作原理概要**
--概念：AbstractQueuedSynchronizer(AQS)又称为队列同步器；
--作用：用来构建锁或其他同步组件的基础框架；
--state：内部通过一个int类型的成员变量state来控制同步状态：
① 当state=0，则说明没有任何线程占有共享资源的锁；
② 当state=1，则说明有线程目前正在使用共享变量，其他线程必须加入同步队列进行等待；
--同步队列：AQS内部通过内部类Node构成FIFO的同步队列来完成线程获取锁的排队工作；
--等待队列：AQS同时利用内部类ConditionObject构建等待队列，当Condition调用await()方法后，线程将会加入等待队列中，而当Condition调用signal()方法后，线程将从等待队列转移动同步队列中进行锁竞争。
注意：这里涉及到两种队列，一种的同步队列，当线程请求锁而等待后将加入同步队列等待，而另一种则是等待队列(可有多个)，通过Condition调用await()方法释放锁后，将加入等待队列。
* **2 AQS中的同步队列模型**
**1）AQS**
**--head和tail：**分别是AQS中的变量。
head：指向同步队列的头部，注意head为空结点，不存储信息。
tail：指向同步队列的队尾，同步队列采用的是双向链表的结构这样可方便队列进行结点增删操作。
**--state：** state变量则是代表同步状态。
state=0：执行当线程调用lock方法进行加锁后，如果此时state的值为0，则说明当前线程可以获取到锁(在本篇文章中，锁和同步状态代表同一个意思)，同时将state设置为1，表示获取成功。
state=1：如果state已为1，也就是当前锁已被其他线程持有，那么当前执行线程将被封装为Node结点加入同步队列等待。
**--Node结点：**是对每一个访问同步代码的线程的封装。
    ```
    
    /** AQS抽象类*/
    public abstract class AbstractQueuedSynchronizer
        extends AbstractOwnableSynchronizer{
    //指向同步队列队头
    private transient volatile Node head;
    //指向同步的队尾
    private transient volatile Node tail;
    //同步状态，0代表锁未被占用，1代表锁已被占用
    private volatile int state;
    //省略其他代码......
    }
    ```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401134928488.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
**2）Node节点**
从图中的Node的数据结构也可看出，其**包含了需要同步的线程本身以及线程的状态**，如是否被阻塞，是否等待唤醒，是否已经被取消等。每个Node结点内部关联其前继结点prev和后继结点next，这样可以方便线程释放锁后快速唤醒下一个在等待的线程，Node是AQS的内部类，其数据结构如下：
**-- SHARED(shared)和EXCLUSIVE(exclusive)常量：**分别代表共享模式和独占模式。
① 共享模式：是一个锁允许多条线程同时操作；
如信号量Semaphore采用的就是基于AQS的共享模式实现的。
② 独占模式：是同一个时间段只能有一个线程对共享资源进行操作，多余的请求线程需要排队等待；
如ReentranLock。
**--waitStatus变量：**表示当前被封装成Node结点的等待状态。
共4种：
① CANCELLED：值为1，在同步队列中等待的线程等待超时或被中断，需要从同步队列中取消该Node的结点，其结点的waitStatus为CANCELLED，即结束状态，进入该状态后的结点将不会再变化。
② SIGNAL：值为-1，被标识为该等待唤醒状态的后继结点，当其前继结点的线程释放了同步锁或被取消，将会通知该后继结点的线程执行。说白了，就是处于唤醒状态，只要前继结点释放锁，就会通知标识为SIGNAL状态的后继结点的线程执行。
③ CONDITION：值为-2，与Condition相关，该标识的结点处于等待队列中，结点的线程等待在Condition上，当其他线程调用了Condition的signal()方法后，CONDITION状态的结点将从等待队列转移到同步队列中，等待获取同步锁。
④ PROPAGATE：值为-3，与共享模式相关，在共享模式中，该状态标识结点的线程处于可运行状态。
⑤ 0状态：值为0，代表初始化状态。
**--pre和next：**分别指向当前Node结点的前驱结点和后继结点；
**--thread变量：**存储的请求锁的线程。
**--nextWaiter：**与Condition相关，代表等待队列中的后继结点，后续会有更详细的分析。
    ```
    
    static final class Node {
        static final Node SHARED = new Node();   //共享模式
        static final Node EXCLUSIVE = null;   //独占模式
        static final int CANCELLED =  1;   //标识线程已处于结束状态
        static final int SIGNAL    = -1;    //等待被唤醒状态
        static final int CONDITION = -2;    //条件状态，
        static final int PROPAGATE = -3; //在共享模式中使用表示获得的同步状态会被传播
        volatile int waitStatus; //等待状态,存在CANCELLED、SIGNAL、
                                           //CONDITION、PROPAGATE 4种
        volatile Node prev;   //同步队列中前驱结点
        volatile Node next;   //同步队列中后继结点
        volatile Thread thread;    //请求锁的线程
        Node nextWaiter;   //等待队列中的后继结点，这个与Condition有关
        final boolean isShared() {   //判断是否为共享模式
            return nextWaiter == SHARED;
        }
        final Node predecessor() throws NullPointerException {  //获取前驱结点
            Node p = prev;
            if (p == null)
                throw new NullPointerException();
            else
                return p;
        }
        //.....
    }
    ```
**3）总结**
总之呢，AQS作为基础组件，对于锁的实现存在两种不同的模式，即共享模式(如Semaphore)和独占模式(如ReetrantLock)，无论是共享模式还是独占模式的实现类，其内部都是基于AQS实现的，也都维持着一个虚拟的同步队列，当请求锁的线程超过现有模式的限制时，会将线程包装成Node结点并将线程当前必要的信息存储到node结点中，然后加入同步队列等会获取锁，而这系列操作都有AQS协助我们完成，这也是作为基础组件的原因，无论是Semaphore还是ReetrantLock，其内部绝大多数方法都是间接调用AQS完成的。
下面是AQS整体类图结构：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021040114195319.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
**4）ReentrantLock与AQS的关系**
**1> ReentrantLock类和继承：**
--AbstractOwnableSynchronizer：抽象类，定义了存储独占当前锁的线程和获取的方法
--AbstractQueuedSynchronizer：抽象类，AQS框架核心类，其内部以虚拟队列的方式管理线程的锁获取与锁释放，其中获取锁(tryAcquire方法)和释放锁(tryRelease方法)并没有提供默认实现，需要子类重写这两个方法实现具体逻辑，目的是使开发人员可以自由定义获取锁以及释放锁的方式。
--Node：AbstractQueuedSynchronizer 的内部类，用于构建虚拟队列(链表双向链表)，管理需要获取锁的线程。
--Sync：抽象类，是ReentrantLock的内部类，**继承自AbstractQueuedSynchronizer**，实现了释放锁的操作(tryRelease()方法)，并提供了lock抽象方法，由其子类实现。
--NonfairSync：是ReentrantLock的内部类，继承自Sync，非公平锁的实现类。
--FairSync：是ReentrantLock的内部类，继承自Sync，公平锁的实现类。
--ReentrantLock：实现了Lock接口的，其内部类有Sync、NonfairSync、FairSync，在创建时可以根据fair参数决定创建NonfairSync(默认非公平锁)还是FairSync。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401142117583.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
**2> ReentrantLock内部类：**
--ReentrantLock内部存在3个实现类，分别是Sync、NonfairSync、FairSync。
--ReentrantLock的所有方法调用都通过间接调用AQS和Sync类及其子类来完成的。
--Sync类：继承自AQS实现了解锁tryRelease()方法；
--NonfairSync(非公平锁)、 FairSync(公平锁)则继承自Sync，实现了获取锁的tryAcquire()方法；
**3> AQS**
--AQS提供功能：
AQS是一个抽象类，但其源码中并没一个抽象的方法，这是因为AQS只是作为一个基础组件，并不希望直接作为直接操作类对外输出，而更倾向于作为基础组件，为真正的实现类提供基础设施，如构建同步队列，控制同步状态等，事实上，从设计模式角度来看，AQS采用的模板模式的方式构建的，其内部除了提供并发操作核心方法以及同步队列操作外，还提供了一些模板方法让子类自己实现，如加锁操作以及解锁操作，为什么这么做？
--为什么？设计理念：
这是因为AQS作为基础组件，封装的是核心并发操作，但是实现上分为两种模式，即共享模式与独占模式，而这两种模式的加锁与解锁实现方式是不一样的，但AQS只关注内部公共方法实现并不关心外部不同模式的实现，所以提供了模板方法给子类使用，也就是说实现独占锁，
如ReentrantLock需要自己实现tryAcquire()方法和tryRelease()方法，而实现共享模式的Semaphore，则需要实现tryAcquireShared()方法和tryReleaseShared()方法，
--好处：无论是共享模式还是独占模式，其基础的实现都是同一套组件(AQS)，只不过是加锁解锁的逻辑不同罢了，更重要的是如果我们需要自定义锁的话，也变得非常简单，只需要选择不同的模式实现不同的加锁和解锁的模板方法即可，AQS提供给独占模式和共享模式的模板方法如下
    ```
    //AQS中提供的主要模板方法，由子类实现。
    public abstract class AbstractQueuedSynchronizer
        extends AbstractOwnableSynchronizer{
        protected boolean tryAcquire(int arg) {     //独占模式下获取锁的方法
            throw new UnsupportedOperationException();
        }
        protected boolean tryRelease(int arg) {    //独占模式下解锁的方法
            throw new UnsupportedOperationException();
        }
        protected int tryAcquireShared(int arg) {   //共享模式下获取锁的方法
            throw new UnsupportedOperationException();
        }
        protected boolean tryReleaseShared(int arg) {   //共享模式下解锁的方法
            throw new UnsupportedOperationException();
        }
        protected boolean isHeldExclusively() {   //判断是否为持有独占锁
            throw new UnsupportedOperationException();
        }
    }
    ```



##### ReentrantLock-公平锁|非公平锁
* ReetrantLock，实现Lock接口，与synchronized作用相当，比其更灵活
* ReetrantLock是基于AQS并发框架实现

1. **公平锁、非公平锁**（sxt2）
* **是什么**
* 公平锁：是指多个线程按照***申请锁的顺序***来获取锁，满足FIFO。
* 非公平：是指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比现申请的线程优先获得锁，在高并发的情况下，有可能会造成优先级反战或者饥饿现象
* **区别**
* 公平锁：就是很公平，在并发环境中，每个线程获取锁时会查看此锁维护的等待队列，如果为空，或者当前线程是等待队列的第一个，就占有锁，否则加入等待队列，以后会按照FIFO的规则从队列中取到自己
* 非公平锁：比较粗鲁，上来就尝试占有锁，如果尝试失败，在采用类似公平锁的方式（非公平锁的优点在于吞吐量比公平锁大）
* **其他**
 Syschronized而言，也是非公平锁（类似lock）
* **ReentrantLock实现公平和非公平锁**
 ```java
    //方法1:无参构造函数：默认非公平锁
     public ReentrantLock() { 
        sync = new NonfairSync(); // 非公平锁
    }
    // 方法2：true时为公平锁，false时为非公平锁
    public ReentrantLock(boolean fair) { 
    sync = fair ? new FairSync() : new NonfairSync();
}
     
 ```
 * ReentrantLock的创建可以制定构造函数的boolean类型来得到公平锁或非
* 在入口等待队列，锁都对应着一个等待队列，如果一个线程没有获得锁，就会进入等待队列，当有线程释放锁的时候，就需要从等待队列中唤醒一个等待的线程。
-- 如果是公平锁，唤醒的策略就是谁等待的时间长，就唤醒谁，很公平；
-- 如果是非公平锁，则不提供这个公平保证，有可能等待时间短的线程反而先被唤醒。

* ----------------------------具体-------------------------------
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401220852189.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
* AQS的实现过程：基于ReetrantLock进一步分析AQS独占模式实现过程，这也是ReetrantLock的内部实现原理。
* **1 ReetrantLock中非公平锁-lock**
--AQS实现：
AQS同步器的实现依赖于内部的同步队列(FIFO的双向链表对列)完成对同步状态(state)的管理，当前线程获取锁(同步状态)失败时，AQS会将该线程以及相关等待信息包装成一个节点(Node)并将其加入同步队列，同时会阻塞当前线程，当同步状态释放时，会将头结点head中的线程唤醒，让其尝试获取同步状态。
--这里重点分析一下**获取同步状态和释放同步状态以及如何加入队列的具体操作**，这里从ReetrantLock入手分析AQS的具体实现，先以非公平锁为例进行分析。
--非公平锁
    ```
    
    public ReentrantLock() {  //默认构造，创建非公平锁NonfairSync
        sync = new NonfairSync();
    }
    public ReentrantLock(boolean fair) {  //根据传入参数创建锁类型
        sync = fair ? new FairSync() : new NonfairSync();
    }
    public void lock() {  //加锁操作 √
         sync.lock();
    }
    ```
--sync是个抽象类：
存在两个不同的实现子类，从非公平锁NonfairSync子类入手：流程：
**1）lock加锁**
获取锁时，首先对同步状态执行CAS操作，尝试把state的状态从0设置为1 -> 
① 返回true：则代表获取同步状态成功，也就是当前线程获取锁成，可操作临界资源；
② 返回false： 则表示已有线程持有该同步状态(其值为1)，获取锁失败，注意这里存在并发的情景，也就是可能同时存在多个线程设置state变量，因此是CAS操作保证了state变量操作的原子性。
    ```
    /**非公平锁实现*/
    static final class NonfairSync extends Sync {
        final void lock() {   //加锁
            if (compareAndSetState(0, 1))    //执行CAS操作，获取同步状态
           //成功则将独占锁线程设置为当前线程  
              setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);   //否则再次请求同步状态
        }
    }
    ```
**2）lock->acquire(1)**
返回false后，执行 acquire(1)-AQS方法，该方法是AQS中的方法，它对中断不敏感，即使线程获取同步状态失败，进入同步队列，后续对该线程执行中断操作也不会从同步队列中移出，方法如下
---传入参数arg：表示要获取同步状态后设置的值(即要设置state的值)；
因为要获取锁，而status为0时是释放锁，1则是获取锁，所以一般传递参数为1，进入方法后首先会执行tryAcquire(arg)-ReetrantLock方法；
在前面分析过该方法在AQS中并没有具体实现，而是交由子类实现，因此该方法是由ReetrantLock类内部实现的
    ```
    public final void acquire(int arg) {   //再次尝试获取同步状态
        if (!tryAcquire(arg) &&  
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    } 
    ```
**3）tryAcquire(arg)**
--tryAcquire(arg)在ReetrantLock的实现
--做了两件事：
① 尝试再次获取同步状态，如果获取成功则将当前线程设置为OwnerThread，否则失败；
② 判断当前线程current是否为OwnerThread，如果是则属于重入锁，state自增1，并获取锁成功，返回true，反之失败，返回false，也就是tryAcquire(arg)执行失败，返回false。
--注意：与公平锁不同的点：
nonfairTryAcquire(int acquires)内部使用的是CAS原子性操作设置state值，可以保证state的更改是线程安全的，因此只要任意一个线程调用nonfairTryAcquire(int acquires)方法并设置成功即可获取锁，不管该线程是新到来的还是已在同步队列的线程；
非公平锁特性，并不保证同步队列中的线程一定比新到来线程请求(可能是head结点刚释放同步状态然后新到来的线程恰好获取到同步状态)先获取到锁。
    ```
    //1 NonfairSync类
    static final class NonfairSync extends Sync {
        protected final boolean tryAcquire(int acquires) {
             return nonfairTryAcquire(acquires);  //由nonfairTryAcquire实现
         }
     }
    //2 Sync类
    abstract static class Sync extends AbstractQueuedSynchronizer {
      final boolean nonfairTryAcquire(int acquires) {   //nonfairTryAcquire方法
          final Thread current = Thread.currentThread();
          int c = getState();
          if (c == 0) {  //判断同步状态是否为0，并尝试再次获取同步状态
              if (compareAndSetState(0, acquires)) {    //执行CAS操作
                  setExclusiveOwnerThread(current);
                  return true;
              }
          }
          //如果当前线程已获取锁，属于重入锁，再次获取锁后将status值加1
          else if (current == getExclusiveOwnerThread()) {
              int nextc = c + acquires;
              if (nextc < 0) // overflow
                  throw new Error("Maximum lock count exceeded");
              //设置当前同步状态，当前只有一个线程持有锁，因为不会发生线程安全问题，可以直接执行 setState(nextc);
              setState(nextc);
              return true;
          }
          return false;
      }
      //省略其他代码
    }
    ```
**4）再看acquire(int arg)**
--理想情况：tryAcquire(arg)返回true，acquireQueued不执行，因为毕竟当前线程已获取到锁；
--tryAcquire(arg)返回false，则会执行addWaiter(Node.EXCLUSIVE)进行入队操作,由于ReentrantLock属于独占锁，因此结点类型为Node.EXCLUSIVE
    ```
    public final void acquire(int arg) {   //再次尝试获取同步状态
        if (!tryAcquire(arg) &&  
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
    ```
**5）addWaiter**
--创建Node：
创建了一个Node.EXCLUSIVE类型Node结点用于封装线程及其相关信息
--tail：其中，tail是AQS的成员变量，指向队尾(这点前面的我们分析过AQS维持的是一个双向的链表结构同步队列)；
-> 如果是第一个结点，则为tail肯定为空，那么将执行enq(node)操作，如果非第一个结点即tail指向不为null，直接尝试执行CAS操作加入队尾，如果CAS操作失败还是会执行enq(node)：
    ```
    private Node addWaiter(Node mode) {
        //将请求同步状态失败的线程封装成结点
        Node node = new Node(Thread.currentThread(), mode);
        Node pred = tail;
        //如果是第一个结点加入肯定为空，跳过。
        //如果非第一个结点则直接执行CAS入队操作，尝试在尾部快速添加
        if (pred != null) {
            node.prev = pred;
            //使用CAS执行尾部结点替换，尝试在尾部快速添加
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        //如果第一次加入或者CAS操作没有成功执行enq入队操作
        enq(node);
        return node;
    }
    ```
**6）enq(node)**
--死循环：使用一个死循环进行CAS操作，可以解决多线程并发问题。
--做了两件事
① 如果还没有初始同步队列则创建新结点并使用compareAndSetHead设置头结点，tail也指向head；
② 队列已存在，则将新结点node添加到队尾。
注意：这两个步骤都存在同一时间多个线程操作的可能，如果有一个线程修改head和tail成功，那么其他线程将继续循环，直到修改成功，这里使用CAS原子操作进行头结点设置和尾结点tail替换可以保证线程安全，从这里也可以看出head结点本身不存在任何数据，它只是作为一个牵头结点，而tail永远指向尾部结点(前提是队列不为null)。
    ```
    private Node enq(final Node node) {
        for (;;) {   //死循环
             Node t = tail;
             //如果队列为null，即没有头结点
             if (t == null) { // Must initialize
                 //创建并使用CAS设置头结点
                 if (compareAndSetHead(new Node()))
                     tail = head;
             } else {//队尾添加新结点
                 node.prev = t;
                 if (compareAndSetTail(t, node)) {
                     t.next = node;
                     return t;
                 } }}}
    ```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021040117002832.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
**7）再看acquire()->acquireQueued()**
--添加到同步队列后，结点就会进入一个自旋过程，即每个结点都在观察时机待条件满足获取同步状态，然后从同步队列退出并结束自旋；
--回到之前的acquire()方法，自旋过程是在acquireQueued(addWaiter(Node.EXCLUSIVE), arg))方法中执行的；
--自旋过程：
---当前线程在自旋(死循环)中获取同步状态，
---当且仅当前驱结点为头结点才尝试获取同步状态，这符合FIFO的规则，即先进先出，其次head是当前获取同步状态的线程结点，只有当head释放同步状态唤醒后继结点，后继结点才有可能获取到同步状态，因此后继结点在其前继结点为head时，才进行尝试获取同步状态，其他时刻将被挂起。
---进入if语句后调用setHead(node)方法，将当前线程结点设置为head
    ```
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {   //自旋，死循环
                final Node p = node.predecessor();   //获取前驱结点
                 // 1 当且仅当p为头结点才尝试获取同步状态
                if (p == head && tryAcquire(arg)) {
                    setHead(node);  //将node设置为头结点
                    p.next = null;  //清空原来头结点的引用便于GC
                    failed = false;
                    return interrupted;
                }
                //2 如果前驱结点不是head，判断是否挂起线程
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);  //最终都没能获取同步状态，结束该线程的请求
        }
    }
    ```
**8）setHead(node)**
--设置为node结点被设置为head后，其thread信息和前驱结点将被清空，因为该线程已获取到同步状态(锁)，正在执行了，也就没有必要存储相关信息了，head只有保存指向后继结点的指针即可；
--便于head结点释放同步状态后唤醒后继结点，执行结果如下图
    ```
    //设置为头结点
    private void setHead(Node node) {
            head = node;
            //清空结点数据
            node.thread = null;
            node.prev = null;
    }
    ```
--从图可知更新head结点的指向，将后继结点的线程唤醒并获取同步状态，调用setHead(node)将其替换为head结点，清除相关无用数据
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401170956117.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
**9）shouldParkAfterFailedAcquire()**
--如果前驱结点不是head执行shouldParkAfterFailedAcquire()方法
--作用：判断当前结点的前驱结点是否为SIGNAL状态(即等待唤醒状态)，如果是则返回true。
如果结点的ws为CANCELLED状态(值为1>0),即结束状态，则说明该前驱结点已没有用应该从同步队列移除，执行while循环，直到寻找到非CANCELLED状态的结点。
倘若前驱结点的ws值不为CANCELLED，也不为SIGNAL(当从Condition的条件等待队列转移到同步队列时，结点状态为CONDITION因此需要转换为SIGNAL)，那么将其转换为SIGNAL状态，等待被唤醒。
--shouldParkAfterFailedAcquire()方法返回true：
即前驱结点为SIGNAL状态同时又不是head结点，那么使用parkAndCheckInterrupt()方法挂起当前线程，称为WAITING状态，需要等待一个unpark()操作来唤醒它，到此ReetrantLock内部间接通过AQS的FIFO的同步队列就完成了lock()操作。
    ```
    //如果前驱结点不是head，判断是否挂起线程
    if (shouldParkAfterFailedAcquire(p, node) &&parkAndCheckInterrupt())
    
          interrupted = true;
    }
    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
            //获取当前结点的等待状态
            int ws = pred.waitStatus;
            //如果为等待唤醒（SIGNAL）状态则返回true
            if (ws == Node.SIGNAL)
                return true;
            //如果ws>0 则说明是结束状态，
            //遍历前驱结点直到找到没有结束状态的结点
            if (ws > 0) {
                do {
                    node.prev = pred = pred.prev;
                } while (pred.waitStatus > 0);
                pred.next = node;
            } else {
                //如果ws小于0又不是SIGNAL状态，
                //则将其设置为SIGNAL状态，代表该结点的线程正在等待唤醒。
                compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
            }
            return false;
        }
    private final boolean parkAndCheckInterrupt() {
            //将当前线程挂起
            LockSupport.park(this);
            //获取线程中断状态,interrupted()是判断当前中断状态，
            //并非中断线程，因此可能true也可能false,并返回
            return Thread.interrupted();
    }
    ```
--总结成逻辑流程图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401171749785.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
* **2 ReetrantLock中非公平锁-可中断lock**
--获取锁的操作，这里看看另外一种可中断的获取方式，即调用ReentrantLock类的**lockInterruptibly()或者tryLock()方法**，最终它们都间接调用到doAcquireInterruptibly()
**1）doAcquireInterruptibly()**
    ```
    
     private void doAcquireInterruptibly(int arg)
            throws InterruptedException {
            final Node node = addWaiter(Node.EXCLUSIVE);
            boolean failed = true;
            try {
                for (;;) {
                    final Node p = node.predecessor();
                    if (p == head && tryAcquire(arg)) {
                        setHead(node);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                    if (shouldParkAfterFailedAcquire(p, node) &&
                        parkAndCheckInterrupt())
                        //直接抛异常，中断线程的同步状态请求
                        throw new InterruptedException();
                }
            } finally {
                if (failed)
                    cancelAcquire(node);
            }
        }
    ```
**--最大的不同是：**
--检测到线程的中断操作后，直接抛出异常，从而中断线程的同步状态请求，移除同步队列。
    ```
    if (shouldParkAfterFailedAcquire(p, node) &&
                        parkAndCheckInterrupt())
         //直接抛异常，中断线程的同步状态请求
           throw new InterruptedException();
    ```

* **3 ReetrantLock中非公平锁-unlock()**
**1）release(1)**
--释放锁实现：
释放同步状态的操作相对简单些，tryRelease(int releases)方法是ReentrantLock类中内部类自己实现的，因为AQS对于释放锁并没有提供具体实现，必须由子类自己实现。
--唤醒：
释放同步状态后会使用unparkSuccessor(h)唤醒后继结点的线程；
    ```
    public void unlock() {  //ReentrantLock类的unlock
        sync.release(1);
    }
    public final boolean release(int arg) { //AQS类的release()方法
        if (tryRelease(arg)) {    //尝试释放锁
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);  //唤醒后继结点的线程
            return true;
        }
        return false;
    }
    
    //ReentrantLock类中的内部类Sync实现的tryRelease(int releases) 
    protected final boolean tryRelease(int releases) {
          int c = getState() - releases;
          if (Thread.currentThread() != getExclusiveOwnerThread())
              throw new IllegalMonitorStateException();
          boolean free = false;
          if (c == 0) {   //判断状态是否为0，如果是则说明已释放同步状态
              free = true;
              setExclusiveOwnerThread(null);   //设置Owner为null
          }
          setState(c);  //设置更新同步状态
          return free;
      }
    ```
**2）unparkSuccessor(h)**
--作用：用unpark()唤醒同步队列中最前边未放弃线程(也就是状态为CANCELLED的线程结点s)。
--前面acquireQueued()：进入自旋的函数acquireQueued()，s结点的线程被唤醒后，会进入acquireQueued()函数的if (p == head && tryAcquire(arg))的判断，如果p!=head也不会有影响，因为它会执行shouldParkAfterFailedAcquire()，由于s通过unparkSuccessor()操作后已是同步队列中最前边未放弃的线程结点，那么通过shouldParkAfterFailedAcquire()内部对结点状态的调整，s也必然会成为head的next结点，因此再次自旋时p==head就成立了，然后s把自己设置成head结点，表示自己已经获取到资源了，最终acquire()也返回了，这就是**独占锁释放的过程**。 
    ```
    private void unparkSuccessor(Node node) {
        //这里，node一般为当前线程所在的结点。
        int ws = node.waitStatus;
        if (ws < 0)  //置零当前线程所在的结点状态，允许失败。
            compareAndSetWaitStatus(node, ws, 0);
    
        Node s = node.next;  //找到下一个需要唤醒的结点s
        if (s == null || s.waitStatus > 0) {//如果为空或已取消
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)  //从这里可以看出，<=0的结点，都是还有效的结点。
                    s = t;
        }
        if (s != null)
            LockSupport.unpark(s.thread);   //唤醒
    }
    ```
--总结：
在AQS同步器中维护着一个同步队列，当线程获取同步状态失败后，将会被封装成Node结点，加入到同步队列中并进行自旋操作，当当前线程结点的前驱结点为head时，将尝试获取同步状态，获取成功将自己设置为head结点。在释放同步状态时，则通过调用子类(ReetrantLock中的Sync内部类)的tryRelease(int releases)方法释放同步状态，释放成功则唤醒后继结点的线程。
* **4 ReetrantLock中公平锁**
--与非公平锁不同的：
在获取锁的时，公平锁的获取顺序是完全遵循时间上的FIFO规则，也就是说先请求的线程一定会先获取锁，后来的线程肯定需要排队，这点与前面我们分析非公平锁的nonfairTryAcquire(int acquires)方法实现有锁不同，下面是公平锁中tryAcquire()方法的实现
--该方法与nonfairTryAcquire(int acquires)方法**唯一的不同是在使用CAS设置尝试设置state值前，调用了hasQueuedPredecessors()判断同步队列是否存在结点，如果存在必须先执行完同步队列中结点的线程，当前线程进入等待状态。**
--这就是非公平锁与公平锁最大的区别：
公平锁在线程请求到来时先会判断同步队列是否存在结点，如果存在先执行同步队列中的结点线程，当前线程将封装成node加入同步队列等待。
非公平锁，当线程请求到来时，不管同步队列是否存在线程结点，直接尝试获取同步状态，获取成功直接访问共享资源。
注意：在绝大多数情况下，非公平锁才是我们理想的选择，毕竟从效率上来说非公平锁总是胜于公平锁。 
    ```
    
    //公平锁FairSync类中的实现
    protected final boolean tryAcquire(int acquires) {
                final Thread current = Thread.currentThread();
                int c = getState();
                if (c == 0) {
                //注意！！这里先判断同步队列是否存在结点
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
    ```
* **5 小结**
 以上便是ReentrantLock的内部实现原理，这里我们简单进行小结，重入锁ReentrantLock，是一个基于AQS并发框架的并发控制类，其内部实现了3个类，分别是Sync、NoFairSync以及FairSync类，其中Sync继承自AQS，实现了释放锁的模板方法tryRelease(int)，而NoFairSync和FairSync都继承自Sync，实现各种获取锁的方法tryAcquire(int)。ReentrantLock的所有方法实现几乎都间接调用了这3个类，因此当我们在使用ReentrantLock时，大部分使用都是在间接调用AQS同步器中的方法，这就是ReentrantLock的内部实现原理,最后给出张类图结构 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401182012555.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
