---
layout: post
title: 'String的不可变性'
categories: Java知识点
date: 2020-06-21 00:00:00
tags:
    - Java
    - String
---


#### 原因
* String类被final修饰，表示不可被继承。
* String的成员变量char[] value被final修饰，初始化后不可更改引用。
* String的成员变量value访问修饰符为private，不对外界提供修改value数组值的方法。
#### 源码
```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
```
