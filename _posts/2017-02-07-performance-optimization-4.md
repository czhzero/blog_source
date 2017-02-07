---
title: Android性能优化(四)－ 代码优化
date: 2017-02-07 22:54:52
categories: Android
tags: 性能优化
---

主要介绍Java代码中性能优化方法，包括缓存、异步、延迟、数据存储等方式。

<!-- more -->

性能优化系列:

- [Android性能优化(一)－UI优化](http://www.czhzero.com/2017/02/07/performance-optimization-1/)
- [Android性能优化(一)－数据库优化](http://www.czhzero.com/2017/02/07/performance-optimization-2/)
- [Android性能优化(三)－ 移动网络优化](http://www.czhzero.com/2017/02/07/performance-optimization-3/)
- [Android性能优化(四)－ 代码优化](http://www.czhzero.com/2017/02/07/performance-optimization-4/)


## 缓存优化

缓存主要包括对象缓存、IO缓存、网络缓存、DB缓存，对象缓存能减少内存的分配，IO缓存减少磁盘的读写次数，网络缓存减少网络传输，DB缓存较少Database的访问次数。
在内存、文件、数据库、网络的读写速度中，内存都是最优的，且速度数量级差别，所以尽量将需要频繁访问或访问一次消耗较大的数据存储在缓存中

Android常用缓存如下,

(1) 线程池
(2) 大文件/图片缓存
(3) 消息缓存(handler.obtainMessage)
(4) view缓存(ListView的contentView)
(5) 网络缓存(Http缓存)

数据库缓存http response，根据http头信息中的Cache-Control域确定缓存过期时间。

(6) 文件IO缓存

使用具有缓存策略的输入流，BufferedInputStream替代InputStream，BufferedReader替代Reader，BufferedReader替代BufferedInputStream.对文件、网络IO皆适用。

- 数据存储优化

	- 数据类型选择

		字符串拼接用StringBuilder代替String，在非并发情况下用StringBuilder代替StringBuffer。如果你对字符串的长度有大致了解，如100字符左右，可以直接new StringBuilder(128)指定初始大小，减少空间不够时的再次分配。
		64位类型如long double的处理比32位如int慢
		使用SoftReference、WeakReference相对正常的强应用来说更有利于系统垃圾回收
		final类型存储在常量区中读取效率更高
		LocalBroadcastManager代替普通BroadcastReceiver，效率和安全性都更高
	
	- 数据结构选择

		常见的数据结构选择如：
		ArrayList和LinkedList的选择，ArrayList根据index取值更快，LinkedList更占内存、随机插入删除更快速、扩容效率更高。一般推荐ArrayList。
		ArrayList、HashMap、LinkedHashMap、HashSet的选择，hash系列数据结构查询速度更优，ArrayList存储有序元素，HashMap为键值对数据结构，LinkedHashMap可以记住加入次序的hashMap，HashSet不允许重复元素。
		HashMap、WeakHashMap选择，WeakHashMap中元素可在适当时候被系统垃圾回收器自动回收，所以适合在内存紧张型中使用。
		Collections.synchronizedMap和ConcurrentHashMap的选择，ConcurrentHashMap为细分锁，锁粒度更小，并发性能更优。Collections.synchronizedMap为对象锁，自己添加函数进行锁控制更方便。
		 
		Android也提供了一些性能更优的数据类型，如SparseArray、SparseBooleanArray、SparseIntArray、Pair。
		Sparse系列的数据结构是为key为int情况的特殊处理，采用二分查找及简单的数组存储，加上不需要泛型转换的开销，相对Map来说性能更优。不过我不太明白为啥默认的容量大小是10，是做过数据统计吗，还是说现在的内存优化不需要考虑这些东西，写16会死吗，还是建议大家根据自己可能的容量设置初始值。




## 异步，利用多线程提高TPS
充分利用多核Cpu优势，利用线程解决密集型计算、IO、网络等操作。
关于多线程可参考：Java线程池
在Android应用程序中由于系统ANR的限制，将可能造成主线程超时操作放入另外的工作线程中。在工作线程中可以通过handler和主线程交互。
 
## 提前或延迟操作，错开时间段提高TPS

(1) 延迟操作
不在Activity、Service、BroadcastReceiver的生命周期等对响应时间敏感函数中执行耗时操作，可适当delay。
Java中延迟操作可使用ScheduledExecutorService，不推荐使用Timer.schedule;
Android中除了支持ScheduledExecutorService之外，还有一些delay操作，如
handler.postDelayed，handler.postAtTime，handler.sendMessageDelayed，View.postDelayed，AlarmManager定时等。
 
(2) 提前操作
对于第一次调用较耗时操作，可统一放到初始化中，将耗时提前。如得到壁纸wallpaperManager.getDrawable();

## 其他方面


-  用位操作代替乘除法
-  合理利用浮点数，浮点数比整形慢两倍
-  不要重复初始化变量，默认情况下，调用类的构造函数时，Java会把变量初始化成确定的值：所有的对象被设置成null，整数变量（byte、short、int、long）设置成0，float和double设置成0.0，逻辑值设置成false。所有尽量不要重复初始化变量。
      
- 尽量使用局部变量，调用方法时传递的参数以及在调用中创建的临时变量都保存在栈（stack）中，速度较快，并且随所在线程的死亡而自动销毁。其他变量，如静态变量、实例变量等，都在堆（heap）中创建，速度较慢，垃圾回收是的耗能会导致APP出现卡顿现象。

- 尽量指定类的final修饰符。带有final修饰符的类是不可派生的。另外，如果指定一个类为final，则该类所有的方法都是final。Java编译器会寻找机会内联（inline）所有的final方法。此举能够使性能平均提高50%。

- 私有内部类要访问外部类的field或方法时，其成员变量不要用private，因为在编译时会生成setter/getter影响性能。可以把外部类的field或者方法声明为包访问权限。

- 如果方法用不到成员变量，可以把方法申明为static，性能会提高到15%到20%。

- 异常对性能不利。抛出异常首先要创建一个新的对象。Throwable接口的构造函数调用名为fillnStackTrace()的本地(Native)方法，fillnStackTrace()方法检查堆栈，收集调用跟踪信息。只要有异常被抛出，VM就必须调整调用堆栈，因为在处理过程中创建了一个新的对象。异常只能用于错误处理，不应该用来控制程序流程。

## 参考文献

- [性能优化之Java(Android)代码优化](http://www.trinea.cn/android/java-android-performance/)
- [Android性能优化 浅析](http://blog.csdn.net/wtyvhreal/article/details/44172125)
