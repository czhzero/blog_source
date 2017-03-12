---
title: 设计模式学习笔记-工厂模式
date: 2017-03-12 16:07:32
categories: 设计模式
tags: 设计模式
---

工厂方法模式，是创建型设计模式之一，是一种结构简单的模式。应用广泛，例如Activity的onCreate方法。

<!-- more -->

## 定义:

> 定义一个用于创建对象的接口，让子类决定实例化哪个类。

## 优点:

- 良好的封装性，结构清晰
- 扩展性好，增加产品类十分方便
- 屏蔽产品类，高层只关心产品抽象类

## 缺点:

## 使用场景:

在任何需要生成复杂对象的地方，都可以使用工厂模式。复杂对象适合工厂模式，简单对象就不需要了。

## 常见实例:

- BitmapFactory
- Activity的onCreate方法


-----

通常工厂模式有三种实现方式。

## 反射工厂模式

产品类

```
public abstract class Product {
    public abstract void method();
}

public class ConcreatProductA extends Product {
    @Override
    public void method() {
        System.out.println("i am ProductA");
    }
}

public class ConcreatProductB extends Product {
    @Override
    public void method() {
        System.out.println("i am ProductB");
    }
}

```


```

public abstract class FactoryType1 {
    public abstract <T extends Product> T createProduct(Class<T> c);
}


public class ConcreateFactoryType1 extends FactoryType1 {
    @Override
    public <T extends Product> T createProduct(Class<T> c) {

        Product p = null;

        try {
            p = (Product) Class.forName(c.getName()).newInstance();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }

        return (T) p;
    }
}

public static void main(String args[]) {

    //反射方式实现工厂模式
    FactoryType1 factory = new ConcreateFactoryType1();
    Product A1 = factory.createProduct(ConcreatProductA.class);
    Product B1 = factory.createProduct(ConcreatProductB.class);
    A1.method();
    B1.method();
}

```

## 多工厂模式

```
public abstract class FactoryType2 {
    public abstract Product createProduct();
}

//工厂A
public class ConcreateFactoryType2A extends FactoryType2 {

    @Override
    public Product createProduct() {
        return new ConcreatProductA();
    }
}

//工厂B
public class ConcreateFactoryType2B extends FactoryType2 {

    @Override
    public Product createProduct() {
        return new ConcreatProductB();
    }
}

public static void main(String args[]) {

    //多工厂方法模式
    FactoryType2 factoryType2A = new ConcreateFactoryType2A();
    Product A2 = factoryType2A.createProduct();
    FactoryType2 factoryType2B = new ConcreateFactoryType2B();
    Product B2 = factoryType2B.createProduct();
    A2.method();
    B2.method();
｝

```

## 简单工厂模式(静态工厂模式)

```
public class FactoryType3 {
    public static ConcreatProductA createProductA() {
        return new ConcreatProductA();
    }

}

public static void main(String args[]) {

//简单工厂模式或者静态工厂模式{
Product A3 = FactoryType3.createProductA();
A3.method();

}

```
