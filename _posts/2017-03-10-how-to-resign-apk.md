---
title: Android如何重新签名APK
date: 2017-03-10 11:17:06
categories:
tags:
---

如何使用jarsigner进行apk的重新签名

<!-- more -->


## Step1.删除apk包中旧的签名文件
将apk重命名为zip文件，解压后，将其中META-INF文件夹删除，再改回.apk格式。

## Step2.进入到java安装目录，进入bin文件夹

## Step3.命令行生成keystore

keytool -genkey -alias nt.android -keyalg RSA -validity 20000 -keystorent.keystore
(-validity 20000代表有效期天数)，命令完成后，bin目录中会生成nt.keystore


## Step4.找到jarsigner文件,执行如下命令

```
jarsigner -verbose -keystore /Users/chenzhaohua/Downloads/sign/nt.keystore -storepass ntandroid -signedjar /Users/chenzhaohua/Downloads/sign/windthunder_signed.apk -digestalg SHA1 -sigalg MD5withRSA /Users/chenzhaohua/Downloads/sign/windthunder_unsigned.apk nt.android

```

其中，


- `/Users/chenzhaohua/Downloads/sign/nt.keystore`为签名证书文件。
- `ntandroid`为证书密码
- `/Users/chenzhaohua/Downloads/sign/windthunder_signed.apk` 为签名后的apk文件名
- `/Users/chenzhaohua/Downloads/sign/windthunder_unsigned.apk` 为待签名的apk
nt.android  为Key的别名