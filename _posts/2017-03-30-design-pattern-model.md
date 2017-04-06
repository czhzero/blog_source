---
title: 设计模式学习笔记-模版方法模式
date: 2017-03-30 23:37:26
categories: 设计模式
tags: 模版方法模式
---

在面向对象开发过程中，通常会遇到这样一类问题。我们知道一个算法所需的关键步骤，并确定这些步骤的执行顺序，但是，某些步骤的具体实现是未知的，或者说某些步骤的具体实现是会随着环境的变化而改变。这时候就需要用到模版方法模式。

<!-- more -->

## 定义:

定义一个操作中的算法的框架，而将一些步骤延迟到子类中，使得子类可以不改变一个算法的结构，即可重定义该算法的某些特定步骤。

## 优点:

- 封装不变部分，扩展可变部分
- 提取公共部分代码，便于维护

## 缺点:

- 模版方法会带来代码阅读的难度，会让用户觉得难以理解

## 使用场景:

- 多个子类有公有的方法，并且逻辑基本相同时。
- 重要、复杂的算法可以把核心算法设计为模版方法，周边细节功能则由子类实现
- 重构时，模版方法模式是一个经常使用的模式，把相同的代码抽取到父类中，然后通过钩子函数约束其行为。

## 常见实例:

- AsyncTask
- Activity的生命周期函数onCreate()等方法均是模版方法的运用


-------


```

public static abstract class CarModel {
    
    //发动
    protected abstract void start();
    
    //鸣笛
    protected abstract void alarm();

    //引擎轰鸣
    protected abstract void engineBoom();
    
    //停止
    protected abstract void stop();
    
    //模版方法,增加final关键字,不允许被复写
    final public void run() {
        start();
        engineBoom();
        if(isAlarm()) {
            alarm();
        }
        stop();
    }
    
    //钩子函数
    protected boolean isAlarm() {
        return true;
    }
    
}
    
    
public static class AudiCar extends CarModel {
    
    private boolean flag;

    @Override
    protected void start() {
        System.out.println("奥迪启动");
    }

    @Override
    protected void alarm() {
        System.out.println("奥迪鸣笛");
    }

    @Override
    protected void engineBoom() {
        System.out.println("奥迪车引擎轰轰作响");
    }

    @Override
    protected void stop() {
        System.out.println("奥迪停车");
    }

    @Override
    protected boolean isAlarm() {
        return flag;
    }
    
    public void setAlarmFlag(boolean flag) {
        this.flag = flag;
    }
    
}
    

public static void main(String[] args) {
    AudiCar car = new AudiCar();
    car.setAlarmFlag(false);
    car.run();
}


```
