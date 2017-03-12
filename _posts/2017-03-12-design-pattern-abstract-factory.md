---
title: 设计模式学习笔记-抽象工厂模式
date: 2017-03-12 16:59:52
categories: 工厂模式
tags: 工厂模式
---
抽象工厂模式也是创建型的设计模式。抽象工厂模式起源于以前对操作系统的图形化解决方案，如不同操作系统的按钮和文本控件，其实现不同。

<!-- more -->

## 定义:

> 为创建一组相关或者是相互依赖的对象提供一个接口，而不需要指定他们的具体类。

## 优点:

－ 封装性,分离接口与实现，客户端只是面向接口编程，不需要具体实现是谁。

## 缺点:

- 类文件爆炸性增加
- 不太容易扩展新的产品类，因为增加一个产品类就要修改抽象工厂，娜美所有具体的工厂类均会被修改。

## 使用场景:

一个对象族有相同约束时可以使用抽象工厂模式，如Android、iOS、WindowPhone下的短信软件和拨号软件。

## 常见实例:

- Activity和Service可以看作一个具体的工厂实习类
- MediaPlayerFactory

----

产品类:

```
public interface IEngine {
    void engine();
}

public class ImportEngine implements IEngine {
    @Override
    public void engine() {
        System.out.println("进口发动机");
    }
}

public class DomesticEngine implements IEngine {
    @Override
    public void engine() {
        System.out.println("国产发动机");
    }
}

public interface ITire {

    void tire();

}

public class NormalTire implements ITire{
    @Override
    public void tire() {
        System.out.println("普通轮胎");
    }
}

public class SUVTire implements ITire{
    @Override
    public void tire() {
        System.out.println("SUV轮胎");
    }
}

```

工厂类:

```
public abstract class CarFactory {

    //生产轮胎
    public abstract ITire createTire();

    //生产发动机
    public abstract IEngine createEngine();

}

public class BMWFactory extends CarFactory {

    @Override
    public ITire createTire() {
        return new SUVTire();
    }

    @Override
    public IEngine createEngine() {
        return new ImportEngine();
    }

}

public class AudiFactory extends CarFactory {

    @Override
    public ITire createTire() {
        return new NormalTire();
    }

    @Override
    public IEngine createEngine() {
        return new DomesticEngine();
    }

}


```

场景类:

```

public static void main() {
    CarFactory audiFactory = new AudiFactory();
    audiFactory.createEngine().engine();
    audiFactory.createTire().tire();

    CarFactory bmwFactory = new BMWFactory();
    bmwFactory.createEngine().engine();
    bmwFactory.createTire().tire();
}

```


## 总结

工厂方法模式：

- 一个抽象产品类，可以派生出多个具体产品类。 　　
- 一个抽象工厂类，可以派生出多个具体工厂类。 
- 每个具体工厂类只能创建一个具体产品类的实例。 

　　        
抽象工厂模式：

- 多个抽象产品类，每个抽象产品类可以派生出多个具体产品类。 
- 一个抽象工厂类，可以派生出多个具体工厂类。 
- 每个具体工厂类可以创建多个具体产品类的实例。 

区别：

- 工厂方法模式只有一个抽象产品类，而抽象工厂模式有多个。
- 工厂方法模式的具体工厂类只能创建一个具体产品类的实例，而抽象工厂模式可以创建多个。



