---
layout: post
title: '倒计时器CountDownLatch、循环栅栏CyclicBarrier'
categories: Java
date: 2019-12-28 00:00:01
tags:
    - Java
    - concurrency
    - 并发工具
---
### 并发工具
#### 一、 CountDownLatch->倒计时器 
1. 使用场景：在多线程协作完成业务功能时，有时候需要等待其他多个线程完成任务之后，主线程才能继续往下执行业务功能。例如，在主线程中启动10个子线程去数据库中获取分页数据，需要等到所有线程数据都返回之后统一做统计处理
2. 例子：
6人运动员跑步比赛，裁判员在终点计时，可以想象每当一个运动员到达终点的时候，对于裁判员来说就少了一个计时任务。直到所有运动员都到达终点了，裁判员的任务也才完成。
这 6 个运动员可以类比成 6 个线程，当线程调用 CountDownLatch.countDown 方法时就会对计数器的值减一，直到计数器的值为 0 的时候，裁判员（调用 await 方法的线程）才能继续往下执行。
 ``` java
public class D81_CountDownLatchDemo{
    private static CountDownLatch startSingnal=new CountDownLatch(1);//构造方法
    //用来表示裁判员需要维护的是6个运动员
    public static CountDownLatch endSingnal=new CountDownLatch(6);//构造方法

    public static void main (String[] args) throws InterruptedException{
        // 创建一个固定大小的线程池
        ExecutorService executorService= Executors.newFixedThreadPool(6);
        for (int i = 0; i <6; i++) {
            executorService.execute(()->{
                try {
                    System.out.println(Thread.currentThread().getName()+"运动员等待裁判响哨~");
                    startSingnal.await();//等到构造方法传入的 N 减到 0 的时候，当前调用await方法的线程继续执行
                    System.out.println(Thread.currentThread().getName()+"正在冲刺~");
                    endSingnal.countDown();//使 CountDownLatch 值 N 减 1
                    System.out.println(Thread.currentThread().getName()+"到达终点~");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
        Thread.sleep(1000);
        System.out.println("裁判发令~");
        startSingnal.countDown();
        endSingnal.await();
        System.out.println("全部到达终点，比赛结束~");
        executorService.shutdown();
    }

  }
 ```
2. 结果输出

 ``` java
pool-1-thread-1运动员等待裁判响哨~
pool-1-thread-5运动员等待裁判响哨~
pool-1-thread-3运动员等待裁判响哨~
pool-1-thread-2运动员等待裁判响哨~
pool-1-thread-6运动员等待裁判响哨~
pool-1-thread-4运动员等待裁判响哨~
裁判发令~
pool-1-thread-1正在冲刺~
pool-1-thread-1到达终点~
pool-1-thread-5正在冲刺~
pool-1-thread-5到达终点~
pool-1-thread-3正在冲刺~
pool-1-thread-3到达终点~
pool-1-thread-2正在冲刺~
pool-1-thread-2到达终点~
pool-1-thread-6正在冲刺~
pool-1-thread-6到达终点~
pool-1-thread-4正在冲刺~
pool-1-thread-4到达终点~
全部到达终点，比赛结束~
 ```
#### 二、 CyclicBarrier->循环栅栏
1. 例子
开运动会时，会有跑步这一项运动，我们来模拟下运动员入场时的情况，假设有 6 条跑道，在比赛开始时，就需要 6 个运动员在比赛开始的时候都站在起点了，裁判员吹哨后才能开始跑步。跑道起点就相当于“barrier”，是临界点，而这 6 个运动员就类比成线程的话，就是这 6 个线程都必须到达指定点了，意味着凑齐了一波，然后才能继续执行，否则每个线程都得阻塞等待，直至凑齐一波即可。cyclic 是循环的意思，也就是说 CyclicBarrier 当多个线程凑齐了一波之后，仍然有效，可以继续凑齐下一波。

 ``` java
public class CyclicBarrierDemo {
    //指定必须有6个运动员到达才行,构造方法public CyclicBarrier(int parties, Runnable barrierAction)
    private static CyclicBarrier barrier = new CyclicBarrier(6, () -> {
        System.out.println("所有运动员已入场，裁判吹起跑哨~");
    });
    public static void main(String[] args) {
        System.out.println("运动员准备入场，欢呼~");
        ExecutorService service = Executors.newFixedThreadPool(6);
        for (int i = 0; i < 6; i++) {
            service.execute(() -> {
                try {
                    System.out.println(Thread.currentThread().getName() + "运动员，进场");
                    barrier.await();//等到所有的线程都到达指定的临界点:6人到齐
                    System.out.println(Thread.currentThread().getName() + "运动员出发~");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            });
        }
    }
}
 ```
 
 #### 三、 区别
1.  CountDownLatch 一般用于某个线程 A 等待若干个其他线程执行完任务之后，它才执行；而 CyclicBarrier 一般用于一组线程互相等待至某个状态，然后这一组线程再同时执行；CountDownLatch 强调一个线程等多个线程完成某件事情。CyclicBarrier 是多个线程互等，等大家都完成，再携手共进。
2.  调用 CountDownLatch 的 countDown 方法后，当前线程并不会阻塞，会继续往下执行；而调用 CyclicBarrier 的 await 方法，会阻塞当前线程，直到 CyclicBarrier 指定的线程全部都到达了指定点的时候，才能继续往下执行；
3.  CountDownLatch 方法比较少，操作比较简单，而 CyclicBarrier 提供的方法更多，比如能够通过 getNumberWaiting()，isBroken()这些方法获取当前多个线程的状态，并且 CyclicBarrier 的构造方法可以传入 barrierAction，指定当所有线程都到达时执行的业务功能；
4.  CountDownLatch 是不能复用的，而 CyclicBarrier 是可以复用的。
