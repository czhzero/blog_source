---
title: 单个电脑上进行多ssh-key配置
date: 2017-03-10 10:39:10
categories: 开发工具
tags: ssh-key
---


ssh是一种网络协议，用于计算机之间的加密登录。由于公司使用的是git进行代码管理，而在公司电脑上有时也需要访问github。所以，在同一台机器上配置两个ssh-key很有必要。我用的mac机器，大部分东西都有了，配置起来还是比较简单的。


<!-- more -->

###1.生成公钥/私钥
	
命令行输入下面一行命令，

```
ssh-keygen -t rsa -C "your@email.com" -f ~/.ssh/id_rsa
```
其中，-f 参数表示生成公钥／私钥的路径，不设置，代表默认路径。

为配置多个ssh-key必须设置路径，否则会相互覆盖。

```
//key-1
ssh-keygen -t rsa -C "first@email.com" -f ~/.ssh/id_rsa
//key-2
ssh-keygen -t rsa -C "second@email.com" -f ~/.ssh/id_rsa_new

```
.ssh文件下会生成四个文件。

```
id_rsa             //第一个私钥

id_rsa.pub         //第一个公钥

id_rsa_new         //第二个私钥

id_rsa_new.pub     //第二个公钥

```

###2.git服务器设置公钥

(1)命令行拷贝公钥内容

```
pbcopy < ~/.ssh/id_rsa.pub
```

(2)登录git服务器，配置公钥。

以github为例,进入到Personal settings－SSH Keys - Add Key, 粘贴公钥内容即可。

![这里写图片描述](http://img.blog.csdn.net/20160105232245917)

![这里写图片描述](http://img.blog.csdn.net/20160105232418300)

###3.设置私钥代理

```
ssh-add ~/.ssh/id_rsa
ssh-add ~/.ssh/id_rsa_new
```
如果执行ssh-add时提示"Could not open a connection to your authentication agent"，可以先执行命令

```
$ ssh-agent bash
```
再执行ssh-add命令。执行完成后，可通过下面两个命令进行修改。

```
#显示私钥列表
$ ssh-add -l   

#清空私钥列表
$ ssh-add -D  
```

###4.增加/修改.ssh/config配置文件

将config文件修改成如下样式。

```
#自建服务器
Host gitlab
HostName your.company.com
Port 5555
User test1
IdentityFile ~/.ssh/id_rsa_new

#github服务器
Host github
HostName github.com
User test2
IdentityFile ~/.ssh/id_rsa

```

###5.验证结果

```
$ ssh -T git@github.com
```

若输出如下结果,

```
Hi ****! You've successfully authenticated, but GitHub does not provide shell access.
```
则ssh-key配置成功。



