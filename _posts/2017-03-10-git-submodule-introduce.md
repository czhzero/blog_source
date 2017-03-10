---
title: git submodule 使用命令简介
date: 2017-03-10 11:10:48
categories: 开发工具
tags: submodule
---

抽取项目公共模块，多项目共用，必然会使用到git submodule命令。
项目中submodule的管理，无外乎添加，更新，删除等操作。

<!-- more -->

###1. submodule 添加

进入到git项目根目录下，输入命令:

> git submodule add  [-b master ]  [URL to Git repo]  [local path]

其中，[-b master] 为submodule的默认分支，[local path]为submodule的本地路径。

若出现如下提示，则表示submodule添加成功。

![这里写图片描述](http://img.blog.csdn.net/20160121154730707)

命令执行完成，会在当前工程根路径下生成一个名为“.gitmodules”的文件，其中记录了子模块的信息，如下：

```
[submodule "libraries/pushserver"]
 path = libraries/pushserver                //本地路径
 url = git@*****/android-library-push.git   //仓库地址
 branch = master                            //默认分支
```

### 2. submodule 更新
 
 若项目中包含.gitmodules, 进入到git项目根目录下，输入命令, .gitmodules中的所有项目都会进行更新:

> git clone 父项目.git 
> git submodule init
> git submodule update  (update时，submodule分支必须已在正确分支上)

submodule远程分支发生变更后，直接使用git submodule update是不会进行更新操作的。必须依次进入到各个submodule的目录，进行git pull操作，如果submodule数目很多，每次发版本时必须进入所有目录进行git pull，这将是噩梦。不过有个更简单的方法，

> git submodule foreach git checkout master
> git submodule foreach git pull


### 3. submodule 删除

删除submodule会麻烦些，仅仅删除submodule模块内容，是无法彻底从git中删除掉。还需要到git相关配置文件中删除条目。

1. 删除 [项目根目录/.gitmodules] 中对应的条目
2. 删除 [项目根目录/.git/config] 中对应的条目
3. 删除 [项目根目录/.git/modules] 目录下的对应的submodule文件夹
4. 执行 git rm --cached [modulename] 命令
4. 删除 submodule 模块内容
 