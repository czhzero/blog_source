---
title: 设计模式学习笔记-建造者模式
date: 2017-03-12 10:53:01
categories: 设计模式
tags: 设计模式
---

Builder模式是一步一步创建一个复杂对象的创建型模式，它允许用户在不知道内部构造细节的情况下，可以更精细的控制对象的构造流程。

<!-- more -->

## 定义:

> 将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。
> 

## 优点:

- 良好的封装性，使用建造者模式可以使客户端不必知道产品内部组成的细节。
- 建造者独立，容易扩展。

## 缺点:

- 会产生多余的Builder对象以及Director对象，额外消耗内存。


## 使用场景:

- 相同的方法，不同的执行顺序，产生不同的事件结果时。
- 多个部件或者零件，都可以装配到一个对象中，但是产生的运行结果又不相同时。
- 产品类非常复杂，或者产品类中的调用顺序不同产生了不同的作用，这个时候使用建造者模式非常合适。
- 当初始化一个对象特别复杂，如参数多，且很多参数都具有默认值时。


## 常见实例:

- AlertDialog.Builder
- ImageLoader的Builder


----

具体实现如下，

## 方式一

- 产品类

```
class Room {

    private Window window;
    private Floor floor;

    public Window getWindow() {
        return window;
    }

    public void setWindow(Window window) {
        this.window = window;
    }

    public Floor getFloor() {
        return floor;
    }

    public void setFloor(Floor floor) {
        this.floor = floor;
    }
}

```

- 定义Builder接口(非必需)

```
interface Builder {

    public void makeWindow();

    public void makeFloor();

    public Room getRoom();
}

```


- 实现Builder

```
class RoomBuilder implements Builder {

    private Room room = new Room();

    @Override
    public void makeWindow() {
        room.setWindow(new Window());
    }

    @Override
    public void makeFloor() {
        room.setFloor(new Floor());
    }

    @Override
    public Room getRoom() {
        return room;
    }

}

```

- 定义Director

```
class Designer {
    
    public void command(Builder builder){
        // 先建造地板
        builder.makeFloor();
        // 再建造窗户
        builder.makeWindow();
    }
}

```

- 场景类

```
public class Client {

    public static void main(String[] args) {
        // 先找来一个工人
        Builder builder = new RoomBuilder();
        // 再找来一个房屋设计师
        Designer designer = new Designer();
        // 工人按照设计师设计建造
        designer.command(builder);
        // 工人向雇主交房子
        Room newRoom = builder.getRoom();
    }

}

```

## 方式二链式调用

```

public class Client {

    public static void main(String[] args) {
        User.Builder builder = new User.Builder();
        User user = builder.setName("corn").setAge(100).setAddress("广州").build();
    }
}

class User {

    private String name;
    private int age;
    private String address;

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public String getAddress() {
        return address;
    }

    public static class Builder {
        private User user = new User();

        public Builder setName(String name) {
            user.name = name;
            return this;
        }

        public Builder setAge(int age) {
            user.age = age;
            return this;
        }

        public Builder setAddress(String address) {
            user.address = address;
            return this;
        }

        public User build() {
            return user;
        }
    }
}

```




