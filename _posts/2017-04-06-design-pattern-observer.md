---
title: 设计模式学习笔记-观察者模式
date: 2017-04-06 23:22:24
categories: 设计模式
tags: 观察者模式
---

观察者模式是一个非常高的模式，简单来讲就是一种订阅发布系统，用来将观察者与被观察者解偶。

<!-- more -->

## 定义:

定义对象间一种一对多的依赖关系，使得每当一个对象改变状态，则所有依赖它的对象都会得到通知并被自动更新。

## 优点:

- 观察者和被观察者之间是抽象耦合，应对业务变化
- 增强系统灵活性、可扩展性

## 缺点:

开发观察者模式需要考虑开放效率和运行效率问题。

- 代码开发调试比较复杂
- 顺序执行的观察者模式，有时不如异步任务。

## 使用场景:

- 关联行为场景，需要注意的是，关联行为说可拆分待，而不是组合关系
- 事件多极触发场景
- 跨系统消息交换场景，如消息队列、事件总线的处理机制

## 常见实例:

- ListView的notifyDataSetChanged()方法


-----

- 定义观察者所具有的共同的接口

```
interface Observer {
    public void update();
}

```

- 定义两个观察者

```
class ObserverA implements Observer {

    @Override
    public void update() {
        System.out.println("ObserverA has received!");
    }
}

```

```
class ObserverB implements Observer {

    @Override
    public void update() {
        System.out.println("ObserverB has received!");
    }
}

```

- 定义被观察者所具有的抽象父类

```
abstract class Observable {

    private Vector<Observer> vector = new Vector<Observer>();

    public void add(Observer observer) {
        vector.add(observer);
    }

    public void del(Observer observer) {
        vector.remove(observer);
    }

    public void notifyObservers() {
        Enumeration<Observer> enumo = vector.elements();
        while (enumo.hasMoreElements()) {
            enumo.nextElement().update();
        }
    }

    public void operation() {

    }
}

```

- 定义具体的被观察者

```
class ConcretObservable extends Observable{
    
    @Override  
    public void operation() {  
        System.out.println("update self!");  
        notifyObservers();  
    }
    
}

```

- 场景类

```
public class ObserverTest {  
      
    public static void main(String[] args) {  
        Observable sub = new ConcretObservable();  
        sub.add(new ObserverA());  
        sub.add(new ObserverB());  
          
        sub.operation();  
    }  
  
}

```
