---
title: Android性能优化(一)－绘制优化
date: 2017-02-07 17:37:31
categories: Android
tags: 性能优化
---


本章为Android性能优化的第一篇——绘制优化，主要包括，性能问题指标，界面卡顿场景和原因，Android系统显示原理，性能分析工具介绍，以及各种优化方式。

<!-- more -->

性能优化系列:

- [Android性能优化(一)－绘制优化](http://www.czhzero.com/2017/02/07/performance-optimization-1/)
- [Android性能优化(一)－存储优化](http://www.czhzero.com/2017/02/07/performance-optimization-2/)
- [Android性能优化(三)－ 网络优化](http://www.czhzero.com/2017/02/07/performance-optimization-3/)
- [Android性能优化(四)－ 代码优化](http://www.czhzero.com/2017/02/07/performance-optimization-4/)
- [Android性能优化(五)－ 安装包大小优化](http://www.czhzero.com/2017/02/07/performance-optimization-5/)
- [Android性能优化(六)－ 省电优化]()
- [Android性能优化(七)－ 内存优化]()
- [Android性能优化(八)－ 稳定优化]()

## 性能问题指标

- 响应时间

指从用户操作开始到系统给用户以正确反馈的时间。一般包括逻辑处理时间,网络传输时间,展现时间。对于非网络类应用不包括网络传输时间。展现时间即网页或App界面渲染时间。响应时间是用户对性能最直接的感受。

- TPS(Transaction Per Second)

TPS为每秒处理的事务数，是系统吞吐量的指标，在搜索系统中也用QPS(Query Per Second)衡量。TPS一般与响应时间反相关。通常所说的性能问题就是指响应时间过长、系统吞吐量过低。

- 系统内存占用

每个Android应用程序都执行在自己的虚拟机中，那了解Java的一定明白，每个虚拟机必定会有堆内存阈值限制（值得一提的是这个阈值一般都由厂商依据硬件配置及设备特性自己设定，没有统一标准，可以为64M，也可以为128M等；它的配置是在Android的属性系统的/system/build.prop中配置dalvik.vm.heapsize=128m即可，若存在dalvik.vm.heapstartsize则表示初始申请大小），也即一个应用进程同时存在的对象必须小于阈值规定的内存大小才可以正常运行。

接着我们运行的App在自己的虚拟机中内存管理基本就是遵循Java的内存管理机制了，系统在特定的情况下主动进行垃圾回收。但是要注意的一点就是在Android系统中执行垃圾回收（GC）操作时所有线程（包含UI线程）都必须暂停，等垃圾回收操作完成之后其他线程才能继续运行。

- 耗电量

在移动设备开发中耗电量是一个非常重要的指标，如果用户一旦发现我们的应用非常耗电，不好意思，他们大多会选择卸载来解决此类问题，所以耗电量是一个十分重要的问题。其实我们一款应用耗电量最大的部分不是UI绘制显示等，常见耗电量最大原因基本都是因为网络数据交互、GPS定位、大量内存性能问题、冗余的后台线程和Service等造成。

## 界面卡顿场景和原因

Android应用卡顿的场景很多，一般有以下几类, UI绘制，应用启动，页面跳转，事件响应。在几个场景下又可以分为几个小类。

- UI
	- 绘制
	- 刷新
- 启动
	- 安装启动
	- 冷启动
	- 热启动
- 跳转
	- 页面间切换
	- 前后台切换
- 响应
	- 按键
	- 系统事件
	- 滑动触摸事件

这四种场景的卡顿原因又可以分为两类。

- 界面绘制

	- 绘制层级深
	- 页面复杂
	- 刷新不合理

- 数据处理
	- 数据处理在UI线程
	- 数据处理占用CPU高，导致主线程拿不到时间片
	- 内存增加导致GC频繁，从而引起卡顿

## Android系统显示原理

Android的显示过程可以简单的概括为：Android应用程序把经过测量、布局、绘制后的surface缓存数据，通过SurfaceFlinger把数据渲染到显示屏幕上，通过Android系统的刷新机制来刷新数据。

### 绘制原理

每个View绘制有三个核心步骤，通过Measure和Layout确定当前需要绘制的View所在的大小和位置，通过Draw方法到surface,在Android系统中整体的绘制源码说在ViewRootImp类的performTraversals()方法。通过这个方法可以看出Measure和Layout都是递归获取View的大小和位置，并且以深度作为优先级。层级越深，元素越多，绘制时间越长。

### 刷新机制

显示数据通过跨进程通信机制将显示数据交给SurfaceFlinger。在Android的显示系统，使用了Android的匿名共享内存：`SharedClient`,每一个应用和SurfaceFlinger之间都会创建一个SharedClient。在每个SharedClient中，最多可以创建31个SharedBufferStack,每个Surface都对应一个SharedBufferStack，也就是一个window。

一个SharedClient对应一个Android应用程序，而一个Android应用程序可能保护多个窗口，即Surface。一个Android应用最多可以包含31个窗口，同时每个SharedBufferStack中又包含了两个(低于4.1版本)或者三个(4.1和以上版本)缓冲区。

总结起来，显示的整个流程为三个模块，应用层绘制到缓存区，SurfaceFlinger把缓存区数据渲染到屏幕，由于是两个不同的进程，所以使用Android的匿名共享内存SharedClient缓存需要显示的数据来达到目的。

如果绘制时间超过16ms(FPS>60)，则用户感觉到卡顿，从4.1开始Android引入了VSYNC、TripleBuffer、Choreographer。

- VSYNC 定时中断，一旦受到中断信号，CPU则开始处理各帧数据
- TripleBuffer， 三缓冲数据，交替切换显示，避免闪烁感
- Choreographer， 收到VSYNC信号，调用用户设置的回调函数


## 性能分析工具

- Android系统的GPU Profile工具, 使用GPU过度绘制分析UI性能, 使用GPU呈现模式图及FPS
- Hierarchy View,查看layout层级
- 使用Memory监测及GC打印与Allocation Tracker进行UI卡顿分析
- 使用Traceview和dmtracedump进行分析优化
- 使用Systrace进行分析优化
- 通过Android SDK中tools目录下的layoutopt命令

更多这些工具的使用，请看这篇文章，[Android应用开发性能优化完全分析](http://blog.csdn.net/yanbober/article/details/48394201)


## 布局优化

### 1. 减少层级

#### 1.1 合理使用RelativeLayout和LinearLayout

RelativeLayout可以减少层级，但是RelativeLayout也存在性能低的问题，原因是其会对子View做两次测量。但是LinearLayout如果使用了weight属性，也会测量两次。

如果布局本事层次太深，还是推荐使用RelativeLayout减少层次，布局层次深会增加内存消耗，甚至引起栈溢出等问题。

因此，

- 尽量使用RelativeLayout和LinearLayout
- 布局层级相同的情况下，使用LinearLayout
- 用LinearLayout使得层数明显变多时，使用RelativeLayout

#### 1.2 合理使用merge

在使用了include后可能导致布局嵌套过多，多余不必要的layout节点，从而导致解析变慢，不必要的节点和嵌套可通过hierarchy viewer(下面布局调优工具中有具体介绍)或设置->开发者选项->显示布局边界查看。

merge标签可用于两种典型情况：

a.布局顶结点是FrameLayout且不需要设置background或padding等属性，可以用merge代替，因为Activity内容试图的parent view就是个FrameLayout，所以可以用merge消除只剩一个。

b.某布局作为子布局被其他布局include时，使用merge当作该布局的顶节点，这样在被引入时顶结点会自动被忽略，而将其子节点全部合并到主布局中。


### 2. 提高显示速度

#### 2.1 ViewStub

ViewStub是一个轻量级的View, 当ViewStub被设置为Visible或者被inflate后，才会被加载和实例化。使用ViewStub需要注意以下几点。

- ViewStub加载一次，之后就不能够通过ViewStub来控制隐藏或者显示。
- ViewStub只能用来加载一个布局文件，而不是某个具体的View。
- ViewStub不能够嵌套merge标签

ViewStub主要应用场景， 

- 在程序运行期间，某个布局加载后，就不会有变化，除非销毁。
- 想要控制显示与隐藏的是一个布局文件，而不是一个View。


#### 2.2 使用include复用布局

include标签常用于将布局中的公共部分提取出来供其他layout共用，以实现布局模块化，这在布局编写方便提供了大大的便利。


## 避免过度绘制

导致过度绘制的原因是，

- XML布局控件有重叠，且都有设置背景
- View的onDraw方法被多次重复执行

### XML避免重叠

- 移除xml中非必要的背景，根据条件设置
- 移除window默认的背景

```
this.getWindow().setBackgroudDrawable(null);

```
- 按需显示占位背景图片


### View的onDraw优化

自定义View中，可以通过`canvas.clipRect()`来帮助系统识别那些可见区域，只有在这个区域内才会被绘制。

> 在listview中，如果itemview比较复杂，建议实现一个自绘View
> 


## 启动优化

### 增加闪屏启动页

闪屏页可以作为品牌宣传展示，同时在闪屏展示的同时，可以进行底层模块的初始化和数据预拉取

### 启动逻辑优化

可以用以下四个维度整理各个启动点。

- 必要且耗时: 启动初始化，考虑用线程来初始化。
- 必要不耗时: 首页绘制
- 非必要要耗时: 数据上报、插件初始化
- 非必要不耗时： 需要时再加载。

把数据整理完成后，按需实现加载逻辑，采取分步加载、异步加载、延期加载等策略。


## 合理的刷新机制

### 减少刷新次数

- 控制刷新频率

没有必要在每次数据变化时都更新对应的控件。

- 避免没有必要的刷新

数据没有变化，就不需要刷新

### 避免后台线程影响

需要刷新时，避免后台开销后台的线程。比如ListView滚动时，暂停其他线程的工作。

### 缩小刷新区域

- 只invalidate需要的区域 - invalidate(0,0,100,100) 

- 只更新需要更新的item - notifyDataSetChanged()

## 提升动画性能

Android平台提供了三种动画框架

- 帧动画
- 补间动画
- 属性动画

### 帧动画

帧动画，就是将一组图片按顺序显示出来，耗费内存资源相当大，能不用就不用。

### 补间动画

补间动画共四种，

- 淡入淡出
- 缩放
- 平移
- 旋转

补间动画使得view重绘非常频繁，补间动画也有其局限性。

- 只能作用于View对象
- 只改变显示效果，不会改变view的属性

### 属性动画

属性动画通过修改实际属性来实现动画效果，相比于补间动画，重绘次数要少得多。

### 硬件加速

Android3.0引入了硬件加速的概念，用来提高渲染速度。硬件加速渲染时，若只有一个view发生变化，只会刷新这个view。通过OpenGLRender负责将RootView中的DisplayList渲染到屏幕上。

如果应用程序中只使用了标准的View或者Drawable,就可以为整个系统打开硬件加速器，但是并不是所有的2D绘制和操作都支持硬件加速。为避免这个问题，Android提供了一个级别控制。

- Application级别

```

<application android:hardwareAccelerated="true" ...>

```

- Activity级别

```

<activity android:hardwareAccelerated="true" ...>

```

- Window级别

```
getWindow().setFlags(WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED);

```

- View级别

```

view.setLayerType(View.LAYER_TYPE_HARDWARE,null);

```

一个View可以使用如下三种Layer类型之一：

- LAYER_TYPE_NONE : 默认普通渲染方式，不会返回一个离屏缓冲。
- LAYER_TYPE_SOFTWARE : 通过软件渲染为一个Bitmap
- LAYER_TYPE_HARDWARE : 如被设置为使用硬件加速，则通过硬件渲染，否则等同于第二种类型。


根据前面的描述，总结出设计一个动画的流程如下，

- 将要执行动画的View的LayerType设置为LAYER_TYPE_HARDWARE
- 计算动画View的属性等信息，更新View属性
- 若动画结束，将LayerType设置为NONE




## 其他

(1) 用SurfaceView或TextureView代替普通View
SurfaceView或TextureView可以通过将绘图操作移动到另一个单独线程上提高性能。
普通View的绘制过程都是在主线程(UI线程)中完成，如果某些绘图操作影响性能就不好优化了，这时我们可以考虑使用SurfaceView和TextureView，他们的绘图操作发生在UI线程之外的另一个线程上。
因为SurfaceView在常规视图系统之外，所以无法像常规试图一样移动、缩放或旋转一个SurfaceView。TextureView是Android4.0引入的，除了与SurfaceView一样在单独线程绘制外，还可以像常规视图一样被改变。
 
(2) 使用RenderJavascript
RenderScript是Adnroid3.0引进的用来在Android上写高性能代码的一种语言，语法给予C语言的C99标准，他的结构是独立的，所以不需要为不同的CPU或者GPU定制代码代码。
 
(3) 使用OpenGL绘图
Android支持使用OpenGL API的高性能绘图，这是Android可用的最高级的绘图机制，在游戏类对性能要求较高的应用中得到广泛使用。
Android 4.3最大的改变，就是支持OpenGL ES 3.0。相比2.0，3.0有更多的缓冲区对象、增加了新的着色语言、增加多纹理支持等等，将为Android游戏带来更出色的视觉体验。
 
(4)尽量为所有分辨率创建资源
减少不必要的硬件缩放，这会降低UI的绘制速度。

(5)尽量减少不必要的背景设置，图片尽量压缩处理显示，尽量避免频繁内存抖动等问题出现。

(6)尽量避免在draw、measure、layout中做过于耗时及耗内存操作，尤其是draw方法中，尽量减少draw、measure、layout等执行次数。

(7)将Activity中的window的背景图设置为空，默认的背景不为空getWindow().setBackgroundDrawable(null)。

(8)View中设置缓存属性.setDrawingCache为true。


详细内容具体参考[Android性能优化总结](http://blog.csdn.net/christopher_411524/article/details/50582740)

## 参考文献

- [Android性能优化总结](http://blog.csdn.net/christopher_411524/article/details/50582740)
- [性能优化之布局优化](http://www.trinea.cn/android/layout-performance/)
- [Android应用开发性能优化完全分析](http://blog.csdn.net/yanbober/article/details/48394201)