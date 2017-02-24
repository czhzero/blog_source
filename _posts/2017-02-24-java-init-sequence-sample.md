---
title: Java类初始化顺序
date: 2017-02-24 22:25:10
categories: Java
tags: 初始化
---

Java程序在它运行之前，并非完全加载，其各个部分是在必需时才加载。Java类的初始化顺序，是先静态对象，而后是非静态对象。类加载器加载class文件时，初始化静态对象。new操作符时，初始化非静态对象。

在类的内部，变量定义的先后决定了初始化的顺序。即使变量定义散布于方法定义之间，它们仍旧会在任何方法(包括构造方法)被调用之前得到初始化。

<!-- more -->

----------

### 1.无继承类初始化

```
public class Client {


	public static class Person {
		public Person(String str) {
			System.out.println(str);
		}
	}
	
	public static class Test {
		
		//静态成员变量 A
		public static Person sPersonA = new Person("A 静态成员变量初始化");
		
		//静态初始化代码块 B
		static {
			System.out.println("B 静态初始化块");
		}
		
		//成员变量 C
		public Person mPersonC = new Person("C 成员变量初始化");
	
		//初始化代码 D
		{
			System.out.println("D 初始化代码块");
		}
		
		//构造函数 E
		public Test() {
			System.out.println("E 构造函数");
		}
		
		
		//静态成员变量 F
		public static Person sPersonF = new Person("F 静态成员变量初始化");
		
		//静态初始化代码块 G
		static {
			System.out.println("G 静态初始化块");
		}
		
		//成员变量 H
		public Person mPersonH = new Person("H 成员变量初始化");
	
		//初始化代码 I
		{
			System.out.println("I 初始化代码块");
		}
		
		
	}
	
	public static void main(String[] args) {
		Test test = new Test();
	}
		
	}
	
```
	
运行结果如下：
	
```
A 静态成员变量初始化
B 静态初始化块
F 静态成员变量初始化
G 静态初始化块
C 成员变量初始化
D 初始化代码块
H 成员变量初始化
I 初始化代码块
E 构造函数
```
	
由此可以得出以下结论：

> **初始化顺序: 静态变量/静态代码块 ->  成员变量/代码块 -> 构造函数**
> 其中，静态变量与静态代码块，成员变量与代码块的初始化顺序只取决于定义顺序。
	

	
### 2. 子类初始化顺序



创建一个TestSub类继承Test类，	
	
```
public static class TestSub extends Test {
	
	//静态成员变量 a
	public static Person sPersonA = new Person("a 静态成员变量初始化");
	
	//静态初始化代码块 b
	static {
		System.out.println("b 静态初始化块");
	}
	
	//成员变量 c
	public Person mPersonC = new Person("c 成员变量初始化");

	//初始化代码 d
	{
		System.out.println("d 初始化代码块");
	}
	
	//构造函数 e
	public TestSub() {
		System.out.println("e 构造函数");
	}
	
	
	//静态成员变量 f
	public static Person sPersonF = new Person("f 静态成员变量初始化");
	
	//静态初始化代码块 g
	static {
		System.out.println("g 静态初始化块");
	}
	
	//成员变量 h
	public Person mPersonH = new Person("h 成员变量初始化");

	//初始化代码 i
	{
		System.out.println("i 初始化代码块");
	}
	
}


public static void main(String[] args) {
	//Test test = new Test();
	TestSub sub = new TestSub();
}
```

运行结果如下：

```
A 静态成员变量初始化
B 静态初始化块
F 静态成员变量初始化
G 静态初始化块
a 静态成员变量初始化
b 静态初始化块
f 静态成员变量初始化
g 静态初始化块
C 成员变量初始化
D 初始化代码块
H 成员变量初始化
I 初始化代码块
E 构造函数
c 成员变量初始化
d 初始化代码块
h 成员变量初始化
i 初始化代码块
e 构造函数
```

由此可以得出以下结论：

> **子类初始化顺序：**
> **父类静态变量/父类静态方法块 -> 子类静态变量/子类静态方法块 -> 父类成员变量/方法块 -> 父类构造函数 ->  子类成员变量/方法块 -> 子类构造函数**

