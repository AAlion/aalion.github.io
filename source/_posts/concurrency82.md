---
layout: post
title: '控制资源并发访问Semaphore'
categories: Java
date: 2019-12-28 00:00:01
tags:
    - Java
    - concurrency
    - 并发工具
---
### 并发工具
#### 一、 Semaphore->控制资源并发访问
1. Semaphore 可以理解为信号量，用于控制资源能够被并发访问的线程数量，以保证多个线程能够合理的使用特定限制资源。

2. 场景：Semaphore 可以用于做流量控制，特别是公共资源有限的应用场景，比如数据库连接。
假如有多个线程读取数据后，需要将数据保存在数据库中，而可用的最大数据库连接只有 10 个，这时候就需要使用 Semaphore 来控制能够并发访问到数据库连接资源的线程个数最多只有 10 个。在限制资源使用的应用场景下，Semaphore 是特别合适的。

3. 场景：我们来模拟这样一样场景。有一天，班主任需要班上 10 个同学到讲台上来填写一个表格，但是老师只准备了 5 支笔，因此，只能保证同时只有 5 个同学能够拿到笔并填写表格，没有获取到笔的同学只能够等前面的同学用完之后，才能拿到笔去填写表格。

 ```java
public class SemaphoreDemo {
    //表示老师只有3支笔
    private static Semaphore semaphore = new Semaphore(3);

    public static void main(String[] args) {
        //表示6个学生使用
        ExecutorService service= Executors.newFixedThreadPool(6);
        for (int i = 0; i < 6; i++) {
            service.execute(()->{
                try {
                    System.out.println(Thread.currentThread().getName()+"学生准备获取笔");
                    semaphore.acquire();//获取许可，如果无法获取到，则阻塞等待直至能够获取为止
                    System.out.println(Thread.currentThread().getName()+"学生获取到笔，并填写表格...");
                    TimeUnit.SECONDS.sleep(3);
                    semaphore.release();
                    System.out.println(Thread.currentThread().getName()+"填写完表格，并还笔~");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
        service.shutdown();
    }
}
 ```
3. 结果输出
 ```java
pool-1-thread-1学生准备获取笔
pool-1-thread-1学生获取到笔，并填写表格...
pool-1-thread-5学生准备获取笔
pool-1-thread-5学生获取到笔，并填写表格...
pool-1-thread-4学生准备获取笔
pool-1-thread-4学生获取到笔，并填写表格...
pool-1-thread-2学生准备获取笔
pool-1-thread-3学生准备获取笔
pool-1-thread-6学生准备获取笔
pool-1-thread-1填写完表格，并还笔~
pool-1-thread-2学生获取到笔，并填写表格...
pool-1-thread-4填写完表格，并还笔~
pool-1-thread-5填写完表格，并还笔~
pool-1-thread-6学生获取到笔，并填写表格...
pool-1-thread-3学生获取到笔，并填写表格...
pool-1-thread-2填写完表格，并还笔~
pool-1-thread-3填写完表格，并还笔~
pool-1-thread-6填写完表格，并还笔~

 ```
 
 #### 其他例子
  ```java
public class JavaConcurrency {

	//控制方法的并发量（Semaphore信号量方案）>>>>>200
	private static Semaphore semaphore = new Semaphore(200);
	
	public static String methodA() throws Exception {
	//无参方法tryAcquire（）的作用是尝试的获得1个许可，如果获取不到则返回false
		if(!semaphore.tryAcquire()) 
			throw new Exception("并发超过200");
		String A = "业务逻辑";
		try {
			// TODO 方法中的业务逻辑
			System.out.println(A);
			return A;
		}finally {
			
			semaphore.release();
		
		}
		
	}
}
 ```
