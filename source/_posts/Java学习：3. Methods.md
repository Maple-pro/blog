---
title: Java学习：3. Methods
date: 2021-03-13 19:12:36
categories: [学习笔记, Java]
tags: [Java]
description: Introduce the method-call stack and activation records in java.
---

# 3.1 Method-Call Stack and Activation Records

## 3.1.1 Stack data structure
 - **Pushing**: Placing a dish on the pile at the top
 - **Popping**: Removing a dish from the pile from the top
 - **LIFO** data structures: The last item pushed(inserted) on the stack is the first item popped(removed) from the stack. (**last-in, first-out**)
## 3.1.2 Activation Records
 - When a program calls a method, the called method must know how to return to its caller
   * The **return address** of the calling method is pushed onto the **method-call stack**
 - If a series of method calls occurs, the successive return addresses are pushed onto the stack in last-on, first-out order
 - The **method-call stack** also contains **the memory for the local variables** used in each invocation of a method during a program's execution
   * Stored as a portion of the program-execution stack known as the **activation record** or **stack frame** of the method call
  - When a **method call** is made, the **stack frame** for that method call is **pushed** onto the **method-call stack**
  - When the **method returns** to its caller, the method's activation record is **popped off** the stack and those local variables are no longer known to the program
  - If more method calls occurs than can have their stack frames stored on the method-call stack, an error known as a **stack overflow** occurs
  - **overflow**与**内存**有关
  - 栈有大小限制
  - 对于stack空间，是根据方法的调用与返回来管理的，**不需要GC**
  - stack保存了方法的调用关系
```java
//Demo.java
public class Demo{
	public static void main(String[] args){
		f();
	}
	public static void f(){
		f();
	}
}
//java.lang.StackoverflowError
```
 - main() -> a() -> b() -> c() 在一个**线程**中，
 - 一个java程序在运行时，有多少个**线程**就需要多少个**栈**，每个线程都有自己的栈空间
 - 一个栈中的方法可以跨对象，即一个线程中的方法可以跨对象，**线程只与方法有关**。因为方法不知道什么叫对象，非静态方法在实现时将**this指针** 首先压入栈中。
# 3.2 Constructors
## 3.2.1 普通构造函数
 - **Roles** of the constructors：**Initializing objects**，instand of requesting memory.
 - Java requies a **constructor** that **initializes an object** of a class **when the object is created**.
 - Keyword **new** **requests memory** from the system to store an object, then calls the corresponding class's **constructor** to **initialize** the object.
 - A constructor must have **the same name** as the class.
 - By default, the **compiler** provides a **default constructor** with **no paramaters** in any class that does not explicitly include a constructor. ( 默认构造函数的访问范围与类的访问范围相同 )
   * Instance variables are initialized to their **default values**.
 - A constructor's **parameter list** specifies the data it requires to perform its task. 
 - Constructors **can't return values**, so they cannot specify a return type.
 - If you declare any constructors for a class, the Java compiler will not create a default constructor for that class.
## 3.2.2 静态构造函数
```java
//Demo.java
public class Demo(){
	static{
	
	}  //静态构造函数
	public static void main(String[] args){
	}
}
```
 - 静态构造函数的**功能**：对**类**初始化，通常用于初始化类的**静态成员变量**
 - 调用时间：在**类定义被load到内存中**时，调用了静态构造函数，而不管有无类的对象
 - 不管有多少个对象，都只会调用**一次**静态构造函数
 - 如果在类中没有自己定义构造函数，框架会为我们生成一个默认构造函数；同样的，如果我们在类中定义了静态变量，而我们又没有定义静态构造函数，那么框架也会生成一个静态构造函数供我们调用
```java
//Emp.java
public class Emp(){
	static{
	}
	public static void main(String[] args){
		Emp e1;
	}
}
//不会加载类定义，即不会调用静态构造函数；没有创建对象

//Emp.java
public class Emp(){
	static{
	}
	public static void main(String[] args){
		Emp e1 = new Emp();
	}
}
//会加载类定义，调用了静态构造函数；创建了对象

//Emp.java
public class Emp(){
	static{
	}
	public static void main(String[] args){
		Emp e1;
		Class.forName("Emp");
	}
}
//会加载类定义，调用了静态构造函数；没有创建对象

//类定义的加载与对象的创建是分开的
```

 - 如果一个类加载器只加载一个类，且方法区不做垃圾回收，则静态初始化只做一次
 - 静态构造方法只会有一个，如果写了多个，编译器会将其合在一起
 - 在字节码中，普通构造函数的函数名均为*init*，静态构造函数的函数名为*Cinit*
## 3.2.3 私有构造函数
```java
//Emp.java
class Emp{
	private static Emp mInstance = new Emp();
	public static getInstance(){
		return mInstance;
	}
	private Emp(){
	}
}
Emp e1 = Emp.getInstance();
```
 - Emp这个类**只有一个**对象   **Singleton**
 - mInstance存储在**class类的对象**中
 - 即便没有`Emp e1 = Emp.getInstance();`，仍然会创建出对象
 - 若方法区不做垃圾回收，mInstance不会被当成垃圾，有可能造成**内存泄漏**；*useless but reachable*
 - **Singleto & Multiton**
     * **Singleton**（单例模式）：通过定义私有的构造函数，使从单例类的外部无法初始化该类，从而确保该类只有一个实例
     * **Multiton**（多例模式）：该类有多个实例，且从类的外部无法初始化该类

```java
//Emp.java
class Emp{
	private static Emp mInstance = null;
	public static getInstance(){
		if( mInstance == null ) mInstance = new Emp();
		return mInstance;
	}
	private Emp(){
	}
}
Emp e1 = Emp.getInstance();
```

 - 若没有`Emp e1 = Emp.getInstance();`，没有对象会被创建
# 3.3 Others
## 3.3.1 instanceof() 和 getClass()
比较两个对象是否同属于一个类时，可以使用`instanceof()`和`getClass()`方法。但二者在判断上是有区别的：

```java
public class Test
{
	public static void testInstanceof(Object x)
	{
		System.out.println("x instanceof Parent:  "+(x instanceof Parent));
		System.out.println("x instanceof Child:  "+(x instanceof Child));
		System.out.println("x getClass Parent:  "+(x.getClass() == Parent.class));
		System.out.println("x getClass Child:  "+(x.getClass() == Child.class));
	}
	public static void main(String[] args) {
		testInstanceof(new Parent());
		System.out.println("---------------------------");
		testInstanceof(new Child());
	}
}
class Parent {
 
}
class Child extends Parent {
 
}
/*
输出:
x instanceof Parent:  true
x instanceof Child:  false
x getClass Parent:  true
x getClass Child:  false
---------------------------
x instanceof Parent:  true
x instanceof Child:  true
x getClass Parent:  false
x getClass Child:  true
*/
```

 - `instanceof()`：你是否属于**该类或该类的派生类**，考虑继承关系
 - `getClass()`：不考虑继承关系
## 3.3.2 Override and Overload
 - **Override（重写）**：覆盖一个方法并对其重写
   * 子类的方法与父类的方法的方法名、参数和返回值完全相同，即**方法的标志完全相同**
   * 是**多态性**的表现
   * 通过子类创建的实例对象调用这个方法时，将调用子类中的定义方法，这相当于把父类中定义的那个完全相同的方法给覆盖了
   * 子类方法的**访问权限**只能比父类的更大，不能更小
   * 如果父类的方法是private类型，那么，子类则不存在覆盖的限制，相当于子类中增加了一个全新的方法
   * 编译时编译器决定调用哪个函数（early blinding）
   * `@Override`：一个*annotation*，告诉编译器接下来将要对方法进行重写，也可以不写；如果接下来没有重写方法，编译器会报错
  - **Overload（重载）**：
    * 表示一个类中可以有多个相同名称的方法，但这些方法的**参数列表**各不相同
    * 如果两个方法的参数列表完全一样，不能通过其**返回值类型**来实现重载。因为Java无法通过返回类型来判断调用哪个方法
    * 不能通过**访问范围**进行重载
    * 对于继承来说，如果某一方法在父类中是访问权限是priavte，那么就不能在子类对其进行重载，如果定义的话，也只是定义了一个新方法，而不会达到重载的效果
    * 调用哪个方法由运行时的环境决定，在运行时绑定（late blinding）
## 3.3.3 Others
### 一、定义初始化和构造初始化
```java
//Emp.java
package test;
class Emp{
	int eid = 10;  //定义初始化，初始化语句不在构造函数内
	Emp(int eid){
	this.eid = eid;  //构造初始化，初始化语句在构造函数内
	}
	Emp(){
	}
}

//Emp.class中对应于两条初始化语句
 Emp(int eid);
     0  aload_0 [this]
     1  invokespecial java.lang.Object() [14]
     4  aload_0 [this]
     5  bipush 10
     7  putfield test.Emp.eid : int [16]
    10  aload_0 [this]
    11  iload_1 [eid]
    12  putfield test.Emp.eid : int [16]
Emp();
     0  aload_0 [this]
     1  invokespecial java.lang.Object() [14]
     4  aload_0 [this]
     5  bipush 10
     7  putfield test.Emp.eid : int [16]
```

 - 定义初始化语句在编译后，会放到所有的构造函数的最前面
 - 每一段代码的执行都是由方法调用而来的，不会游离于方法之外
### 二、this(1)
```java
//Emp.java
package test;
class Emp{
	Emp(int eid){
	this.eid = eid;  
	}
	Emp(){
	this(1);  //在一个构造函数中调用另外一个构造函数，即调用Emp（int eid)
	}
}
```

 - **this**:
   * 类似于指针，`this.eid = eid`
   * `this(1)`：在一个构造函数中调用另外一个构造函数
### 三、声明变长参数的方法

```java
public static void add(int... x){
	System.out.println(x.getClass().getName());
}

/*Output:
[I
*/
```

 - `int... x`：实际上，参数是一个int类型的数组
### 四、传参

```java
//Demo.java
package test;
import java.util.List;
import java.util.ArrayList;

public class Demo{
	public static void main(String[] args){
		List<Integer> list = new ArrayList<Integer>();
		list.add(1);
		System.out.println(list.size());
		addElement(list);
		System.out.println(list.size());
	}
	public static void addElement(List<Integer> L){   //传入的是L的地址
		L.add(2);
	}
}
/*Output:
1
2
*/
```
Java中传参只有传值，上面的代码穿的是L的地址，本质上还是在传值
### 五、静态函数和静态成员变量
- static与非static方法的区别：
  * static方法：没有this变量
  * 非static方法：有this变量，先将this压入栈中
  * 都在方法区
- 静态的方法不能访问非静态的成员变量
  * 因为静态方法没有this指针
- 非静态方法可以访问静态成员变量
### 六、流程控制语句
#### for循环

```java
//方法1
for(int i=0; i<=20; i++){
}

//方法2: for(声明循环变量: 数组名){}
int[] ints = new int[]{1, 2, 3, 4, 5};
for(int i: ints){
}
```
#### switch语句

```java
switch(key){
case value:
	break;
default:
	break;
}

String country = "xx"
switch(country){
case "China":
	break;
case "USA":
	break;
}
//Java中switch语句的value可以为String类型
```
#### break & continue

 - **break**：在switch或loop外部中断（中止当前所在的循环语句）
 - **continue**：中止本次循环，开始下一次循环

```java
class Demo
{
	public static void main(String[] args)
	{
		outer: for(int i=0; i<3; i++){
			inner: for(int j=0; j<2; j++){
				System.out.println("hello world");
				break outer;
			}
		}
	}
}
//可以通过标记中止外层循环
```

  