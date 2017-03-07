---
title: SurfaceView和View的区别
date: 2017-03-07 19:35:17
categories: Android	
tags: surfaceview
---

<!-- more -->

## View简介

View一般在onDraw方法里面绘图，onDraw在UI主线程执行。onDraw默认只在View初始化的时候调用一遍，所以View不会自动刷新画面，一般要调用invalidate或者postInvalidate来重新执行。

onDraw里面的代码进行刷新画面。UI主线程一般用来渲染组件、处理组件与用户之间的交互事件，比如说按钮的点击事件、文本框的输入事件。

如果的画图任务相当繁重，那么onDraw方法里面的代码要执行好长一段时间，就可能会造成UI主线程阻塞。

![](http://o7y1sf21i.bkt.clouddn.com/blog/023/20160811221917915.png)


## SurfaceView简介

Surfaceview是视图（view）的一个继承类，这个视图里内嵌了一个专门用于绘制的Surface。你可以控制这个这个Surface的格式和尺寸，Surfaceview控制这个Surface的绘制位置。

Surfaceview也可以在onDraw里面绘图，即直接在UI主线程绘图并渲染，因为Surfaceview是View的子类。我们可以考虑这样的方案：在后台线程执行繁重的绘图任务，把所有绘制的东西缓存起来；绘制完毕后，再回到UI线程，一次性把所绘制的东西渲染到屏幕上（本质：就是后台线程绘制，UI主线程渲染）

只使用View的onDraw方法是无法实现这种方案的，而Surfaceview可以实现这种方案。先看看Surfaceview的工作原理图：

![](http://o7y1sf21i.bkt.clouddn.com/blog/023/20160811222116554)


## 区别

- View缺乏双缓冲机制，SurfaceView有双缓冲
- 当程序需要更新View上的图像时，程序必须重绘View上显示的整张图片
- SurfaceView是在一个新起的单独线程中可以重新绘制画面，而View必须在UI的主线程中更新画面

