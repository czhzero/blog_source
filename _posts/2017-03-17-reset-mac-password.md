---
title: 一招破解Mac系统登录密码
date: 2017-03-17 10:07:57
categories: Mac
tags: 忘记密码
---

Mac系统忘记密码不用担心，几行命令帮你搞定密码，让你重新登录进入系统。

<!-- more -->


1: 开机，启动时候按command+s
2: 进入Single User Mode, root用户下的命令行界面

3: 在命令行界面，依次输入如下命令:

```
fsck  -y

mount  -uaw /

rm /var/db/.AppleSetupDone

reboot

```

开机后是全新的欢迎界面，就是第一次使用时候设置的状态。不需要担心。重新进去管理员，里面会有原来的账号，切换到原来的账号，一切都还在。
如果是老用户，不需要输入原来密码就可以输入新的密码更改，前提是把小锁给点下，权限去除。