---
title: Java学习：1. Introduction to Java
date: 2021-03-13 19:07:36
categories: [学习笔记, Java]
tags: [Java]
description: Introduce to java basic information
---

# 1.1 A Typical Java Development Environment

## 1.1.1 The steps to create and execute a Java application

**Phase 1: Edit**
Program is created in an editor and **stored on disk** in a file whose name ends with ***.java***

**Phase 2: Compile**
Compiler creates **bytecodes** and **stores them on disk** in a file whose name ends with ***.class*** ( .class文件的个数取决于类的个数，而不是.java文件的个数）

**Phase 3: Load**
**Class loader** reads ***.class*** files containing bytecodes from disk and **puts those bytecodes in memory**
load 后，类定义以**class类的对象**的形式存储在内存中

**Phase 4: Verify**
**Bytecode verifier** confirms that all bytecodes are valid and do not violate Java's security restrictions

**Phase 5: Execution**
To execute the program, the **JVM** reads bytecodes and **just-in-time(JIT)** **compiles(translates)** them into a language that the computer can understand. As the program executes, it may store data values in **primary memory**

## 1.1.2 Tips

- ***.class*** 以 Java Bytecode 的形式存储
- 在 Ececution 阶段，JVM 既有 interpreter，也有compile。JVM 在解释的同时也有动态/即时编译，即 JIT Compile。(hot spot)

# 1.2 Java Characteristics

- **Simple**
- **Secure**
- **Portable**
- **Robust**
- **Object-oriented**
- **Architecture-neutral**
- **Multithreaded (多线程的)**
  进程 process ----&gt; 线程 thread（多个线程并发）
- **Interpreted (解释性的)**
- **High performance(高性能的)**
- **Distributed(分布式的)**
  Java was designed with the distributed environment. It can be transmit, run over the internet.
- **Dynamic(动态的)**
  Class loader 将类动态加载如JVM中

# 1.3 Different from C++

<table>
<tbody>
<tr>
<th>Features</th>
<th>C++</th>
<th>Java</th>
</tr>
<tr>
<td>pointer</td>
<td>Yes</td>
<td>No</td>
</tr>
<tr>
<td>goto Statement</td>
<td>Yes</td>
<td>No</td>
</tr>
<tr>
<td>unsigned Data Types</td>
<td>Yes</td>
<td>No</td>
</tr>
<tr>
<td>Operator Overloading</td>
<td>Yes</td>
<td>No</td>
</tr>
<tr>
<td>Preprocessor Directives (#define)</td>
<td>Yes</td>
<td>No</td>
</tr>
<tr>
<td>Multiple Inheritance</td>
<td>Yes</td>
<td>No</td>
</tr>
<tr>
<td>Memory Management</td>
<td>Explicit ( 显式 )</td>
<td>Implicit ( 隐式 )</td>
</tr>
<tr>
<td>Structures Concept</td>
<td>Yes</td>
<td>No</td>
</tr>
</tbody>
</table>

- C++支持多继承，Java不支持多继承
- C++为显式的内存管理，Java为隐式的内存管理
- Memory Leak（内存泄漏）: 当没有用的内存没有释放的时候，会造成内存泄漏。C++需要手动释放内存，Java中的JVM自动管理内存。Java程序仍然存在内存泄漏，但要比C++少很多
- 判断一个内存是否应当回收：有无指针指向这片内存（寻找GC Roots）

# 1.4 Others

## 1.4.1 Java Class Libraries

- Java programs consist of pieces called **classes**
- Classed include **methods** that perform tasks and return information when the tasks complete
- **Java class libraries**
- Rich collections of existing classes
- Also known as the **Java APIs**

## 1.4.2 Program Languages

- **Imperative（命令式的）**: C, C++, Java, ...

* 下达命令时很清晰，明确指定方法的步骤
* 问题：冯·诺依曼瓶颈

- **Functional（函数式的）**: SQL( 非过程化的语言 ）, ...

* 这一类语言特点是将函数当变量用，类似Lambda Expressions

- **Logical（逻辑式的）**

## 1.4.3 Class 类

- **Instance of class** represent classes and interfaces in a running Java application.
- class类没有构造函数，class类的对象是Class Loader在load .class 文件时产生的，不能手动new一个