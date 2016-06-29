---
title: Java代码规范
date: 2016-06-29 18:25:04
categories: Java
tags: Java
---

## 写在前面 

**为什么需要编码规范**

- 一个软件的生命周期中，80%的花费在于维护 
- 几乎没有任何一个软件，在其整个生命周期中，均由最初的开发人员来维护 
- 编码规范可以改善软件的可读性，可以让程序员尽快而彻底地理解新的代码 
- 如果你将源码作为产品发布，就需要确任它是否被很好的打包并且清晰无
- 误，一如你已构建的其它任何产品

<!-- more -->


## 命名规则

### 1.Java源文件的命名

JAVA源文件名必须和源文件中所定义的类的类名相同

### 2.Package的命名

Package名的第一部分应是小写ASCII字符，并且是顶级域名之一，通常是com、edu、gov、mil、net、org或由ISO标准3166、1981定义的国家唯一标志码。Package名的后续部分由各组织内部命名规则决定，内部命名规则指定了各组件的目录名，所属部门名、项目名等。

示例:

> package com.apple.quicktime.v2;

### 3.Class/Interface的命名

Class名应是首字母大写的名词。命名时应该使其简洁而又具有描述性。异常类的命名，应以Exception结尾。Interface的命名规则与Class相同。

示例:

```
public interface Set {
        …
};
public class CounterSet implement Set {
        …
};
public class InvalidException extends Exception {
        …
};

```

### 4.常量的命名

常量名的字母应全部大写，不同的单词之间通过下划线进行连接，并且名字组合应该赋予含义。

示例:

```
static final int MIN_WIDTH = 4;

```

### 5.变量的命名

普通变量名的首字母小写，其它每个单词的首字母大写。命名时应该使其简短而又有特定含义，简洁明了的向使用者展示其使用意图。

示例:

``` 
float floatWidth = 0.0;
float doubleWidth = 0.0;
```

### 6.方法的命名

方法名的第一个单词应是动词，并且首字母小写，其它每个单词首字母大写。

示例:

```
void findPersonID(int nID);
void isStringEmpty(String str);
```

### 7.方法参数的命名

应该选择有意义的名称作为方法的参数名。如果可能的话，选择和需要赋值的字段一样的名字。

示例：

```
void setCounter(int size) {
    this.size = size;
}
```


## 样式结构

### 1.缩进与对齐

一个缩进单位为四个空格，缩进排版时以缩进一个单位为最小缩进量。不要使用制表符（Tab键），因为不同的系统对它的解释不尽相同。

- 缩进

当某行语句在逻辑上比下面的语句高一个层次时，该行下面的语句都要在该行的基础上缩进一个单位。

示例：

```

public void someMethod(parameterA,parameterB) {
        int variantA=0;

        Sentence1;
        if (Conditions) {
            Sentence2;
        }
}

```

### 2.行宽

为了和linux,unix等字符界面的操作系统兼容，JAVA代码行应限制在100个字符之内，多余部分应换行。

示例:

```
variantA = someMethod(longExpression1, longExpression2, longExpression3);
    应改为：
variantA = someMethod(longExpression1, longExpression2, 
                          longExpression3);

```

### 3.断行规则

当一句完整的语句大于100个字符时需要断行，断行时 ，应遵循下面规则。

- 在逗号后换行

- 在操作符前换行

- 换行后，应和断行处的前一层次对齐

- 换行时尽量选择高层次的地方进行换行

- 在使用上述的规则换行后对齐时，如果次行的长度大于80个字符，应改用两个单位的缩进来代替层次对齐


### 5.空白的使用

#### 空格的使用

- 关键字和括号()之间要用空格隔开

```
 例：
    while (condition) {
        Sentence1;
    }
    if (condition) {
        Sentence2;
    }

```

- 参数列表中逗号的后面应该使用空格

```
例：
    public void methodA(parameterA, parameterB, parameterC) {
        Sentence1;
    }

```

- 所有的二元运算符，除了"."，应该使用空格将之与操作数分开

```
例：
    longName1 = longName2 * (longName3 + longName4) + 4 * longName5;

```

- 强制类型转换后应该跟一个空格

```
  例：
    methodA((byte) parameterA, (Object) parameterB);

```

- 左括号右边和右括号左边不能有空格

```
例：
    longName1 = longName2 * ( longName3 + longName4 );
    应改为：
    longName1 = longName2 * (longName3 + longName4);

```

- 方法名与其参数列表的左括号之间不能有空格

```
例：
    methodA (parameter1, parameter2);
    应改为：
    methodA(parameter1, parameter2);

```

- 一元操作符和操作数之间不应该加空格，比如：负号("-")、自增("++")和自减("--")

```
variantA += variantB --;
    应改为：
variantA += variantB--;

```

#### 空白行的使用

空白行将逻辑相关的代码段分隔开，以提高可读性，有如下几种情形

 
- 两个类声明或接口声明之间使用两个空白行

```
例：
    public class A {
        …
    }

    private class B {
        …
    }

```

- 两个方法的声明之间使用一个空白行

```
例：
    public class A {
        private void methodA() {
            …
        }

        priavte void methodB() {
            …
        }
    }

```

## 语句

### 1.简单语句

- 每行至多包含一条完整语句

```
例：
    variantA++; variantB++;
    应改为：
    variantA++;
    variantB++;

```

- 在没有必要的情况下，不要在return语句中使用括号

```
例：
    return (0);
    应改成：
    return 0;

```

### 2.复合语句

复合语句是包含在大括号中的语句序列，形如"{ 语句 }"，其编码应有如下基本规则：
- 被括其中的语句应该比复合语句缩进一个层次
- 左大括号"{"应位于复合语句起始行的行尾，并且空一个空格，右大括号"}"应另起一行并与复合语句首行对齐
- 复合语句即使只有一个语句，也要有大括号作为界定
- 每行至多包含一条完整语句

#### 判断语句

```
例：
    if-else语句应该具有如下格式：
    if (condition) {
        Sentences;
    }

    if (condition) {
        Sentences;
    } else if (condition) {
        Sentences;
    } else {
        Sentences;
    }

```

#### 选择语句

在选择语句中应添加 default情况，防止不可预知的情况发生。当一个case在没有break语句的情况下，它将顺着往下执行。应在break语句的位置添加注释.

```
例：
    switch (condition) {
    case ABC:
        Sentences;
        
    case DEF:
        Sentences;
        break;

    case XYZ:
        Sentences;
        break;

    default:
        Sentences;
        break;
    }


```

#### 循环语句

在for语句的初始化或更新子句中，如果存在多项，各项间应用逗号隔开。同时，应避免使用三个以上子句，从而导致复杂度提高；若确实需要，可以在for循环之前放置初始化子句或在for循环末尾放置更新子句。

```
例：
    for (int i = 0, j = 10, k = 10, m = 50; i < j + k + m; i++, j--, k--, m--) {
        Sentences;
    }
    应改为：
    int i = 0;
    int j = 100;
    int k = 1000;
    int m = 500;

    for (; i < j + k + m ;) {
        Sentences;
        i++;
        j--;
        k--;
        m--;
    }

```

## 声明

### 1.变量的声明

- 一行只声明一个变量

```
例：
    int variantA = 0, variantB = 0;
    应改为：
    int variantA = 0;
    int variantB = 0;

```

- 临时变量放在其作用域内声明

```
例：
    int tempA = 0;

    if (condition) {
        tempA = methodA();
        methodB(tempA);
    }
    应改为：
    if (condition) {
        int tempA = 0;

        tempA = methodA();
        methodB(tempA);
    }

```

- 声明应集中放在作用域的顶端

```
例：
    if (condition) {
        int tempA = 0;

        tempA = methodA();
        int tempB = 0;

        tempB = methodB();
    }
    应改为：
    if (condition) {
        int tempA = 0;
        int tempB = 0;

        tempA = methodA();
        tempB = methodB();
    }

```

- 临时变量放在其作用域内声明

```
例：
    int tempA = 0;

    if (condition) {
        tempA = methodA();
        methodB(tempA);
    }
    应改为：
    if (condition) {
        int tempA = 0;

        tempA = methodA();
        methodB(tempA);
    }

```

- 避免声明的局部变量覆盖上一级声明的变量

```
例：
    int counter = 0;

    if (condition) {
        int counter = 0;

        counter = methodA();
    }
    应改为：
    int counter = 0;

    if (condition) {
        int counterTemp = 0;

        counterTemp = methodA();
    }


```

### 2.类和接口的声明

当编写类和接口时，应该遵守以下规则：

- 在方法名与其参数列表之前的左括号"("间不要有空格
- 左大括号"{"位于声明语句同行的末尾，并与末尾之间留有一个空格
- 右大括号"}"另起一行，与相应的声明语句对齐。如果是一个空语句，"}"应紧跟在"{"之后
- 方法与方法之间以空白行分隔

```
public class Sample extends Object {
        private int ivar1;
        private int ivar2;

        public Sample(int i, int j) {
            ivar1 = i;
            ivar2 = j;
        }

        public int emptyMethod() {}
        …
}

```



