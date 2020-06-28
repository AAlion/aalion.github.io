---
layout: post
title: '线程间交换数据的工具Exchanger'
categories: Java
date: 2019-12-28 00:00:01
tags:
    - Java
    - concurrency
    - 工具
---

### 并发工具
#### 2 Exchanger->线程间交换数据的工具
Exchanger 是一个用于线程间协作的工具类，用于两个线程间能够交换。它提供了一个交换的同步点，在这个同步点两个线程能够交换彼此的数据。
们来模拟这样一个情景，在青春洋溢的中学时代，下课期间，男生经常会给走廊里为自己喜欢的女孩子送情书，相信大家都做过这样的事情吧 ：)。男孩会先到女孩教室门口，然后等女孩出来，教室那里就是一个同步点，然后彼此交换信物，也就是彼此交换了数据。

1. 如下，说话内容有交换：
```java
public class ExchangerDemo {
    private static Exchanger<String> exchanger = new Exchanger<>();

    public static void main(String[] args) {
        //两个线程代表你和快递员
        ExecutorService service = Executors.newFixedThreadPool(2);
        service.execute(()->{
            try {
                //朋友对你说的话
                String friend = exchanger.exchange("等你很久了~~");
                System.out.println("朋友说："+friend);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        service.execute(()->{
            try {
                System.out.println("朋友从远处走来...");
                TimeUnit.SECONDS.sleep(3);
                //你对朋友说的话
                String you=exchanger.exchange("我也等你很久了。。。");
                System.out.println("你说："+you);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
    }
}

```


2. 输出结果
```java
朋友从远处走来...
朋友说：我也等你很久了。。。
你说：等你很久了~~
```
