---
title: Java学习：4. Array and Collection
date: 2021-03-13 19:14:36
categories: [学习笔记, Java]
tags: [Java]
description: Introduce the array and collection in java.
---

# 4.1 枚举

```java
public class Demo{
	public static void main(String[] args){
		turnTo(Direction.EAST);
	}
	public static void turnTo(Direction direction){
		switch(direction)
		{
		case EAST:
			break;
		case WEST:
			break;
		}
	}
	public enum Direction{ EAST, WEST }; //枚举
}
```
## 4.1.1 枚举的实现
编译后内部实现的方式

```java
//继承java.lang.Enum并声明为final
public final class Direction extends Enum{
	/*枚举类型的常量*/
	public static final Direction EAST;
	public static final Direction WEST;
	private static final Direction[] $VALUES;//values使用数组进行存储
	
	private Direction(String name, int ordinal){
		super(name, ordinal);
	}//私有构造函数，外部无法实例化

	public static Direction[] values(){
		return (Direction[])$VALUES.clone();
	}

	public static Direction valueOf(String name){
		return (Direction)Enum.valueOf(Direction, name);
	}
	
	static{
		EAST = new Direction("EAST", 0);
		WEST = new Direction("WEST", 1);
		$VALUES = ( new Direction[] { EAST, WEST } );
	}
}
```

> Java中枚举实际上是一个特殊的类，编译后会生成相对应的`.class`文件`Demo$Direction.class`，是**多例模式（Multiton）**
## 4.1.2 匿名的内部类
```java
public class Demo{
	public static void main(String[] args){
	}
	enum Direction
	{
		EAST(1), WEST(10){
		};
		int did;
		Direction(int i){
			did=i;
		}
	}
}	
```

> `WEST(10){ };`在编译后会生成`Demo$Direction$1.class`，是Direction类的子类（匿名的内部类）；同时也是Direction类的实例
# 4.2 Array
## 4.2.1 Overview

 - Data structures consisting of related data items of the **same type**.
 - Arrays are objects so they are **reference type**.
 - Elements can be either **primitive** or **reference** types.
 - **Remain** the same **length** once they are created.
 - **Variable-length arguments lists**: can create methods are with varying numbers of arguments.
 - Process command-line *arguments* in method **main**.
 - Common array manipulations with ***static*** methods of class **Arrays** from the `java.utill` package.
 - 数组本身是一个**对象**，其中也可以存放对象，object类是数组类的父类
 - 一维数组中的元素存放在**连续**的存储空间中；多维数组的元素在内存中不连续
   * Java：Jagged Array 锯齿状的数组，多维数组存放在**非连续的内存**中，因此`ints[0].length`与`ints[1].length`可以不同
   * C++：Rregular Array 整齐的数组，多维数组存放在**连续的内存**中，因此`ints[0].length`与`ints[1].length`必须相同
 -  Java数组是一种**引用**数据类型。数组变量并不是数组本身，而是指向**堆内存**中存放的数组对象。因此，可以改变一个数组变量所引用的数组
```java
/*Object类是Array类的父类*/
public class Demo{
	public static void main(String[] args){
	int[] ints = new int[4];
	System.out.println(ints.getClass().getSuperclass().get.Name());
	}
}

//Output
java.lang.Object
```

## 4.2.2 数组的声明

```java
/*一维数组*/
int[] ints = new int[4];//数组的动态初始化；必须要声明数组的大小；Java中默认初始化为0
int[] ints = {1, 2, 3, 4};//数组的静态初始化
String[] strs = new String[4];//对象数组声明后，数组中每一个类对象仍然为空
for(int i: ints){
}
for(int i=0; i<ints.length; i++){
}
```

 - java数组是**静态**的，必须初始化后才可以使用，一旦初始化数组长度，长度是**不可以改变**的
 - **动态初始化** & **静态初始化**：
   * 静态初始化：初始化时由程序员显式指定每个数组元素的初始值，由系统决定数组的长度
   * 动态初始化：初始化时指定数组的长度，此时已经分配内存


```java
/*二维数组*/
int[][] a = new int [4][3] //声明方式1
int[][] ints = new int[4][];
ints[0] = new int[2];
ints[1] = new int[3]; //声明方式2

for(int[] i: a){
	for(int j: i){
	}
} //遍历方式1
for(int i=0; i<a.length; i++{
	for(int j=0; j<a[i].length; j++){
	}
} //遍历方式2
```
# 4.3 Colletion
![Java集合框架图](https://img-blog.csdnimg.cn/20191019223739604.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDA0MjI3MA==,size_16,color_FFFFFF,t_70)

>  - 实线边框：实现类			
>  -  折线边框：抽象类			
>  -  点线边框：接口

具体内容参见：
[集合继承关系图解](https://blog.csdn.net/biexiaofei/article/details/77031003)
[Java集合框架关系图](https://blog.csdn.net/diweikang/article/details/88381601)

## 4.3.1 Overview

 - Collection本身是一个**接口**（*interface*），不是一个类
 - Collection中不可能存放基本数据类型：向ArrayList中放入100时，会先变为Integer类
 - 底层实现的数据结构由程序员决定，可以通过数组实现，也可以通过链表实现
 - 集合的大小并不确定
 - Collection分为List，Set，Queue：
   *  **List**：元素之间有顺序，可以有重复项
      +  **ArrayList**：底层由**数组**实现
      +  **LinkedList**：底层由**链表**实现
   * **Set**：元素之间没有顺序，不可以有重复项
      + **HashSet**
      + **TreeSet**
   * **Queue**
 - **Collection** & **Collections**
   * **Collection**：泛型的**接口**，接口中无法定义具体的方法（即带有方法体的方法），只能定义抽象方法；但在java1.8后可以在接口中定义具体的方法，无需在伴生类中定义
   * **Collections**：工具类，是Collection的**伴生类**，其中全部为**静态方法**，不用创造对象

```java
interface I{
	void f();
	static void g(){
	}
}
```

## 4.3.2 ArrayList

```java
import java.util.List;
import java.util.ArrayList;

public class Demo{
	public static void main(String[] args){
		List list = new ArrayList();
		list.add(1);
		list.add("abc");
		list.get(0);
		System.out.println(list.size());
	}
}
```

 - 基于**动态数组**的一个List，底层用`Object[]`来存储
 - 其中封装了很多方法，如`add()`，`get()`方法
 - **特点**：插入、删除元素很麻烦，但可以实现随机访问
## 4.3.3 LinkedList
 - 基于**链表**的一个List
 - **特点**：插入、删除简便，但不能随机访问
## 4.3.4 HashSet

```java
import java.util.Set
import java.util.HashSet
public class Demo{
	public static void main(String[] args){
		Set<String> set = new HashSet<String>();
	}
}
```
### equals() & hashCode()
Set集合中不允许出现相同的项，Set集合在用`Add()`方法添加一个新项时，首先会调用`equals(Object o)`方法来比较新项与已有的某项是否相等，而不是用`==`来判断相等性。对于字符串等已重写`equals()`方法的类，是按值来比较其相等性的。

在set类型的集合中，如何判断元素是否重复呢，这就需要使用`Object.equals()`方法，但如果元素很多了，添加一个新元素时，比较的次数 就很多，例如已经有100个元素了，添加第101个元素时，就要和前面的元素比较100次，效率很低。

**HashSet**类用**哈希算法**作为搜索算法来存取对象，当向集合中加入一个新对象时，会调用对象的`hashCode()`方法得到对象的哈希码，从而判断两个对象是否相等，复杂度为O(1)。

> 哈希算法也称为散列算法，是将数据依据算法直接指定到一个地址上，`hashCode()`实际上返回的是对象存储的物理地址。
> **Object类**中定义了`hashCode()`和`equals(Object o)`方法,如果`object1.equals(object2)`，那么说明这两个引用变量指向同一个对象，那么object1和object2的hashCode也一定相等。

[重写equals和hashCode](https://blog.csdn.net/kangguowei/article/details/82458265)

调用`set.add()`时，先用`hashCode()`方法，再对哈希码相同的对象调用`equals()`方法
 > 只用`equals()`方法：效率低
 > 只用`hashCode()`方法：不可靠
## 4.3.5 TreeSet
### 一、Comparable & Comparator

 - **Comparable**
 -  **Packaga** `java.lang`
 -  Interface Comparable<T>
 - This ordering is referred to as the class's natural ordering, and the class's **compareTo** method is referred to as its natural comparison method.（自然排序算法）
 - `int compareTo(T o)`：有this参数
 - **Comparator**
 - **Package** `java.util`
 - public interface Comparator<T>
 - `int compare(T o1, T o2)`：没有this参数
 - 所使用的Comparator可以是当前类的，也可以是其父类的（泛型编程）
 > List之间的区别：底层实现的数据结构不同
 > Set之间的区别：底层的数据结构以及搜索算法不同
### 二、构造函数
| Constructor                                | Description                                                  |
| ------------------------------------------ | ------------------------------------------------------------ |
| TreeSet()                                  | Constructs a new, empty tree set, sorted according to the **natural ordering** of its elements. |
| TreeSet (Collection<? extends E> c)        | Constructs a new tree set containing the elements in the specified collection, sorted according to the **natural ordering** of its elements. |
| TreeSet (Comparator<? super E> comparator) | Constructs a new, empty tree set, sorted according to the **specified comparator**. |
| TreeSet (SortedSet<E> s>)                  | Constructs a new tree set comtaining the same elements and using the **same ordering** as the specified sorted set. |
### 三、Brief Introduction
 - 放入TreeSet的对象一定要能比较大小
 - 同一类型及其子类型的对象可以放入一个TreeSet
 - 默认采用自然排序（Comparable的`compareTo()`方法）
 - 可以重写`compareTo()`或new不同的Comparator

## 4.3.6 TreeMap & HashMap
> public interface Map<K, V>
> Map不属于Collection
> Map中的一个key和一个value构成一个entry
> `Set<Map.Entry<K, V>> entrySet()` 会产生一个set

 -  Map这个接口取代了Dictionary这个类
 - TreeMap和HashMap的区别在于查找算法的不同

# 4.4 Array和Collection的区别

```java
import java.util.List;
import java.util.ArrayList;

public class Demo{
	public static void main(String[] args){
		/*Example1*/
		Person p = new Emp(); //p仍然为Person类的对象
		Emp e = null;
		p = e; //yes
		e = p; //no
		e = (Emp)p; //yes （如果没有继承关系，则no）

		/*Example2*/
		List<Person> plist = null;
		List<Emp> emplist = null;
		plist = emplist; //no
		emplist = plist; //no
		g(emplist); //no
		//Person与Emp有继承关系，但plist与emplist没有继承关系，集合为invariant不变的

		/*Example3*/
		Person[] persons = null;
		Emp[] emps = null;
		persons = emps; //yes
		emps = persons; //no
		f(emps) //yes
		//Person类是Emp类的父类，persons也是emps的父类，数组为covariant协变的
	}
	public static void f(Persons[] persons){
	}
	public static void g(List<Person> plist){
	}
}

class person{
}
class Emp extends Person{
}
```

 > **Array**：covariant 协变的
 > **Collection**：invariant 不变的
 > （contravariant 逆变的）
 - 为什么不把集合做成协变的？
   * 集合中有**类型擦除**，所以若集合是协变的，则有可能add不属于该集合的对象进入
   * 而数组没有类型擦除，所以在编译时便会报错
 - Java泛型编程中可以利用一些机制达到协变的效果

```java
//协变返回：通过编译后产生的桥方法实现
class Person{
	Person f(){
		return null;
	}
}
class Emp extends Person{
	Emp f(){
		return null;
	}
}
```
# 4.5 Iterator
> public interface Iterator<E>
> This interface is a member of the Java Collections Framework

 - 循环遍历是通过Iterator来实现的，通过Iterable中的`iterator()`方法获得迭代器
 - 要使用Iterator必须要是iterable的