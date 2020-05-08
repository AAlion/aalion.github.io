---
layout: post
title: 'aop，ioc，cglib，jdk'
categories: Java
date: 2020-04-23 10:27:24
tags:
    - Java
    - Aop
    - Ioc
---

1.  aop，ioc及实现原理，aop及实现，cglib，jdk动态代理实现原理，aop场景题

1） aop基本概念
* 概念：
AOP(Aspect-Oriented Programming)：面向切面的编程。
OOP(Object-Oriented Programming)面向对象的编程。
AOP框架是spring的一个重要组成部分。但是Spring IoC容器并不依赖于AOP，这意味着你有权利选择是否使用AOP，AOP做为Spring IoC容器的一个补充,使它成为一个强大的中间件解决方案。
* 作用：可以进行日志记录,可以进行事务管理,可以进行安全控制,可以进行异常处理,可以进行性能统计
* 3个概念：
a.切面：关注点形成的类，就叫切面(类)。面向切面编程，就是指对很多功能都有的重复的代码抽取，再在运行的时候网业务方法上动态植入“切面类代码”
b.切点：执行目标对象方法，动态植入切面代码。可以通过切入点表达式，指定拦截哪些类的哪些方法； 给指定的类在运行的时候植入切面类代码。
c.通知：在对象上面执行的内容。

<!--more-->

2） aop实现原理

* aop的底层实现是代理模式加反射.
* aop：反射（略）,代理模式分为多种,静态代理和动态代理,动态代理面又分为jdk动态代理和cglib动态代理
* 代理分类：
按照代理的创建时期，代理类可以分为两种。静态代理：由程序员创建或特定工具自动生成源代码，再对其编译。在程序运行前，代理类的.class文件就已经存在了。动态代理：在程序运行时，运用反射机制动态创建而成。 
* 代理模式。
代理模式是常用的java设计模式，他的特征是代理类与委托类有同样的接口，代理类主要负责为委托类预处理消息、过滤消息、把消息转发给委托类，以及事后处理消息等。代理类与委托类之间通常会存在关联关系，一个代理类的对象与一个委托类的对象关联，代理类的对象本身并不真正实现服务，而是通过调用委托类的对象的相关方法，来提供特定的服务
* 代理模式的好处
可以防止对方得到我们真实的方法;

3）Java动态代理（JDK和cglib）
* 代理模式
a.代理模式
** 常用的java设计模式。
** 特征：是代理类与委托类有同样的接口，代理类主要负责为委托类预处理消息、过滤消息、把消息转发给委托类，以及事后处理消息等。
** 代理类和委托类的关系：
代理类与委托类之间通常会存在关联关系，一个代理类的对象与一个委托类的对象关联，代理类的对象本身并不真正实现服务，而是通过调用委托类的对象的相关方法，来提供特定的服务。 
b.代理类的分类（按照代理的创建时期）：
静态代理：由程序员创建或特定工具自动生成源代码，再对其编译。在程序运行前，代理类的.class文件就已经存在了。 
动态代理：在程序运行时，运用反射机制动态创建而成。
c.静态代理
①静态代理例子
Count.java
```java
pacckage net.battier.dao;
/**
 * 定义一个账户接口
 *
 */
 public interface Count{
    //查看账户方法
    public void queryConut();
    
    //修改账户方法
    public void updateCount();
 }
```
CountImpl.java
```java
package net.battier.dao.impl;

import net.battier.dao.Count;

/**
 *委托类（包含业务逻辑）
 *
 */
 public class CountImpl implements Count{
    
    @Override
    public void queryCount(){
        System.out.println("查看账户方法");
    }
    
    @Override
    public void updateCount(){
        System.out.println("修改账户方法...");
    }
    
 }
```
CountProxy.java 
```java
package net.battier.dao.impl;  
  
import net.battier.dao.Count;  
  
/** 
 * 这是一个代理类（增强CountImpl实现类） 
 *  
 */  
public class CountProxy implements Count {  
    private CountImpl countImpl;  
  
    /** 
     * 覆盖默认构造器 
     *  
     * @param countImpl 
     */  
    public CountProxy(CountImpl countImpl) {  
        this.countImpl = countImpl;  
    }  
  
    @Override  
    public void queryCount() {  
        System.out.println("事务处理之前");  
        // 调用委托类的方法;  
        countImpl.queryCount();  
        System.out.println("事务处理之后");  
    }  
  
    @Override  
    public void updateCount() {  
        System.out.println("事务处理之前");  
        // 调用委托类的方法;  
        countImpl.updateCount();  
        System.out.println("事务处理之后");  
  
    }  
  
}  
```
TestCount.java 

```java
import net.battier.dao.impl.CountImpl;  
import net.battier.dao.impl.CountProxy;  
  
/** 
 *测试Count类 
 *  
 */  
public class TestCount {  
    public static void main(String[] args) {  
        CountImpl countImpl = new CountImpl();  
        CountProxy countProxy = new CountProxy(countImpl);  
        countProxy.updateCount();  
        countProxy.queryCount();  
  
    }  
}  
```
输出：
```java
事务处理前
查看账户方法...
处理事务之后
处理事务之前
修改账户方法...
处理事务之后
```
②问题&解决：
** 问题：每一个代理类只能为一个接口服务，这样一来程序开发中必然会产生过多的代理，且所有的代理操作除了调用的方法不一样之外，其他的操作都一样，则此时肯定是重复代码。
** 解决：可以通过一个代理类完成全部的代理功能，那么此时就必须使用动态代理完成。

d.动态代理
① JDK动态代理包含：一个类和一个接口
** InvocationHandler接口
```java
public interface InvocationHandler{
    public Object invoke(Object proxy,Method method,Object[] args) throws Throwable; 
}
// Object proxy 指被代理的对象
// Methhod method 要调用的方法
// Object[] args 方法调用时所需要的参数
//可以将InvocationHandler接口的子类想象成一个代理的最终操作类，替换掉ProxySubject
```
** Proxy类： 
Proxy类是专门完成代理的操作类，可以通过此类为一个或多个接口动态地生成实现类，此类提供了如下的操作方法： 
```java
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, 
InvocationHandler h) 
                               throws IllegalArgumentException 
//ClassLoader loader：类加载器 
//Class<?>[] interfaces：得到全部的接口 
//InvocationHandler h：得到InvocationHandler接口的子类实例
```
Ps:类加载器 
在Proxy类中的newProxyInstance（）方法中需要一个ClassLoader类的实例，ClassLoader实际上对应的是类加载器，在Java中主要有一下三种类加载器; 
Booststrap ClassLoader：此加载器采用C++编写，一般开发中是看不到的； 
Extendsion ClassLoader：用来进行扩展类的加载，一般对应的是jre\lib\ext目录中的类; 
AppClassLoader：(默认)加载classpath指定的类，是最常使用的是一种加载器。 
② 动态代理例子
与静态代理类对照的是动态代理类，动态代理类的字节码在程序运行时由Java反射机制动态生成，无需程序员手工编写它的源代码。动态代理类不仅简化了编程工作，而且提高了软件系统的可扩展性，因为Java 反射机制可以生成任意类型的动态代理类。java.lang.reflect 包中的Proxy类和InvocationHandler 接口提供了生成动态代理类的能力。

BookFacade.java
```java
package net.battier.dao;  
  
public interface BookFacade {  
    public void addBook();  
} 
```
BookFacadeImpl.java 
```java
package net.battier.dao.impl;  
  
import net.battier.dao.BookFacade;  
  
public class BookFacadeImpl implements BookFacade {  
  
    @Override  
    public void addBook() {  
        System.out.println("增加图书方法。。。");  
    }  
  
}  
```
BookFacadeProxy.java 

```java
package net.battier.proxy;  
  
import java.lang.reflect.InvocationHandler;  
import java.lang.reflect.Method;  
import java.lang.reflect.Proxy;  
  
/** 
 * JDK动态代理代理类 
 */  
public class BookFacadeProxy implements InvocationHandler {  
    private Object target;  
    /** 
     * 绑定委托对象并返回一个代理类 
     * @param target 
     * @return 
     */  
    public Object bind(Object target) {  
        this.target = target;  
        //取得代理对象  
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),  
                target.getClass().getInterfaces(), this);   //要绑定接口(这是一个缺陷，cglib弥补了这一缺陷)  
    }  
  
    @Override  
    /** 
     * 调用方法 
     */  
    public Object invoke(Object proxy, Method method, Object[] args)  
            throws Throwable {  
        Object result=null;  
        System.out.println("事物开始");  
        //执行方法  
        result=method.invoke(target, args);  
        System.out.println("事物结束");  
        return result;  
    }  
  
}  
```
TestProxy.java 
```java
package net.battier.test;  
  
import net.battier.dao.BookFacade;  
import net.battier.dao.impl.BookFacadeImpl;  
import net.battier.proxy.BookFacadeProxy;  
  
public class TestProxy {  
  
    public static void main(String[] args) {  
        BookFacadeProxy proxy = new BookFacadeProxy();  
        BookFacade bookProxy = (BookFacade) proxy.bind(new BookFacadeImpl());  
        bookProxy.addBook();  
    }  
  
} 
```
输出：

```java
事务开始
增加图书方法...
事务结束
```
③ 弊端
JDK的动态代理依靠接口实现，如果有些类并没有实现接口，则不能使用JDK代理，这就要使用cglib动态代理了。 
e.Cglib动态代理 
① 介绍 
JDK的动态代理机制只能代理实现了接口的类，而不能实现接口的类就不能实现JDK的动态代理，cglib是针对类来实现代理的，他的原理是对指定的目标类生成一个子类，并覆盖其中方法实现增强，但因为采用的是继承，所以不能对final修饰的类进行代理。
② 示例
BookFacadeCglib.java 
```java
package net.battier.dao;  
  
public interface BookFacade {  
    public void addBook();  
}  
```
BookCadeImpl1.java 
```java
package net.battier.dao.impl;  
  
/** 
 * 这个是没有实现接口的实现类 
 */  
public class BookFacadeImpl1 {  
    public void addBook() {  
        System.out.println("增加图书的普通方法...");  
    }  
}  
```
BookFacadeProxy.java 
```java
package net.battier.proxy;  
  
import java.lang.reflect.Method;  
  
import net.sf.cglib.proxy.Enhancer;  
import net.sf.cglib.proxy.MethodInterceptor;  
import net.sf.cglib.proxy.MethodProxy;  
  
/** 
 * 使用cglib动态代理 
 */  
public class BookFacadeCglib implements MethodInterceptor {  
    private Object target;  
  
    /** 
     * 创建代理对象 
     *  
     * @param target 
     * @return 
     */  
    public Object getInstance(Object target) {  
        this.target = target;  
        Enhancer enhancer = new Enhancer();  
        enhancer.setSuperclass(this.target.getClass());  
        // 回调方法  
        enhancer.setCallback(this);  
        // 创建代理对象  
        return enhancer.create();  
    }  
  
    @Override  
    // 回调方法  
    public Object intercept(Object obj, Method method, Object[] args,  
            MethodProxy proxy) throws Throwable {  
        System.out.println("事物开始");  
        proxy.invokeSuper(obj, args);  
        System.out.println("事物结束");  
        return null;  
  
  
    }  
  
}  
```
TestCglib.java 
```java
package net.battier.test;  
  
import net.battier.dao.impl.BookFacadeImpl1;  
import net.battier.proxy.BookFacadeCglib;  
  
public class TestCglib {  
      
    public static void main(String[] args) {  
        BookFacadeCglib cglib=new BookFacadeCglib();  
        BookFacadeImpl1 bookCglib=(BookFacadeImpl1)cglib.getInstance(new BookFacadeImpl1());  
        bookCglib.addBook();  
    }  
}  
```

3）CGLIB动态代理与JDK动态区别
java动态代理是利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理。

而cglib动态代理是利用asm开源包，对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。

Spring中。

① 如果目标对象实现了接口，默认情况下会采用JDK的动态代理实现AOP

② 如果目标对象实现了接口，可以强制使用CGLIB实现AOP

③ 如果目标对象没有实现了接口，必须采用CGLIB库，spring会自动在JDK动态代理和CGLIB之间转换

JDK动态代理只能对实现了接口的类生成代理，而不能针对类 。
CGLIB是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法 。
因为是继承，所以该类或方法最好不要声明成final ，final可以阻止继承和多态。
[参考1][4]
[参考2][5]

 [4]: https://www.cnblogs.com/jqyp/archive/2010/08/20/1805041.html#3460821
  [5]: https://www.cnblogs.com/xiufengchen/p/10761036.html
