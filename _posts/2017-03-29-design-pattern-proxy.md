---
title: 设计模式学习笔记-代理模式
date: 2017-03-29 22:43:34
categories: 设计模式
tags: 代理模式
---

代理模式(Proxy Pattern)也称为委托模式，是一种结构型设计模式。

<!-- more -->

## 定义:

> 为其他对象提供一种代理以控制对这个对象的访问。

## 优点:

- 职责清晰，真实的角色只要做自己份内的事情，后期通过代理完成一件事务。
- 高扩展性，代理类可以在不修改原来的真实类的情况下，改变逻辑。


## 缺点:

除了增加了一个类之外，没有什么缺点。

## 使用场景:

当无法或者不想直接访问某个对象或者访问某个对象存在困难时可以通过一个代理对象来间接访问，为了保证客户端使用的透明性，委托对象与代理对象要实现相同的接口。

## 常见实例:

－ ActivityManagerProxy代理类，其具体代理的是ActivityManagerNative的子类ActivityManagerService。


----

```
/**
 * 诉讼流程
 */
public interface ILawsuit {
    void submit();

    void burden();

    void defend();

    void finish();
}

```

```
/**
 * 诉讼人
 */
public static class XiaoMing implements ILawsuit {

    @Override
    public void submit() {
        System.out.println("老板拖欠工资,申请仲裁");
    }

    @Override
    public void burden() {
        System.out.println("合同书和银行流水在此");
    }

    @Override
    public void defend() {
        System.out.println("铁证如山,不必多说");
    }

    @Override
    public void finish() {
        System.out.println("诉讼成功,回家等通知");
    }
}

```

```
/**
 * 律师
 */
public static class Lawyer implements ILawsuit {

    private ILawsuit mLawsuit;

    public Lawyer(ILawsuit lawsuit) {
        mLawsuit = lawsuit;
    }

    @Override
    public void submit() {
        System.out.println("律师开始替 " + mLawsuit.getClass().getSimpleName() + "提交诉讼");
        mLawsuit.submit();
        System.out.println("律师提交诉讼完毕");
    }

    @Override
    public void burden() {
        System.out.println("律师开始替 " + mLawsuit.getClass().getSimpleName() + "举证");
        mLawsuit.burden();
        System.out.println("律师举证完成");
    }

    @Override
    public void defend() {
        System.out.println("律师开始替 " + mLawsuit.getClass().getSimpleName() + "辩护");
        mLawsuit.defend();
        System.out.println("律师辩护完成");
    }

    @Override
    public void finish() {
        System.out.println("律师申请" + mLawsuit.getClass().getSimpleName() + "回执");
        mLawsuit.finish();
        System.out.println("律师等待回执");
    }
}

```


```
public static void main(String args[]) {

    XiaoMing user = new XiaoMing();
    Lawyer lawyer = new Lawyer(user);
    lawyer.submit();
    lawyer.burden();
    lawyer.defend();
    lawyer.finish();

}
```

## 代理的分类

从代码到角度，可以分为静态代理和动态代理，静态代理就如上面例子所示，动态代理则是通过Java的InvocationHandler实现。代码如下

```
public static class DynamicProxy implements InvocationHandler {

    private Object object;

    public DynamicProxy(Object obj) {
        this.object = obj;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("律师开始" + method.getName());
        Object result = method.invoke(object, args);
        System.out.println("律师结束" + method.getName());
        return result;
    }
}


```


```
public static void main(String args[]) {

    XiaoMing user = new XiaoMing();

    DynamicProxy dynamicProxy = new DynamicProxy(user);
    ClassLoader loader = user.getClass().getClassLoader();
    ILawsuit dynamicLawyer = (ILawsuit) Proxy.newProxyInstance(loader, new Class[]{ILawsuit.class}, dynamicProxy);
    dynamicLawyer.submit();
    dynamicLawyer.burden();
    dynamicLawyer.defend();
    dynamicLawyer.finish();

}

```

在适用范围上，又可以将代理分为以下几种，

### 普通代理

客户端只能访问代理角色，而不能访问真实角色。

```
public static class Person {

    public String mUserName;

    public Person(String userName) {
        mUserName = userName;
    }

    public void submit() {
        System.out.println("dosomething");
    }
}


public static class Proxy {

    private Person mPerson;

    public Proxy(String userName) {
        mPerson = new Person(userName);
    }

    public void submit() {
        mPerson.submit();
    }

}
    
public static void main(String args[]) {

	 Proxy proxy = new Proxy("xiaoming");
	 proxy.submit();

}
    
    
```

### 强制代理

强制代理的概念是指，要从真实角色找到代理角色，不允许直接访问真实角色，高层模块只要调用getProxy就可以访问真实角色的所有方法。代理管理由真实角色自己管理。

```
public interface IGamePlayer {
    void login();

    void upgrade();

    IGamePlayer getProxy();
}

public static class GamePlayerProxy implements IGamePlayer {

    public GamePlayer mPlayer;

    public GamePlayerProxy(GamePlayer player) {
        mPlayer = player;
    }

    @Override
    public void login() {
        mPlayer.login();
    }

    @Override
    public void upgrade() {
        mPlayer.upgrade();
    }

    @Override
    public IGamePlayer getProxy() {
        return this;
    }
}


public static class GamePlayer implements IGamePlayer {

    public GamePlayerProxy mProxy;
    public String mName;

    public GamePlayer(String name) {
        mName = name;
    }

    @Override
    public void login() {
        if (isMyProxy()) {
            System.out.println("登录游戏");
        } else {
            System.out.println("请使用官方指定代理");
        }
    }

    @Override
    public void upgrade() {
        if (isMyProxy()) {
            System.out.println("打怪升级");
        } else {
            System.out.println("请使用官方指定代理");
        }
    }

    @Override
    public IGamePlayer getProxy() {
        mProxy = new GamePlayerProxy(this);
        return mProxy;
    }

    private boolean isMyProxy() {
        if (mProxy == null) {
            return false;
        } else {
            return true;
        }
    }

}

public static void main(String args[]) {
    GamePlayer player = new GamePlayer("zhangsan");

    player.login();                 //被禁用
    player.upgrade();               //被禁用

    GamePlayerProxy proxy = new GamePlayerProxy(player);
    proxy.login();                  //被禁用
    proxy.upgrade();                //被禁用

    player.getProxy().login();      //指定代理
    player.getProxy().upgrade();    //指定代理
}

```

### 虚拟代理

使用一个代理对象表示一个十分耗资源掉对象，并在真正需要时才创建。

```
public class Proxy implements Subject {

	private Subject subject;
	
	public void request() {
		if(subject == null) {
			subject = new RealSubject();
		}
		subject.request();
	}
	
}

```


### 个性代理

一个代理可以实现多个接口，实现多个任务。代理的主要目的就是在目标对象方法的基础上增强，这种增强的本质就是对目标对象的方法进行拦截和过滤。

### 远程代理

为某个对象在不同的内存地址空间提供局部代理，使系统可以将Server部分的实现隐藏，以便Client可以不必考虑Server的存在。


### 保护代理

使用代理控制对原始对象的访问。该类型的代理常被用于原始对象有不同访问权限的情况。

### 智能引用

在访问原始对象时执行一些自己的附加操作并对指向原始对象的引用计数。



