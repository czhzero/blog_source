---
title: Hexo加载vendor目录下js文件失败导致博客页面加载空白
date: 2016-11-16 19:14:36
categories: Hexo	
tags: javascript
---

今天下午用hexo框架deploy一篇文章，发布完成后，发现网页无法打开了。出现了很多404错误。


各种vender目录下的js加载都是失败的。本地部署是没有问题的，一旦上传到托管平台github或者coding后，就无法找到这些js文件。折腾了半天，网上搜资料才知道原来我的代码托管平台把vender目录禁止了。

<!-- more -->

![](http://o7y1sf21i.bkt.clouddn.com/blog/014/jserror.png)

网上有两种解决方案:

- 手动修改

```
首先修改source/vendors为source/lib，然后修改_config.yml，将 _internal: vendors修改为_internal:lib，然后修改next底下所有引用source/vendors路径为source/lib。
这些地方可以通过文件查找找出来。主要集中在这几个文件中。
1. Hexo\themes\next.bowerrc 
2. Hexo\themes\next.gitignore 
3. Hexo\themes\next.javascript_ignore 
4. Hexo\themes\next\bower.json 。

```
- 更新next主题

next新主题代码，已经将vender目录改为了lib目录，不过更新之前，要对自己的私人配置进行保存。



参考文章: [hexo+Github Page搭建的博客无法加载样式表？](https://www.zhihu.com/question/52272170/answer/129909383)
