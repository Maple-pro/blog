---
title: Java学习：2. Basic Java Grammar
date: 2021-03-13 19:10:36
categories: [学习笔记, Java]
tags: [Java]
description: Basic Java grammar, data type
---

# 2.1 Data Type

## 2.1.1 基本数据类型

数值类型：byte, short, int, long, float, double
其它类型：char, boolean
（无unsigned类型）

### char

 - 一个char有两个字节；采用UCS-4字符集，UTF-8编码方案；
 - charset（字符集）：将字符与数值一一对应；
 - ASCⅡ：一个字符对应一个字节的编码；
 - GB2312：一个字符对应两个字节的编码；
 - Unicode：一个字符对应四个字节的编码，但表示时用两个字节，英文编码变为双字节，高字节补零；

## 2.1.2 引用数据类型

### 一、JVM内存结构

![JVM内存结构](https://img-blog.csdnimg.cn/20190920235826175.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDA0MjI3MA==,size_16,color_FFFFFF,t_70)

### 二、Java程序在内存中的位置

*person.class* 文件
```java
public class person
{
	int age;
	String name;
	void sleep(){};
}
person p1, p2;
```

 - *person.class* 被load至内存区时，在方法区（**meta space**）中创建**class类的对象**；
 - person类的对象p1, p2在**Java堆**中创建，并由指针指向方法区中class类的对象；
 - object类是所有类的基类，*object* 类中有*getclass()* 方法，可以找到对象所属的类；

<table>
<tbody>
<tr>
<th></th>
<th>局部变量</th>
<th>成员变量</th>
</tr>
<tr>
<td>基本数据类型</td>
<td>变量名和值都在方法区</td>
<td>变量名和值在堆内存中</td>
</tr>
<tr>
<td>引用数据类型</td>
<td>变量在方法栈，变量指向的对象在堆</td>
<td>变量名和变量名指向的对象都在堆</td>
</tr>
</tbody>
</table>

 - Java堆中有GC，方法区中没有GC，为持久代（Permanent generation）；
 - 类的方法在编译后生成一系列指令，放入**方法区**

## 2.1.3 Others

### 一、包装类

 - 8种基本数据类型不是面向对象的，没有对应的类，但每个基本数据类型都有对应的**包装类**（Wrapper Class）
 - **包装类**：Byte, Short, Integer, Character, Long, Float, Double
 - **自动装箱/拆箱**（Auto Boxing/Unboxing）:8种基本数据类型与其包装类会自动相互转换
 - 包装类是**不变类**，是不可更改的（Immutable）
 - 为什么使用不变类：
    * 线程**安全**的，由于不变类的状态在创建后不会发生改变，所以可以进行线程间的数据共享，不需要同步
    * 不变类的instance可以被**重复使用**(reuse)
  - **类型转换**
    * int -> long (true)
    * Integer -> Long (false): Integer类与Long类没有继承关系
    * `Integer a = new Long();`: 错误

### 二、Examples

```java
Integer i1 = 12;
Integer i2 = i1;
i2 = 100;
//此时 i1 = 12; i2 = 100;
//即此时有2个对象，因为包装类是不变类
```
```java
int x=10;
Integer i1 = 10;  //此处有Auto Boxing
Integer i2 = 10;
System.out.println(i1 == i2)
//true
//"=="比较的是地址，即现在只有一个对象（重用）
//当x <= 127时，为true，有一个对象；当x > 128时，为false，有两个对象
```

### 三、静态方法

```java
public class Demo
{
	public static void f(){}
	public void g(){}
}
```

在调用时，

```java
Demo.f();
//静态方法在实现时没有this参数，不会管是由谁调用的，与类的实例无关
new Demo().g();
//非静态方法在实现时会将this参数传入，aload_0[this]，将Demo的对象传入g()方法中
```

### 四、import和package

 - **import**
 - `import java.io.File`
 - `import java.lang.*`会自动加上，不用写
 - **package**
 - `Package a.b.c`
 - 包与目录是相结合的，控制访问范围
 - 当没有访问范围修饰符时，限定在包内访问
### 五、其它
 - 比较
   * == 比较的是物理上的，即比较的是地址
   * 若要比较内容，则需重写equals()方法
 - Javadoc Comments
   * `/**           */`
   * Java.doc会将其转换为HTML文件
 - 文件名
   * 一个Java文件只能最多有一个public class，文件名就是类名
   * 若无public class，也可任取一个top level的类名
   * top level的类只能用public或无