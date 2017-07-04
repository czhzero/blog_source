---
title: Android注解使用快速入门
date: 2017-03-10 10:59:40
categories: Android
tags: 注解
---

注解是那些插入到源码中用于某种工具处理的标签。在java语言中，注解被当作一个修饰符使用的，每个注解的名称前面都加上了@符号。注解自身并不会做任何事情，它需要通过注解事件处理器处理后，才会有用。

注解在Android应用开发当中，使用还是很广泛的。很多热门的框架都使用了注解，例如，butterknife ，retrofit，一些主流的数据库框架等。

一个完整的注解应用通常由注解接口定义、注解事件处理器、注解应用场景类三部分组成。

<!-- more -->

----------

## 1.注解接口定义 ##

定义注解格式：

> public @interface 注解名 {定义体}

注解参数的可支持数据类型：

1.所有基本数据类型（int,float,boolean,byte,double,char,long,short)
2.String类型
3.Class类型
4.enum类型
5.Annotation类型
6.以上所有类型的数组
 
 
 
```
package com.chen.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ActionListener {
    public String source() default "nothing";
}
```

其中，

**@Target**

> @Target说明了Annotation所修饰的对象范围：Annotation可被用于
> packages、types（类、接口、枚举、Annotation类型）、类型成员（方法、构造方法、成员变量、枚举值）、方法参数和本地变量（如循环变量、catch参数）。在Annotation类型的声明中使用了target可更加明晰其修饰的目标。
> 
> 取值(ElementType)有：
> 1.CONSTRUCTOR:用于描述构造器
> 2.FIELD:用于描述域
> 3.LOCAL_VARIABLE:用于描述局部变量
> 4.METHOD:用于描述方法
> 5.PACKAGE:用于描述包
> 6.PARAMETER:用于描述参数
> 7.TYPE:用于描述类、接口(包括注解类型) 或enum声明

**@Retention**

> @Retention定义了该Annotation被保留的时间长短：某些Annotation仅出现在源代码中，而被编译器丢弃；而另一些却被编译在class文件中；编译在class文件中的Annotation可能会被虚拟机忽略，而另一些在class被装载时将被读取（请注意并不影响class的执行，因为Annotation与class在使用上是被分离的）。使用这个meta-Annotation可以对
> Annotation的“生命周期”限制。
> 
> 取值（RetentionPoicy）有：
> 1.SOURCE:在源文件中有效（即源文件保留）, 例如，@Override
> 2.CLASS:在class文件中有效（即class保留）,例如， @NonNull
> 3.RUNTIME:在运行时有效（即运行时保留）, 例如，@Deprecated


## 2.注解事件处理器 ##

 编写注解事件处理器，通过反射获得场景类对象的annotation的source对象。并且为source对象调用setOnClickListener方法进行事件绑定。

```
package com.chen.annotation;

import android.view.View;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * Created by chenzhaohua on 16/1/31.
 */
public class ActionListenerInstaller {

    public static void processAnnotations(Object client) {
        Class<?> clientClass = client.getClass();

        for (Method m : clientClass.getDeclaredMethods()) {

            //获取指定Annotation对象
            ActionListener listener = m.getAnnotation(ActionListener.class);

            if (listener != null) {
                try {
                    Field f = clientClass.getDeclaredField(listener.source());
                    f.setAccessible(true);
                    //控件对象
                    Object focusView = f.get(client);
                    //addListenr函数添加监听，当click事件发生时，调用 onBtnClick() 函数
                    addListenr(focusView, client, m);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }

        }

    }

    private static void addListenr(final Object focusView, final Object client, final Method m) throws Exception {

        InvocationHandler handler = new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                //场景类调用 onBtnClick() 方法
                return m.invoke(client);
            }
        };

        Object onClickListenr = Proxy.newProxyInstance(null, new Class[]{View.OnClickListener.class}, handler);

        Method setOnClickListenerMethod = focusView.getClass().getMethod("setOnClickListener", View.OnClickListener.class);

        setOnClickListenerMethod.invoke(focusView, onClickListenr);

    }

}

```

## 3.注解应用场景类 ##

注解应用场景类中，只需要调用处理器中的processAnnotations方法，通过反射完成click事件的绑定。

同时为click事件指定source控件,添加事件处理函数onBtnClick。

这样一个简单的Android注解应用就完成了。

```
package com.chen.annotation;

import android.app.Activity;
import android.os.Bundle;
import android.widget.Button;



public class MainActivity extends Activity {

    private Button test_btn;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        test_btn = (Button) findViewById(R.id.test_btn);
        ActionListenerInstaller.processAnnotations(this);
    }

    @ActionListener(source = "test_btn")
    public void onBtnClick() {
        android.util.Log.d("czh","CLICK 事件发生了");
    }

}
```