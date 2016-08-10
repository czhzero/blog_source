---
title: LeakCanary内存泄露检测工具
date: 2016-07-15 21:37:36
categories: Android
tags: 性能优化
---

### LeakCanary

内存泄漏(traditional memory leak)的原因是：当该对象的所有引用都已经释放了，对象仍未被释放。逻辑内存泄漏(logical memory leak)的原因是：当应用不再需要这个对象，当仍未释放该对象的所有引用。如果持有对象的强引用，垃圾回收器是无法在内存中回收这个对象。
常见的原因在[Android内存泄漏常见场景分析](http://www.czhzero.com/2016/07/04/memory-leak-possibility/)一文已经有了详细的介绍。

LeakCanary是一个开源的检测内存泄露的java库。项目地址：https://github.com/square/leakcanary
LeakCanary实际上就是在本机上自动做了Heap dump，对生成的hprof文件分析，展示结果。和手工分析Heap Dump的方式一样。

<!-- more -->

下面是一个LeakCanary的结果截图：

![LeakCanary内存泄露检测工具](http://o7y1sf21i.bkt.clouddn.com/blog/003/1.png)
