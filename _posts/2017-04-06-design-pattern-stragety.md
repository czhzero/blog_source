---
title: 设计模式学习笔记-策略模式
date: 2017-04-06 22:02:54
categories: 设计模式
tags: 策略模式
---

在软件开发过程中，常常遇到这样的情况，实现某一个功能可以有多种算法，我们根据实际情况，选择不同的算法来完成这个功能。常规方法是通过if/else条件判断而选择算法。也可以通过策略模式来实现。

<!-- more -->

## 定义:

策略模式定义了一系列的算法，并每个算法封装起来，而且使他们还可以互相替换。策略模式让算法独立于使用它的客户而独立变化。

## 优点:

## 缺点:

## 使用场景:

- 针对同一类型问题的多种处理方式，仅仅是具体行为有差别时
- 需要安全地封装多种同一类型的操作时
- 出现同一抽象类有多个子类，而又需要使用条件判断语句来选择具体子类时。

## 常见实例:

- 动画中的时间插值器

------

- 对加、减、乘每个算各自封装成一个类，首先定义其共同的接口

```
interface ICalculator{
     
     public int calculator(int a, int b);

}

```

- 分别定义加、减、乘算法类

```

class Plus implements ICalculator {

    @Override
    public int calculator(int a, int b) {
        return a + b;
    }

}

class Minus implements ICalculator {

    @Override
    public int calculator(int a, int b) {
        return a - b;
    }

}

class Multiply implements ICalculator {

    @Override
    public int calculator(int a, int b) {
        return a * b;
    }

}


```

- 定义上下文环境类，用于封装各个算法类操作


```
class Context{
    
    private ICalculator iCalculator;
    
    public Context(ICalculator iCalculator){
        this.iCalculator = iCalculator;
    }
    
    public int calculator(int a, int b){
        return iCalculator.calculator(a, b);
    }
    
}

```

- 场景类


```
public class StrategyTest {

    public static void main(String[] args) {
        Context context = new Context(new Plus());
        context.calculator(2, 3);
    }

}

```


