---
title: Android性能优化(一)－UI优化
date: 2017-02-07 17:37:31
categories: Android
tags: 性能优化
---


本文为Android性能优化的第一篇——UI优化，主要介绍使用抽象布局标签(include, viewstub, merge)、去除不必要的嵌套和View节点、减少不必要的infalte及其他Layout方面可调优点，顺带提及布局调优相关工具。

<!-- more -->

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

## 性能调优方式总括

- 利用多线程并发或分布式，提高TPS
- 同步改异步，提高TPS
- 提前或延迟操作，错峰提高TPS
- 缓存(包括对象缓存、IO 缓存、网络缓存等)
- 数据结构和算法优化
- 性能更优的底层接口调用，如 JNI 实现
- 逻辑优化
- 需求优化
- 布局优化


## UI优化

### 1.抽象布局标签

- \<include>标签

include标签常用于将布局中的公共部分提取出来供其他layout共用，以实现布局模块化，这在布局编写方便提供了大大的便利。
下面以在一个布局main.xml中用include引入另一个布局foot.xml为例。main.mxl代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <TextView
        android:id="@+id/tv_text"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_marginBottom="@dimen/dp_80" />

    <include layout="@layout/foot.xml" />

</RelativeLayout>

```

其中include引入的foot.xml为公用的页面底部，代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
 
    <Button
        android:id="@+id/button"
        android:layout_width="match_parent"
        android:layout_height="@dimen/dp_40"
        android:layout_above="@+id/text"/>
 
    <TextView
        android:id="@+id/text"
        android:layout_width="match_parent"
        android:layout_height="@dimen/dp_40"
        android:layout_alignParentBottom="true"
        android:text="@string/app_name" />
 
</RelativeLayout>

```
<include>标签唯一需要的属性是layout属性，指定需要包含的布局文件。可以定义android:id和android:layout_*属性来覆盖被引入布局根节点的对应属性值。注意重新定义android:id后，子布局的顶结点i就变化了。


- \<viewstub>标签 

viewstub标签同include标签一样可以用来引入一个外部布局，不同的是，viewstub引入的布局默认不会扩张，即既不会占用显示也不会占用位置，从而在解析layout时节省cpu和内存。
viewstub常用来引入那些默认不会显示，只在特殊情况下显示的布局，如进度布局、网络失败显示的刷新布局、信息出错出现的提示布局等。
下面以在一个布局main.xml中加入网络错误时的提示页面network_error.xml为例。main.mxl代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
 
	……
    <ViewStub
        android:id="@+id/network_error_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout="@layout/network_error" />
 
</RelativeLayout>

```

其中network_error.xml为只有在网络错误时才需要显示的布局，默认不会被解析，示例代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
 
    <Button
        android:id="@+id/network_setting"
        android:layout_width="@dimen/dp_160"
        android:layout_height="wrap_content"
        android:layout_centerHorizontal="true"
        android:text="@string/network_setting" />
 
    <Button
        android:id="@+id/network_refresh"
        android:layout_width="@dimen/dp_160"
        android:layout_height="wrap_content"
        android:layout_below="@+id/network_setting"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="@dimen/dp_10"
        android:text="@string/network_refresh" />
 
</RelativeLayout>

```

在java中通过(ViewStub)findViewById(id)找到ViewStub，通过stub.inflate()展开ViewStub，然后得到子View，如下：

```
private View networkErrorView;
 
private void showNetError() {
	// not repeated infalte
	if (networkErrorView != null) {
		networkErrorView.setVisibility(View.VISIBLE);
		return;
	}
 
	ViewStub stub = (ViewStub)findViewById(R.id.network_error_layout);
	networkErrorView = stub.inflate();
	Button networkSetting = (Button)networkErrorView.findViewById(R.id.network_setting);
	Button refresh = (Button)findViewById(R.id.network_refresh);
}
 
private void showNormal() {
	if (networkErrorView != null) {
		networkErrorView.setVisibility(View.GONE);
	}
}

```

在上面showNetError()中展开了ViewStub，同时我们对networkErrorView进行了保存，这样下次不用继续inflate。这就是后面第三部分提到的减少不必要的infalte。
viewstub标签大部分属性同include标签类似。

viewstub标签大部分属性同include标签类似。

上面展开ViewStub部分代码

```

ViewStub stub = (ViewStub)findViewById(R.id.network_error_layout);
networkErrorView = stub.inflate();

```

也可以写成下面的形式

```
View viewStub = findViewById(R.id.network_error_layout);
viewStub.setVisibility(View.VISIBLE);   // ViewStub被展开后的布局所替换
networkErrorView =  findViewById(R.id.network_error_layout); // 获取展开后的布局

```

效果一致，只是不用显示的转换为ViewStub。通过viewstub的原理我们可以知道将一个view设置为GONE不会被解析，从而提高layout解析速度，而VISIBLE和INVISIBLE这两个可见性属性会被正常解析。

- \<merge>标签

在使用了include后可能导致布局嵌套过多，多余不必要的layout节点，从而导致解析变慢，不必要的节点和嵌套可通过hierarchy viewer(下面布局调优工具中有具体介绍)或设置->开发者选项->显示布局边界查看。

merge标签可用于两种典型情况：

a.布局顶结点是FrameLayout且不需要设置background或padding等属性，可以用merge代替，因为Activity内容试图的parent view就是个FrameLayout，所以可以用merge消除只剩一个。

b.某布局作为子布局被其他布局include时，使用merge当作该布局的顶节点，这样在被引入时顶结点会自动被忽略，而将其子节点全部合并到主布局中。


### 2.去除不必要的嵌套和View节点

(1) 首次不需要使用的节点设置为GONE或使用viewstub

(2) 使用RelativeLayout代替LinearLayout大约在Android4.0之前，新建工程的默认main.xml中顶节点是LinearLayout，而在之后已经改为RelativeLayout，因为RelativeLayout性能更优，且可以简单实现LinearLayout嵌套才能实现的布局。

(3) 


### 3.减少不必要的infalte

(1) 对于inflate的布局可以直接缓存，用全部变量代替局部变量，避免下次需再次inflate
如上面ViewStub示例中的

```
if (networkErrorView != null) {
	networkErrorView.setVisibility(View.VISIBLE);
	return;
}

```

(2) ListView提供了item缓存，adapter getView的标准写法，如下：

```
@Override
public View getView(int position, View convertView, ViewGroup parent) {
	ViewHolder holder;
	if (convertView == null) {
		convertView = inflater.inflate(R.layout.list_item, null);
		holder = new ViewHolder();
		……
		convertView.setTag(holder);
	} else {
		holder = (ViewHolder)convertView.getTag();
	}
}
 
/**
 * ViewHolder
 * 
 * @author trinea@trinea.cn 2013-08-01
 */
private static class ViewHolder {
 
	ImageView appIcon;
	TextView  appName;
	TextView  appInfo;
}

```

### 4.其他

(1) 用SurfaceView或TextureView代替普通View
SurfaceView或TextureView可以通过将绘图操作移动到另一个单独线程上提高性能。
普通View的绘制过程都是在主线程(UI线程)中完成，如果某些绘图操作影响性能就不好优化了，这时我们可以考虑使用SurfaceView和TextureView，他们的绘图操作发生在UI线程之外的另一个线程上。
因为SurfaceView在常规视图系统之外，所以无法像常规试图一样移动、缩放或旋转一个SurfaceView。TextureView是Android4.0引入的，除了与SurfaceView一样在单独线程绘制外，还可以像常规视图一样被改变。
 
(2) 使用RenderJavascript
RenderScript是Adnroid3.0引进的用来在Android上写高性能代码的一种语言，语法给予C语言的C99标准，他的结构是独立的，所以不需要为不同的CPU或者GPU定制代码代码。
 
(3) 使用OpenGL绘图
Android支持使用OpenGL API的高性能绘图，这是Android可用的最高级的绘图机制，在游戏类对性能要求较高的应用中得到广泛使用。
Android 4.3最大的改变，就是支持OpenGL ES 3.0。相比2.0，3.0有更多的缓冲区对象、增加了新的着色语言、增加多纹理支持等等，将为Android游戏带来更出色的视觉体验。
 
(4) 尽量为所有分辨率创建资源
减少不必要的硬件缩放，这会降低UI的绘制速度。

(5)背景和图片等内存分配优化；尽量减少不必要的背景设置，图片尽量压缩处理显示，尽量避免频繁内存抖动等问题出现。

(6)自定义View等绘图与布局优化；尽量避免在draw、measure、layout中做过于耗时及耗内存操作，尤其是draw方法中，尽量减少draw、measure、layout等执行次数。

(7)避免ANR，不要在UI线程中做耗时操作，遵守ANR规避守则，譬如多次数据库操作等。

### 5.布局调优工具

- hierarchy viewer
- lint
- 使用GPU过度绘制分析UI性能
- 使用GPU呈现模式图及FPS
- 使用Memory监测及GC打印与Allocation Tracker进行UI卡顿分析
- 使用Traceview和dmtracedump进行分析优化
-  使用Systrace进行分析优化


详细内容具体参考[Android性能优化总结](http://blog.csdn.net/christopher_411524/article/details/50582740)

## 参考文献

- [Android性能优化总结](http://blog.csdn.net/christopher_411524/article/details/50582740)
- [性能优化之布局优化](http://www.trinea.cn/android/layout-performance/)