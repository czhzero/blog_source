---
title: Android Studio 使用小技巧和快捷键
date: 2016-06-28 22:34:39
categories: Android
tags: studio
---

原文地址:http://www.open-open.com/lib/view/open1458715872710.html

### 写在前面

本文翻译自 [Android Studio Tips by Philippe Breault](https://github.com/pavlospt/Android-Studio-Tips-by-Philippe-Breault/wiki)，一共收集了62个 Android Studio 使用小技巧和快捷键。 根据这些小技巧的使用场景，本文将这62个小技巧分为常用技巧（1 – 28）、编码技巧（29 – 49）和调试技巧（50 – 62），分成三个部分。

<!-- more -->

### 1.书签

- 描述：这是一个很有用的功能，让你可以在某处做个标记（书签），方便后面再跳转到此处。
- 调用：Menu → Navigate → Bookmarks
- 快捷键：
 + 添加/移除书签：F3(OS X) 、F11(Windows/Linux);
 + 添加/移除书签(带标记)：Alt + F3(OS X)、Ctrl + F11(Windows/Linux);
 + 显示全部书签：Cmd + F3(OS X) 、Shift + F11(Windows/Linux)，显示所有的书签列表，并且是可以搜索的。
 + 上一个/下一个书签：无，可以在设置中设置快捷键。

- 更多：当你为某个书签指定了标记，你可以使用快捷键 Ctrl + 标记 来快速跳转到标记处，比如输入Ctrl + 1，跳到标记为1的书签处。

### 2.折叠/展开代码块

- 描述：该操作提供一种方法，让你隐藏你不关心的部分代码，以一种较为简洁的格式显示关键代码。一个有意思的用法是隐藏匿名内部类的代码，让其看起来像一个Lambda表达式。
- 快捷键：Cmd + “+”/”-“(OS X)、Ctrl + Shift + “+”/”-“(Windows/Linux);
- 更多：可以在Settig → Editor → General → Code Folding 中设置折叠规则。

### 3.与分支对比

- 描述：该操作提供一种方法，让你隐藏你不关心的部分代码，以一种较为简洁的格式显示关键代码。一个有意思的用法是隐藏匿名内部类的代码，让其看起来像一个Lambda表达式。
- 快捷键：Cmd + “+”/”-“(OS X)、Ctrl + Shift + “+”/”-“(Windows/Linux);
- 更多：可以在Settig → Editor → General → Code Folding 中设置折叠规则。

### 4.与剪切板比对

- 描述：将当前选中的部分与剪切板上的内容进行比对。
- 调用：右键选中的部分，在右键菜单中选择“Compare With Clipboard”。

### 5.上下文信息

- 描述：当前作用域定义超过滚动区域，执行该操作将显示所在的上下文信息，通常它显示的是类名或者内部类类名或者当前所在的方法名。该操作在xml文件中同样适用。
- 调用：Menu → View → Context Info
- 快捷键：Alt + Q (Windows/Linux)
更多：个人认为，这个功能更好的用法是快速查看当前类继承的父类或者实现的接口。

### 6.查找操作

- 描述：输入某个操作的名称，快速查找，对于没有快捷键的部分操作这是一个很有用的技巧。
- 快捷键：Cmd +Shift + A(OS X)、Ctrl + Shift + A(Windows/Linux)；
- 更多：当某个操作是有快捷键的，会显示在旁边。

### 7.查找补全

- 描述：当你在一个文件中进行查找时，使用自动补全快捷键可以给出在当前文件中出现的建议单词；
- 快捷键：Cmd + F(OS X),Ctrl + F(Windows/Linux),输入一些字符，然后使用自动补全；

### 8.隐藏所有面板

- 描述：切换编辑器铺满整个程序界面，隐藏其他的面板。再次执行该操作，将会回到隐藏前的状态。
- 调用：Menu → Window → Active Tool Window → Hide All Windows；
- 快捷键：Cmd +Shift + F12(OS X)、Ctrl + Shift + F12(Windows/Linux)；

### 9.高亮一切

- 描述：该操作将会高亮某个字符在当前文件中所有出现的地方。这不仅仅是简单的匹配，实际上它会分析当前的作用域，只高亮相关的部分。
- 调用：Menu → Edit → Find → Highlight Usages in File；
- 定位到上一处/下一处：Menu → Edit → Find → Find Next/Previous；
- 快捷键：相关快捷键请在菜单中查看；
- 更多：
 + 如果高亮一个方法的return或throw语句，将会高亮这个方法的所有出口/结束点；
 + 如果高亮某个类定义处的extend或implements语句，将会高亮继承的或实现的方法；
 + 高亮一个import语句也会高亮使用到的地方；
 + 按下Esc可以退出高亮模式

 ### 10.回到上一个工具窗口
 
 - 描述：有时候你会从某个工具窗口跳到编辑器里面，然后又需要重新回到刚才操作的那个工具窗，比如你查找使用情况的时，使用该操作可以在不使用鼠标的情况下跳转到之前的工具窗口。
 - 快捷键：F12；
