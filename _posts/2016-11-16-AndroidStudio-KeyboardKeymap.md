---
title: Android Studio常用快捷键汇总
date: 2016-11-16 14:24:28
categories: Android
tags: Keymaps
---


Android Studio 是基于 IntelliJ IDEA 的官方 Android 应用开发集成开发环境 (IDE)。本篇总结了部分Android Studio常用的快捷键。

<!-- more -->


## 代码


- 普通补全代码

使用快捷键 `Control+Space` 。显示对变量、类型、方法和表达式等的基本建议。 如果连续两次调用基本自动完成，将显示更多结果，包括私有成员和非导入静态成员。

- 智能代码补全

使用快捷键 `Control+Shift+空格` 。根据上下文显示相关选项。 智能自动完成可识别预期类型和数据流。 如果连续两次调用智能自动完成，将显示更多结果，包括链。

- 语句自动完成

使用快捷键 `Shift+Command+Enter` 。 自动完成当前语句，添加缺失的圆括号、大括号、花括号和格式化等。

- 执行代码快速修复并显示建议的操作，用来补全代码

使用快捷键 `Alt+Enter` 可以进行快速修复，补全代码。 例如:

```

//光标移动到OnClickListener处，执行快捷键补全。
View.OnClickListener() {
                
            }

```

- 自动代码生成

使用 `Command + N` 自动生成一些代码，如getters, setters, constructors, hashCode/equals, toString, override 等， 同时也包括一些自定义插件，如GsonFormat,Parcelable等

- Override methods

`Control + O`

- Implement methods

`Control + I`

- 删除光标所在行的全部内容

`Command + Backspace`

- 展开或者关闭当前代码块

`Command + minus/plus`

- 展开或者关闭所有代码块

`Command + Shift + minus/plus`

- 复制选中的行内容

`Command + D`

- 跳转到代码声明处

`Command + B or Command + Click`

- 跳转到代码实现处

`Command + Alt + B`

- 跳转到超类，或者父方法。

`Command + U`


## 编译运行

- Build

`Command + F9`

- Build and run

`Control + R`

## 代码重构

- Extract method	

`Command + Option + M`

- Extract variable

`Command + Option + V`

- Extract field

`Command + Option + F`

- Extract constant

`Command + Option + C`

- Extract parameter

`Command + Option + P`

## 导航

- 查看最近访问文件

使用最近文件操作在最近访问的文件之间切换。按 Control+E（在 Mac 上，按 Command+E）调出“最近文件”操作。 默认情况下将选择最后一次访问的文件。 在此操作中您还可以通过左侧列访问任何工具窗口。

- 查看当前文件的结构，快速定位具体代码

使用文件结构操作查看当前文件的结构。 按 Control+F12（在 Mac 上，按 Command+F12）调出“文件结构”操作。您可以使用此操作快速导航至当前文件的任何部分。

- 在项目中搜索指定类

使用导航至类操作搜索并导航至项目中的特定类。 按 Control+N（在 Mac 上，按 Command+O）调出此操作。 “导航至类”支持复杂的表达式，包括驼峰、路径、直线导航和中间名匹配等。 如果连续两次调用此操作，将显示项目类以外的结果。

- 在项目中搜索指定的文件或者文件夹

使用导航至文件操作导航至文件或文件夹。 按 Control+Shift+N（在 Mac 上，按 Command+Shift+O）调出“导航至文件”操作。要搜索文件夹，但不搜索文件，请在表达式末尾添加“/”。

- Find Usages

按 Alt+F7 查找引用当前光标位置处的类、方法、字段、参数或语句的所有代码片段。

- 项目搜索

使用 `Command + F` 搜当前文件的文本内容， 使用 `Command + Shift + F` 搜索整个项目。

- 项目搜索并替换

使用 `Command + R` 搜当前文件并替换， 使用 `Command + Shift + R` 搜索并替换整个项目。

- Jump to source	

使用 `Command + 鼠标按下` 跳转到代码部分


- Search everything

搜索所有东西，连续按两次Shift。搜索全项目所有文件。

- 跳转到上次更改的代码位置

`Command + Shift + Backspace` 跳转到上次更改的位置，不用拖动鼠标在找到之前的位置。

- 跳转到指定代码行

`Command + L`

- Open type hierarchy

`Control + H`
 
- Open method hierarchy

`Command + Shift + H`
 
- Open call hierarchy

`Command + Shift + H`


## 样式和格式化

在您编辑时，Android Studio 将自动应用代码样式设置中指定的格式设置和样式。您可以通过编程语言自定义代码样式设置，包括指定选项卡和缩进、空格、换行、花括号以及空白行的约定。要自定义代码样式设置，请点击 File > Settings > Editor > Code Style（在 Mac 上，点击 Android Studio > Preferences > Editor > Code Style）。

虽然 IDE 会在您工作时自动应用格式化，但您也可以通过按 Control+Alt+L（在 Mac 上，按 Opt+Command+L）显式调用重新格式化代码操作，或按 Control+Alt+I（在 Mac 上，按 Alt+Option+I*）自动缩进所有行。



## 版本控制基础知识

Android Studio 支持多个版本控制系统 (VCS)，包括 Git、GitHub、CVS、Mercurial、Subversion 和 Google Cloud Source Repositories。

在将您的应用导入 Android Studio 后，使用 Android Studio VCS 菜单选项启用对所需版本控制系统的 VCS 支持、创建存储库、导入新文件至版本控制以及执行其他版本控制操作。

- 在 Android Studio VCS 菜单中点击 Enable Version Control Integration。
- 从下拉菜单中选择要与项目根目录关联的版本控制系统，然后点击 OK。

此时，VCS 菜单将根据您选择的系统显示多个版本控制选项。

> 注： 您还可以使用 File > Settings > Version Control 菜单选项设置和修改版本控制设置。


更多快捷键相关的请参考:

[Keyboard Shortcuts](https://developer.android.com/studio/intro/keyboard-shortcuts.html)