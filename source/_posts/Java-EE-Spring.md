---
title: 'Java EE: Spring'
date: 2021-03-15 16:44:51
tags: [JavaEE]
categories: [学习笔记, JavaEE]
description: Introduce to Spring framework.
---

> Spring; SpringMVC; Mybatis: ORM的一种(object ralation Mapping)，轻量级

# 1. Spring概述

- Spring是分层的Java SE/EE应用
- 以IoC (Inverse Of Control, 反转控制) 和AOP (Aspect Oriented Programming, 面向切面编程) 为内核
  - AOP是OOP的补充
  - Java是静态语言，AOP使用反射等技术上改变Java的原生类
  - AOP动态生成class对象
  - IoC：创建模式，单例模式，多例模式...

# 2. 面向对象的SOLID原则

- Single Responsibility Principle (单一责任原则)
- Open-Closed Principle (开放封闭原则)
- Liskov Substitution Principle (里氏替换原则)
- Interface Segregation Principle (接口分离原则)
- Dependency Inversion Principle (依赖倒置原则)

# 3. 反射

- RT-TI (Run Time Type Identificaiton)
- 反射的入口是Class类的对象
- 反射在java.lang.reflect包中
- 得到Class类对象的三种方式
- 调用类的构造函数
- 调用类的方法

```java
public class TestReflect {

    public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {
        // 反射：通过Class类的对象为入口

        // 得到Class类的对象
        // 1. getClass方法
        String s = "ab";
        Class aClass = s.getClass(); // 一定有Class类的对象

        // 2. .class
        Class c = int.class; // 不关注类的示例，直接去拿类对象，和类耦合(编译期错误）

        // 3. Class.forName
        // Class c2 = Class.forName("abc11"); // 不关注类的实例，且不与类耦合（运行期错误）



        // 得到constructor
        // Car.class.newInstance(); // 调用类的无参构造函数, 如果没有会报异常

        Constructor constructor = Car.class.getConstructor(int.class); // 依据参数确定构造函数
        Car newCar = (Car) constructor.newInstance(1);



        // 得到method
        Method m = Car.class.getDeclaredMethod("f", null);
        m.invoke(newCar, null);
    }

    public TestReflect(int i) {

    }

    public void f() {
        System.out.println("in car f()...");
    }

}
```

# 4. Spring中的IoC & DI

- IoC：运行时创建对象 (JavaBean对象)
- DI：将创建的对象根据依赖关系注入进去
- Spring是一个管理对象的容器，用<key, value>对的形式来管理

# 5. Annotation的定义与使用

- MetaAnnotation（元注解）：给注解的注解
  - @Documented :被注解的类会被记录在Java documentation中
  - @Target(ElementType.TYPE)：表示注解可以用的地方
  - @Retention(RetentionPolicy.RUNTIME)：表示注解写给谁看的，Annotation最远可以到什么地方
    - SOURCE：编译器
    - CLASS：class loader
    - RUNTIME: JVM
- 属性的定义
- 使用反射来帮助找到Annotation（RUNTIME）
- Annotation之间的关系：
  - 使用注解之间的修饰（相当于继承），除了元注解
  - 注解之间没有继承的关系

```java
@MyAnno(detail = "abc")
public class Test {
    public static void main(String[] args) {
        MyAnno myAnno = (MyAnno) Test.class.getAnnotation(MyAnno.class);
        System.out.println(myAnno.value());
    }
}

@Documented
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@interface MyAnno {
    String detail(); // MyAnno中有一个属性叫做detail
    String value() default "xyz"; // 属性名为value时可以不写"value = ..."
}
```

# 6. Spring中DI的细节

- 一个接口实现两个类，且未指定，注入哪个？
  - 会报异常
  - @Autowired