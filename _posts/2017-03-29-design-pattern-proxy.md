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

在适用范围上，又可以将代理分为以下四种，

### 远程代理

为某个对象在不同的内存地址空间提供局部代理，使系统可以将Server部分的实现隐藏，以便Client可以不必考虑Server的存在。

### 虚拟代理

使用一个代理对象表示一个十分耗资源掉对象，并在真正需要时才创建。

### 保护代理

使用代理控制对原始对象的访问。该类型的代理常被用于原始对象有不同访问权限的情况。

### 智能引用

在访问原始对象时执行一些自己的附加操作并对指向原始对象的引用计数。



