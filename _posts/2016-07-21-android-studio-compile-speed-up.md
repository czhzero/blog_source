---
title: Android Studio提升编译速度方法总结
date: 2016-07-21 14:57:52
categories: Android
tags: Android Studio
---

编译会占用你的时间，加快编译会影响你的开发效率，让你的项目工作更加顺畅。本文会定期更新一些提高编译效率的方法。加速gradle编译。

<!-- more -->

### 方法一:开启gradle守护线程，并行编译

在下面的目录下面创建gradle.properties文件：

- /Users/<username>/.gradle/ (Mac)
- /home/<username>/.gradle/ (Linux)
- C:\Users\<username>\.gradle (Windows)

在文件中增加:

```
org.gradle.daemon=true  //就是让你让你编译时使用守护进程。

org.gradle.parallel=true //使用并行编译

org.gradle.jvmargs=-Xmx2048m -XX:MaxPermSize=512m //JVM最大允许分配的堆内存，按需分配

```

***网上关于提速的方法，这类介绍最多，不知道怎么回事，我亲测后，发现没什么效果。***


### 方法二:设置离线状态

![离线状态](http://o7y1sf21i.bkt.clouddn.com/blog/5/lALOYktUms0CEM0FeQ_1401_528.png)

***设置离线，Vpn用上后，似乎对整个编译也没有什么直观的改善。***


### 方法三:更新gradle版本

gradle是一个比较复杂的‘怪物’，大多数的项目随着每个release版本越来越快，所以用最新的版本有很大意义。[下载地址](https://gradle.org/gradle-download/)

Tips:
如果发现IDE下载gradle较慢,则可改为自定义下载，下载好的zip包放到如下目录

```
~/.gradle/wrapper/dists/gradle-2.10-all/5b6kqrscumh9p4evkkemdk57ad/
```

***我升级到gradle-2.14.1之后，也没发现有多大的改变。编译还是要1分多钟。***

dev包中设置minSdkVersion为21


### 方法四:优化项目

- 删除无用的module, 多module相互依赖会降低编译速度
- 删除无用的文件资源, 如单元测试代码，废弃图片等等
- 减少方法数目，不使用multidex。[multidex](http://blog.csdn.net/t12x3456/article/details/40837287)


***这类方法，肯定是会提高编译速度，本文不再详述。***



***这个方法肯定有用。***

### 方法五:使用aar依赖

我们都知道我们或多或少使用第三方的开源库或者工具，还有自己封装的类库，但是一次编译的时候在Library Module 目录下， 打开build/outputs/aar，就有生成的aar 文件。把他放在libs 目录下，在build.gradle 添加。

```
allprojects {
   repositories {
      jcenter()
      flatDir {
        dirs 'libs'
      }
   }
}

dependencies {
    compile(name:'test', ext:'aar')
}

```

当然也可以这样添加

我们可以新建一个jar/aar Module，选择aar 文件，然后新建的Module 目录下，就会多了个build.gradle 和xxx.aar。

```
configurations.maybeCreate("default")
artifacts.add("default", file('mylibrary-debug.aar'))

```

然后在我们的Module 中这样引用即可

```
compile project(':mylibrary-debug')

```

***如果大量的module都被置换成aar文件，确实会使得整个项目编译速度加快。但是若aar中代码需要改变，就会比较麻烦。所以这种方法，需要酌情考虑。***


### 方法六:对第三方库进行优化

- 利用debugCompile来依赖debug时才用到的库
- 利用更小的库替代现有的库
- 利用exclude来排出某些不需要的依赖

对于远程仓库

```
compile ('com.facebook.react:react-native:+'){
    exclude group: 'com.squareup.okhttp3', module: 'okhttp'
    exclude group: 'com.android.support', module: 'support-v4'
    exclude group: 'com.android.support', module: 'support-v7'
  }

```

对于本地项目

```
compile(project(':react-native-custom-module')) {
    exclude group: 'com.facebook.react', module: 'react-native'
}
```

### 方法七:采用增量编译

- Instant Run
- Jrebel

正在研究中。



### 参考文献

- [Speeding up Gradle builds](http://kevinpelgrims.com/blog/2015/06/11/speeding-up-your-gradle-builds/)
- [6个技巧加速你的gradle编译](http://www.jianshu.com/p/2ff3717199da)
- [知道Android 中Gradle 的这些技巧，提升编译构建速度](http://tikitoo.github.io/2016/05/26/android-studio-gradle-build-run-faster/)
- [Android打包提速实践](http://www.jianshu.com/p/e456a5ac8613?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)
