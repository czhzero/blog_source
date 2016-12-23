---
title: Java中String数据类型转换与format总结
date: 2016-12-23 10:10:48
categories: Java
tags: formatter
---

Java中的String类型与List,数组等格式在日常工作中，经常需要进行相互转换，本文总结了几种常见的格式转换与format例子。

<!-- more -->

## 1. List<String> 与 String数组

- List<String>转String数组

```
ArrayList list = new ArrayList();
list.add("aaa");
list.add("bbb");
String[] arrString = (String[]) list.toArray(new String[list.size()]);

```


- String数组转List<String>

```
String[] s = {"1", "2", "3", "5", "6"};
List<String> listA = Arrays.asList(s);
listA.add(3, "4");     //java.lang.UnsupportedOperationException异常

ArrayList<String> listB = new ArrayList<>(listA);
listB.add(3, "4");       //正确

```

> Arrays.asList() 返回的是 `java.util.Arrays.ArrayList` 而不是 `java.util.ArrayList` , 
> 这两个类同名，都继承了AbstractList，
> 但是前者并未实现add,remove等方法，所有要进行增删操作，必须先转换成真正的 `java.util.ArrayList`,
> 否则会报`java.lang.UnsupportedOperationException异常`


## 2. String[]数组 与 String

- String[]数组转String

```
String[] s = {"1", "2", "3", "4"};
String str = Arrays.toString(s);
```


- String转String[]数组

```
String str = "1,2,3,4";
String[] s = str.split(",");

```

## 3. List<String> 与 String

- List<String>转String

```
String str = listA.toString();

```

- String转List<String>

String先转换成String数组，再转成List<String>。


## 4.String 与 数值类型互转

- String转int

```
int i = Integer.parseInt("1");
int j = Integer.valueOf("2").intValue();
int k = Integer.valueOf("3");

```

> 字串转成 Double, Float, Long 的方法大同小异。

- int转String


```
String a = String.valueOf(1);
String b = Integer.toString(2);
String c = "" + 3;

```

> Double, Float, Long 转成字串的方法大同小异。


## 5. String.format格式化

- 整数格式format

```
int d = Integer.parseInt("99099");  
System.out.println(d);  
//格式化为整形型字符串  
System.out.println(String.format("%d",d));  
//整数长度为7，如果不到7位就用0填充  
System.out.println(String.format("%07d",9909));  
System.out.println(String.format("%07d",99099999));  
//长度不满7就用空格填充  
System.out.println(String.format("% 7d",9909));  
//使用,对数字分组  
System.out.println(String.format("%,d",9909999));  
//显示正负数  
System.out.println(String.format("%+d",d));  
System.out.println(String.format("%+d",-345));  

```

- 小数点格式format

```
double d = Double.parseDouble("9909999999.9999");  
System.out.println(d);  
//格式化为浮点型字符串  
System.out.println(String.format("%f",d));  
//整数部分全部显示，小数部分后面保留5位小数  
System.out.println(String.format("%.5f",d));  
//使用,对数字分组  
System.out.println(String.format("%,f",d));  
//显示正负数  
System.out.println(String.format("%+f",d));  
System.out.println(String.format("%+f",-345.98));  
//算小数点后面的位数一起时15  
System.out.println(String.format("%015f",345.98));  
//小数点后面保留三位小数  
System.out.println(String.format("%015.3f",345.98));  

```

除了用`String.format`之外还有些其他方法。

```
//方法一

java.text.DecimalFormat df = new java.text.DecimalFormat("#.##");
double d = 3.14159;
System.out.println(df.format(d));

//方法二

BigDecimal bd = new BigDecimal("3.14159265");
bd = bd.setScale(2, BigDecimal.ROUND_HALF_UP);

//方法三

double value = 1234.5678;
long l1 = Math.round(value * 100);   //四舍五入
double result = l1 / 100.0;          //注意：使用100.0而不是100

//方法四

double value = 1234.5678;
value = ((int) (value * 100)) / 100.0;

```
 

- 时间格式format

```
Date now = new Date();  
System.out.println("全部日期和时间信息:"+String.format("%tc", now));  
System.out.println("年-月-日格式："+String.format("%tF", now));  
System.out.println("月/日/年格式:"+String.format("%tD", now));  
System.out.println("HH:MM :"+String.format("%tR", now));  
System.out.println("HH:MM:SS PM格式（12时制）："+String.format("%tr", now));  
System.out.println("HH:MM:SS格式（24时制）:"+String.format("%tT", now));  

```

## 参考文献

[String.format格式化](http://hbiao68.iteye.com/blog/1769053)
[Java中取小数点后两位(四种方法)](http://irobot.iteye.com/blog/285537)