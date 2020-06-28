---
layout: post
title: 'Java中equals和==的区别'
categories: Java
date: 2019-12-28 00:00:01
tags:
    - Java
    - equals
    - 区别
---

equals和==的区别，两个String之间判别，两个Integer之间判别

* java中的两类数据类型
> * 基本数据类型，也称原始数据类型：
数值型：包含整数型(byte short int long)和浮点型(float double)
字符型：char
布尔型：boolean
> * 引用数据类型
类(class,复合数据类型）:String,Integer,Date
接口（interface）
数组（array）

* "=="
> * 基本数据类型。应用双等号"==",比较的是他们的值。
> * 引用数据类型。当用"=="进行比较的时候，比较的是在内存中的存放地址（堆内存地址）。
注：除非是同一个new出来的对象，比较后的结果为true，否则比较后结果为false。因为每new一次，都会重新开辟堆内存空间。

<!--more-->

* equals()
>* JAVA当中所有的类都是继承于Object这个超类的，类中定义了一个equals的方法。
>* 初始默认行为是比较对象的内存地址值
>* 对于String、Integer、Date复合数据类型之间进行equals比较，在没有覆写equals方法的情况下，他们之间的比较还是内存中的存放位置的地址值，跟"=="的结果相同；如果被复写，按照复写的要求来。
```java
//在Object类中equals()源码：
public boolean equals(Object obj){
//this - s1
//obj -s2
return (this == obj);
}
```

* "=="和equals作用：
>* 基本类型：比较的就是值是否相同
引用类型：比较的就是地址值是否相同
>* 引用类型：默认情况下，比较的是地址值。


* String类
>* 例子
String s1 = "Hello";
String s2 = "Hello";
String s3 = new String("hello");
String s4 = s3; //引用传递
s1 == s2  //true
s1 == s3  //false
s1 == s4  //false
s3 == s4  //true
s1.equals(s3) //true
s1.equals(s4) //true
s3.equals(s4) //true
>* 字符串比较之中“==”和equals()的**区别**:
==：比较的是两个字符串内存地址（堆内存）的数值是否相等，属于数值比较；
 equals()：比较的是两个字符串的内容，属于内容比较。

* 字符串缓冲池
>* 例子
String s1 = "Hello";
String s2 = new String("Hello");
s2 = s2.intern();
s1 == s2 //true
s1 equals s2 //true
>* （java.lang.String的intern()方法"abc".intern()方法的返回值还是字符串"abc"，表面上看起来好像这个方 法没什么用处。但实际上，它做了个小动作：检查字符串池里是否存在"abc"这么一个字符串，如果存在，就返回池里的字符串；如果不存在，该方法会 把"abc"添加到字符串池中，然后再返回它的引用。
）


**参考**
[数据类型][3]
[区别参考1][4][区别参考2][5]

 [3]: https://blog.csdn.net/Coding_Zhu/article/details/53096178
  [4]: https://www.cnblogs.com/qianguyihao/p/3929585.html
  [5]: https://www.cnblogs.com/zhxhdean/archive/2011/03/25/1995431.html
