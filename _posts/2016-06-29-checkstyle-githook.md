---
title: Java代码规范之CheckStyle + Git Hook
date: 2016-06-29 11:18:37
categories: 开发工具
tags: [checkstyle, githook]
---



### CheckStyle简介

CheckStyle提供了一个帮助JAVA开发人员遵守某些编码规范的工具。它能够自动化代码规范检查过程，从而使得开发人员从这项重要，但是枯燥的任务中解脱出来。

<!-- more -->

CheckStyle检验的主要内容如下。

- Annotations
- Block Checks
- Class Design
- Coding
- Headers
- Imports
- Javadoc Comments
- Metrics
- Miscellaneous
- Modifiers
- Naming Conventions
- Regexp
- Size Violations
- Whitespace

官网给出了两个规则示例, [Google's Style](http://checkstyle.sourceforge.net/google_style.html) 和 [Sun's Style
](http://checkstyle.sourceforge.net/sun_style.html) 用以参考。

除了上述检验内容，CheckStyle还支持[自定义规则](http://checkstyle.sourceforge.net/writingchecks.html),用来定义自己想要的代码风格。


### Git Hook简介

和其它版本控制系统一样，Git 能在特定的重要动作发生时触发自定义脚本。 有两组这样的钩子：客户端的和服务器端的。 客户端钩子由诸如提交和合并这样的操作所调用，而服务器端钩子作用于诸如接收被推送的提交这样的联网操作。 你可以随心所欲地运用这些钩子。

钩子都被存储在 Git 目录下的 hooks 子目录中。 也即绝大部分项目中的 .git/hooks 。 当你用 git init 初始化一个新版本库时，Git 默认会在这个目录中放置一些示例脚本。这些脚本除了本身可以被调用外，它们还透露了被触发时所传入的参数。 所有的示例都是 shell 脚本，其中一些还混杂了 Perl 代码，不过，任何正确命名的可执行脚本都可以正常使用 —— 你可以用 Ruby 或 Python，或其它语言编写它们。 这些示例的名字都是以 .sample 结尾，如果你想启用它们，得先移除这个后缀。

把一个正确命名且可执行的文件放入 Git 目录下的 hooks 子目录中，即可激活该钩子脚本。 这样一来，它就能被 Git 调用。

客户端钩子分为很多种。代码规范检查我们使用pre-commit钩子即可，pre-commit 钩子在键入提交信息前运行。 它用于检查即将提交的快照，例如，检查是否有所遗漏，确保测试运行，以及核查代码。 如果该钩子以非零值退出，Git 将放弃此次提交，不过你可以用 git commit --no-verify 来绕过这个环节。 你可以利用该钩子，来检查代码风格是否一致（运行类似 lint 的程序）、尾随空白字符是否存在（自带的钩子就是这么做的），或新方法的文档是否适当。

更多关于Git Hook内容的请看 [这里](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90)。


### 准备工作

GitHook的pre-commit脚本配合CheckStyle即可完成代码规范的自动检查。每次代码commit之前，若代码不符合规范，则无法commit成功。 

要执行CheckStyle需要以下几个条件。

- jdk与git环境配置
- pre-commit脚本, 用来执行代码检测与结果分析
- checkstyle的jar包 , 例如 [checkstyle-7.0-all.jar](http://downloads.sourceforge.net/project/checkstyle/checkstyle/7.0/checkstyle-7.0-all.jar)
- 规则配置的xml文件, 例如 [sun_checks](https://raw.githubusercontent.com/checkstyle/checkstyle/master/src/main/resources/sun_checks.xml)。


### 自定义检查规则示例

官方提供的代码规范往往太过严格，在工作中使用不太现实，所以有必要根据具体情况来定制具体的代码规范，CheckStyle对代码规范的定制提供了很多大灵活性。
下面我们来定义一些基本的规范，后续有增加我们再修改。

```
<?xml version="1.0"?>
<!DOCTYPE module PUBLIC
    "-//Puppy Crawl//DTD Check Configuration 1.3//EN"
    "http://www.puppycrawl.com/dtds/configuration_1_3.dtd">


<module name = "Checker">
    <property name="charset" value="UTF-8"/>

    <property name="severity" value="warning"/>

    <property name="fileExtensions" value="java, properties, xml"/>

    <!-- 检查文件中是否含有tab键-->
    <module name="FileTabCharacter"/>


    <!-- 代码行数最大不超过1000行 -->
    <module name="FileLength">
        <property name="max" value="1500"/>
    </module>

    <module name="TreeWalker">


        <!-- 避免.*,重复多余的和不使用的import-->
        <module name="AvoidStarImport"/>
        <module name="RedundantImport"/>
        <module name="UnusedImports"/>


        <!-- 常量全部用大写-->
        <module name="ConstantName"/>

        <!-- 方法名称 -->
        <module name="MethodName"/>



        <!-- 检查代码块:起始大括号和if等同行，不能有空的代码块，结束大括号另起一行-->
        <module name="LeftCurly"/>
        <module name="NeedBraces"/>
        <module name="RightCurly"/>

        <!-- 代码缩进格式 -->
        <module name="Indentation">
            <property name="basicOffset" value="4"/>
            <property name="braceAdjustment" value="0"/>
            <property name="caseIndent" value="4"/>
            <property name="throwsIndent" value="4"/>
            <property name="lineWrappingIndentation" value="4"/>
            <property name="arrayInitIndent" value="4"/>
        </module>

        <!-- 操作符周围要有空格 -->
        <module name="WhitespaceAround"/>

        <!-- 当有多重修饰符时,修饰符采用以下顺序:
             (public,protected,private,abstract,static,final,
             transient,volatile,synchronized,native,strictfp) -->
        <module name="ModifierOrder"/>

        <!-- 检查是否有多余的修饰符，例如：接口中的方法不必使用public、abstract修饰
                   tokens: 检查的类型 -->
        <module name="RedundantModifier" />


        <!-- 禁止空catch快 -->
        <module name="EmptyCatchBlock">
            <property name="exceptionVariableName" value="expected"/>
        </module>


        <!-- 代码最长长度不能超过100 -->
        <module name="LineLength">
            <property name="max" value="100"/>
            <property name="ignorePattern" value="^package.*|^import.*|a href|href|http://|https://|ftp://"/>
        </module>


        <!-- 最多方法数不超过30 -->
        <module name="MethodCount">
            <property name="maxTotal" value="30"/>
        </module>


        <!-- 方法长度不超过60 -->
        <module name="MethodLength">
            <property name="tokens" value="METHOD_DEF"/>
            <property name="max" value="60"/>
        </module>


        <!-- 避免Equals左边为空 -->
        <module name="EqualsAvoidNull"/>

        <!-- 避免复杂内联函数 -->
        <module name="AvoidInlineConditionals"/>

        <!--
        1. 类（静态）变量。首先应当是public类变量，然后是protected类变量，然后是package类变量（没有访问标识符），最后是private类变量。
        2. 实例变量。首先应当是public类变量，然后是protected类变量，然后是package类变量（没有访问标识符），最后是private类变量。
        3. 构造器
        4. 方法
        -->
        <module name="DeclarationOrder"/>


        <!-- 避免忘记break,continue等 -->
        <module name="FallThrough"/>

        <!-- for循环深度最多为2 -->
        <module name="NestedForDepth">
            <property name="max" value="1"/>
        </module>

        <!-- if else深度最多为3 -->
        <module name="NestedIfDepth">
            <property name="max" value="2"/>
        </module>

        <!-- try catch深度最多为3-->
        <module name="NestedTryDepth">
            <property name="max" value="2"/>
        </module>



    </module>


</module>

```






### Git Hook 脚本示例

```
#! /usr/bin/env python
# -*- encoding: utf-8 -*-

import sys,os,re

print '\n.......................Code Style Checking....................\n'

#the count of level, like ERROR,WARN
def get_level(content, level):
    return len(re.compile(r"\[%s\]" % level).findall(content))

#get the commit file name (whole path)
def get_file_name(content, postfix=None):
    content = content.replace("\t", " ")
    line_divided = content.split("\n")
    space_divided = [[j for j in i.split(" ") if j.strip()]for i in line_divided]
    filenames = [i[5] for i in space_divided if i]
    if not postfix:
        return filenames
    return [i for i in filenames if ".%s" % postfix in i]

jarpath = os.popen('git config --get checkstyle.jar').read()
checkfilepath = os.popen('git config --get checkstyle.checkfile').read()

#check code command
command = 'java -jar ' + jarpath[:-1] + ' -c ' + checkfilepath[:-1]

#the file to check
files = os.popen('git diff-index --cached HEAD').read()

#the result of command
content = get_file_name(files, 'java')

resultsum = 0

for i in content:
    result = os.popen(command + ' ' + i).read()
    print result
    resultsum += get_level(result,'ERROR')
    resultsum += get_level(result,'WARN')

if resultsum > 0:
    print '\n.......................You must fix the errors and warnings first, then excute commit command again...........\n'
    sys.exit(-1)
else:
    print '\n.................................Code is very good...................\n'
    sys.exit(0)  
  

```

以上pre-commit脚本使用的是python语言, 主要做了三件事。

- 通过git diff命令找到本次提交的java文件的完整路径
- 循环执行check命令

> java -jar checkstyle-7.0-all.jar -c /sun_checks.xml MyClass.java

- 找出check命令结果中的ERROR和WARN数量, 返回0, commit成功, 否则,  commit失败


### 部署到项目中

因为pre-commit脚本,是存放在.git/hooks目录下的。所以他不会被提交到远程服务器。所以只能将
pre-commit脚本, checkstyle-7.0-all.jar, 以及checks.xml存放到项目目录下。

每个终端通过git pull拉取到文件后, 执行下列三个命令。

- git config --add checkstyle.jar ../../config/checkstyle-7.0-all.jar
- git config --add checkstyle.checkfile ../../config/test_checks.xml
- cp ../../config/pre-commit ./.git/hooks/ 

这样,再运行git commit的时候就会自动帮你检查你的代码了。


![Alt text](http://o7y1sf21i.bkt.clouddn.com/blog/002/1.png)


根据提示修改代码吧。

注: 若有submodule模块,pre-commit要拷贝到.git/modules/.../hooks目录下。


参考文献:

- [check style官网](http://checkstyle.sourceforge.net/)
- [git hook](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90)
- [详解CheckStyle的检查规则（共138条规则）](http://blog.csdn.net/yang1982_0907/article/details/18086693)

