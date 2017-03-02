---
title: HashMap vs. LinkedHashMap vs. TreeMap vs. HashTable 四种容器对比
date: 2017-03-02 21:03:11
categories: Java
tags: 容器
---


Java为数据结构中的映射定义了一个接口`java.util.Map`,它有四个实现类,分别是HashMap Hashtable LinkedHashMap 和TreeMap。

<!-- more -->

## HashMap

根据键的HashCode 值存储数据,根据键可以直接获取它的值，具有很快的访问速度。

![](https://camo.githubusercontent.com/8565fd90daccb7f62375f2164a5eaf5b88b6e2b1/687474703a2f2f696d672e626c6f672e6373646e2e6e65742f32303134303730313139313430333736343f77617465726d61726b2f322f746578742f6148523063446f764c324a736232637559334e6b626935755a585176626e4e665932396b5a513d3d2f666f6e742f3561364c354c32542f666f6e7473697a652f3430302f66696c6c2f49304a42516b46434d413d3d2f646973736f6c76652f37302f677261766974792f536f75746845617374)


- 遍历时，取得数据的顺序是完全随机的
- 不支持线程的同步
- 最多只允许一条记录的键为Null
- 允许多条记录的值为Null
- 在插入、删除和定位元素，速度较快。

更多内容请查看[HashMap源码剖析](https://github.com/GeniusVJR/LearningNotes/blob/master/Part2/JavaSE/HashMap%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90.md)

## LinkedHashMap

LinkedHashMap是HashMap的子类，与HashMap有着同样的存储结构，但它加入了一个双向链表的头结点，将所有put到LinkedHashmap的节点一一串成了一个双向循环链表，因此它保留了节点插入的顺序，可以使节点的输出顺序与输入顺序相同。

LinkedHashMap可以用来实现LRU算法。

LinkedHashMap同样是非线程安全的，只在单线程环境下使用。
在遍历的时候会比HashMap慢，不过有种情况例外，当HashMap容量很大，实际数据较少时，遍历起来可能会 比LinkedHashMap慢，因为LinkedHashMap的遍历速度只和实际数据有关，和容量无关。

更多内容请查看[LinkedHashMap源码剖析](https://github.com/GeniusVJR/LearningNotes/blob/master/Part2/JavaSE/LinkedHashMap%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90.md)

## TreeMap

TreeMap实现SortMap接口，能够把它保存的记录根据键排序,默认是按键值的升序排序，也可以指定排序的比较器，当用Iterator 遍历TreeMap时，得到的记录是排过序的。

TreeMap取出来的是排序后的键值对。但如果您要按自然顺序或自定义顺序遍历键，那么TreeMap会更好。

## HashTable

HashMap和Hashtable都实现了Map接口，但决定用哪一个之前先要弄清楚它们之间的分别。主要的区别有：线程安全性，同步(synchronization)，以及速度。Hashtable的方法是同步的，而HashMap的方法不是。所以有人一般都建议如果是涉及到多线程同步时采用HashTable，没有涉及就采用HashMap。


更多内容请查看[HashTable源码剖析](https://github.com/GeniusVJR/LearningNotes/blob/master/Part2/JavaSE/HashTable%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90.md)

