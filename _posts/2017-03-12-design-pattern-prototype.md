---
title: 设计模式学习笔记-原型模式
date: 2017-03-12 15:03:47
categories: 设计模式
tags: 设计模式
---

原型模式是一个创建型的模式。原型二字表明了该模式应该有一个样板实例，用户从这个样板对象中复制出一个内部属性一致的对象。这个过程也就是我们俗称的“克隆，本质就是对象拷贝。

原型模式中的拷贝分为"浅拷贝"和"深拷贝":
浅拷贝: 对值类型的成员变量进行值的复制,对引用类型的成员变量只复制引用,不复制引用的对象.
深拷贝: 对值类型的成员变量进行值的复制,对引用类型的成员变量也进行引用对象的复制

<!-- more -->

## 定义:

> 用原型实例指定创建对象的种类，并通过拷贝这些原型创建新的对象。
> 

## 优点:

- 原型模式是在内存中二进制流的拷贝，要比直接new一个对象的性能好很多，特别是要在一个循环体内产生大量对象时，原型模式可以更好地体现其优点。

## 缺点:

- 直接在内存中的拷贝，构造函数不会知晓，在实际开发当中应该注意这个问题。

## 使用场景:

- 类的初始化需要消化非常多大资源，这些资源包括数据、硬件资源等，通过原型拷贝避免这些消耗。
- 通过new产生一个对象需要非常繁琐的数据准备或者访问权限，这时可以使用原型模式。
- 一个对象需要提供给其他对象访问，而且各个调用者可能都需要修改其值时，可以考虑使用原型模式拷贝多个对象供调用者使用，即是保护性拷贝。


## 常见实例:

- ArrayList的clone方法
- Intent的clone方法

----


```
import java.util.ArrayList;

/**
 * Created by chenzhaohua on 17/3/12.
 */
public class ProtoTypeExample {


    private static class ShallowWordDocument implements Cloneable {

        public String title;
        public ArrayList<String> imageList;

        public ShallowWordDocument() {
            System.out.println("ShallowWordDocument 执行构造函数 ");
        }

        @Override
        protected Object clone() {
            try {
                ShallowWordDocument document = (ShallowWordDocument) super.clone();
                document.title = this.title;
                document.imageList = this.imageList;
                return document;
            } catch (Exception e) {

            }
            return null;
        }

        public void printDocument() {

            System.out.println("-------------  printDocument start -----------");
            System.out.println("标题 :" + title);

            for (String str : imageList) {
                System.out.println("图片 :" + str);
            }
            System.out.println("-------------  printDocument end -----------");
        }

    }


    private static class DeepWordDocument implements Cloneable {

        public String title;
        public ArrayList<String> imageList;

        public DeepWordDocument() {
            System.out.println("DeepWordDocument 执行构造函数 ");
        }

        @Override
        protected Object clone() {
            try {
                DeepWordDocument document = (DeepWordDocument) super.clone();
                document.title = this.title;
                document.imageList = (ArrayList<String>) this.imageList.clone();
                return document;
            } catch (Exception e) {

            }
            return null;
        }

        public void printDocument() {

            System.out.println("-------------  printDocument start -----------");
            System.out.println("标题 :" + title);

            for (String str : imageList) {
                System.out.println("图片 :" + str);
            }
            System.out.println("-------------  printDocument end -----------");
        }

    }


    public static void main(String args[]) {

        ShallowWordDocument orginDoc = new ShallowWordDocument();
        orginDoc.title = "原始文档";
        orginDoc.imageList = new ArrayList<>();
        orginDoc.imageList.add("图片1");
        orginDoc.imageList.add("图片2");


        ShallowWordDocument copyDoc = (ShallowWordDocument) orginDoc.clone();;
        copyDoc.title = "拷贝文档";
        copyDoc.imageList.add("图片3");


        //浅拷贝, 只拷贝title和imageList的地址
        orginDoc.printDocument();
        copyDoc.printDocument();

        DeepWordDocument orginDoc2 = new DeepWordDocument();
        orginDoc2.title = "原始文档";
        orginDoc2.imageList = new ArrayList<>();
        orginDoc2.imageList.add("图片1");
        orginDoc2.imageList.add("图片2");


        DeepWordDocument copyDoc2 = (DeepWordDocument) orginDoc2.clone();;
        copyDoc2.title = "拷贝文档";
        copyDoc2.imageList.add("图片3");


        //深拷贝, 拷贝title和imageList的完整内容
        orginDoc2.printDocument();
        copyDoc2.printDocument();

    }


}


```

运行结果:

```
ShallowWordDocument 执行构造函数 
-------------  printDocument start -----------
标题 :原始文档
图片 :图片1
图片 :图片2
图片 :图片3
-------------  printDocument end -----------
-------------  printDocument start -----------
标题 :拷贝文档
图片 :图片1
图片 :图片2
图片 :图片3
-------------  printDocument end -----------
DeepWordDocument 执行构造函数 
-------------  printDocument start -----------
标题 :原始文档
图片 :图片1
图片 :图片2
-------------  printDocument end -----------
-------------  printDocument start -----------
标题 :拷贝文档
图片 :图片1
图片 :图片2
图片 :图片3
-------------  printDocument end -----------

```

