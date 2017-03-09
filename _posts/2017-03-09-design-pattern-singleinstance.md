---
title: 设计模式学习笔记-单例模式
date: 2017-03-09 22:57:47
categories: 设计模式
tags: 设计模式
---



单例模式是23种设计模式中最为简单的一种。通常情况下，单例模式实现有两类方式，饿汉式单例和懒汉式单例。

<!-- more -->

定义：

>Ensure a class has only one instance, and provide a global point of access to it.
(确保某一个类只有实例，并向整个系统提供这是实例)

优点:

- 由于单例模式在内存中只有一个实例，减少了内存开支，特别是一个对象需要频繁的创建和销毁时，优势十分明显。
- 由于单例模式只生成一个实例，所以减少了重复的性能开销，如读取配置等。
- 单例模式可以避免对资源的多重占用，例如一个写文件动作，由于只有一个实例存在在内存中，避免对同一个资源文件进行写动作。
- 单例模式可以在系统设置全局的访问点，优化和共享资源访问。

缺点：

- 单例模式一般没有接口，扩展很困难，若要扩展，除了修改代码基本没有其他方途径。
- 单例模式对测试不利，在并行开发环境中，如果单例模式没有完成，是不能进行测试的，没有接口也不能使用mock的方式虚拟一个对象。

使用场景:

- 要求生成唯一序列号的环境
- 在整个项目中需要一个共享访问点或共享数据
- 创建一个对象需要消耗的资源过多，如要访问IO和数据库等资源
- 需要定义大量的静态常量和静态方法的环境

实例:

- Android常见的ImageLoader图片加载库
- Android系统的LayoutInflater类布局加载类


----


具体实现根据实际情况不同，分为以下几种。

1. 懒汉式单例(非线程安全)

```
public class Singleton {  
    private static Singleton instance;  
    private Singleton (){}  
  
    public static Singleton getInstance() {  
    if (instance == null) {  
        instance = new Singleton();  
    }  
    return instance;  
    }  
}
```
这种懒汉式是最通俗易懂的写法，一般非多线程环境下，可直接使用。

2.懒汉式单例（线程安全）

```
public class Singleton {  
    private static Singleton instance;  
    private Singleton (){}  
    public static synchronized Singleton getInstance() {  
    if (instance == null) {  
        instance = new Singleton();  
    }  
    return instance;  
    }  
}  
```

通过synchronized关键字可以保证多线程模式下，内存中实例也会只有一个。但是大部分情况下，并不需要同步，影响效率。

3.饿汉式单例

```
public class Singleton {  
    private static Singleton instance = new Singleton();  
    private Singleton (){}  
    public static Singleton getInstance() {  
    return instance;  
    }  
}
```

通过classloader,在类加载的时候，就会在内存中创建一个实例。不管你需不需要使用这个实例。若在某个逻辑条件下，并需要这个单例实例，就会造成内存的浪费。

4.静态内部类

```
public class Singleton {  
    private static class SingletonHolder {  
    private static final Singleton INSTANCE = new Singleton();  
    }  
    private Singleton (){}  
    public static final Singleton getInstance() {  
    return SingletonHolder.INSTANCE;  
    }  
}  
```
这种方式与第3种方式都是利用classloader进行控制。不同的是第四种方式在装载Singleton类时，并不会去实例化单例。只有在调用getInstance方法后，才会进行加载，实现了懒加载。

5.枚举式

```
class Resource{
}

public enum SomeThing {
    INSTANCE;
    private Resource instance;
    SomeThing() {
        instance = new Resource();
    }
    public Resource getInstance() {
        return instance;
    }
}

```
上面的类Resource是我们要应用单例模式的资源，具体可以表现为网络连接，数据库连接，线程池等等。 
获取资源的方式很简单，只要 SomeThing.INSTANCE.getInstance() 即可获得所要实例。


这种方式是Effective Java作者Josh Bloch 提倡的方式，它不仅能避免多线程同步问题，而且还能防止反序列化重新创建新的对象。



总结：
个人而言，如果实例对象不是很大的话，一般用第三种就可以了。
如果考虑运行时内存大小，推荐第三种，也是比较常用的一种用法。

