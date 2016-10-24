---
title: 快速分享Android Library项目到jcenter
date: 2016-10-24 15:05:33
categories: Android
tags: jcenter
---


如果你想在Android Studio中引入一个library到你的项目，你只需添加如下的一行代码到模块的build.gradle文件中

```
dependencies {
    compile 'org.chen.statusbar:statusbar:1.0.0'
}

```

<!-- more -->

上面的例子中，引用的library内容，包含了3部分信息

> GROUP_ID:ARTIFACT_ID:VERSION

其中，GROUP_ID是 `org.chen.statusbar` , ARTIFACT_ID 是 `statusbar`, VERSION是`1.0.0`。添加这行代码后，点击Android Studio的sync按钮，你就可以在项目的 `External Libraries` 目录下看到 `statusbar-1.0.0` 这个Library包，并引用其中的代码了。

![](http://o7y1sf21i.bkt.clouddn.com/blog/012/%E7%B2%98%E8%B4%B4%E5%9B%BE%E7%89%872.png)

Android Studio是从build.gradle里面定义的Maven 仓库服务器上下载library的。Apache Maven是Apache开发的一个工具，提供了用于贡献library的文件服务器。总的来说，只有两个标准的Android library文件服务器：jcenter 和  Maven Central。随着Android Studio升级换代，目前默认Android Studio默认配置，只有jcenter了。推荐使用jcenter。


```
allprojects {
    repositories {
        jcenter()
    }
}

```

那么如何上传到jcenter呢？

## 注册Bintray账号

Bintray是jcenter的托管商, Bintray是jcenter的托管商，因此你必须注册一个Bintray账号，注册完成后，进入账户管理界面，记录下username 和 api_key。

![账号密码](http://o7y1sf21i.bkt.clouddn.com/blog/012/%E7%B2%98%E8%B4%B4%E5%9B%BE%E7%89%871.png)


## 在Bintray网站上，创建package

- 点击maven

![](http://o7y1sf21i.bkt.clouddn.com/blog/012/%E7%B2%98%E8%B4%B4%E5%9B%BE%E7%89%874.png)

- Add new package
![](http://o7y1sf21i.bkt.clouddn.com/blog/012/%E7%B2%98%E8%B4%B4%E5%9B%BE%E7%89%875.png)

- Create Package
![](http://o7y1sf21i.bkt.clouddn.com/blog/012/%E7%B2%98%E8%B4%B4%E5%9B%BE%E7%89%873.png)

注意这里的Version control是必填的，所以在上传类库到bintray之前最好是把项目push到github上，这样就会很方便了。


## 创建Android Studio项目

创建一个Android Studio项目， 并新建一个module模块， 如下图所示，我创建了一个名为statusbar的库, 用于上传。
这个library会被自动打包成aar文件，上传到jcenter。

![](http://o7y1sf21i.bkt.clouddn.com/blog/012/%E7%B2%98%E8%B4%B4%E5%9B%BE%E7%89%876.png)


## 配置项目根目录的build.gradle文件


```
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
    
        classpath 'com.android.tools.build:gradle:2.0.0'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files

		 //增加两个插件
		 //android-maven-gradle-plugin插件是用来打包Maven所需文件的
		 //gradle-bintray-plugin插件是用来将生成的Maven所需文件上传到Bintray的
        classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.2'
        classpath 'com.github.dcendents:android-maven-gradle-plugin:1.3'
    }
}

allprojects {
    repositories {
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

```

## 配置待上传的Android Library的gradle文件

在Android Library目录下需要增改如下几个文件。

> install.gradle
> bintray.gradle
> dependencies.gradle
> build.gradle
> local.properties


### install.gradle

```
apply plugin: 'com.github.dcendents.android-maven'
apply from: 'dependencies.gradle'

group = publishedGroupId   // Maven Group ID for the artifact

install {
    repositories.mavenInstaller {

        // This generates POM.xml with proper parameters

        pom {
            project {
                packaging 'aar'
                groupId publishedGroupId
                artifactId artifact

                // Add your description here
                name libraryName
                description libraryDescription
                url siteUrl

                // Set your license
                licenses {
                    license {
                        name licenseName
                        url licenseUrl
                    }
                }
                developers {
                    developer {
                        id developerId
                        name developerName
                        email developerEmail
                    }
                }
                scm {
                    connection gitUrl
                    developerConnection gitUrl
                    url siteUrl
                }
            }
        }
    }
}

```


### bintray.gradle

```
apply plugin: 'com.jfrog.bintray'
apply from: 'dependencies.gradle'

version = libraryVersion

if (project.hasProperty("android")) { // Android libraries
    task sourcesJar(type: Jar) {
        classifier = 'sources'
        from android.sourceSets.main.java.srcDirs
    }

    task javadoc(type: Javadoc) {
        source = android.sourceSets.main.java.srcDirs
        classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
    }
} else { // Java libraries
    task sourcesJar(type: Jar, dependsOn: classes) {
        classifier = 'sources'
        from sourceSets.main.allSource
    }
}

task javadocJar(type: Jar, dependsOn: javadoc) {
    classifier = 'javadoc'
    from javadoc.destinationDir
}

artifacts {
    archives javadocJar
    archives sourcesJar
}

// Bintray
Properties properties = new Properties()
properties.load(project.rootProject.file('local.properties').newDataInputStream())

bintray {
    user = properties.getProperty("bintray.user")
    key = properties.getProperty("bintray.apikey")

    configurations = ['archives']

    pkg {
        repo = bintrayRepo
        name = bintrayName
        desc = libraryDescription
        websiteUrl = siteUrl
        vcsUrl = gitUrl
        licenses = allLicenses
        publish = true
        publicDownloadNumbers = true

        version {
            desc = libraryDescription
            gpg {
            		//Optional. The passphrase for GPG signing'
                sign = false //Determines whether to GPG sign the files. The default is false
                passphrase = properties.getProperty("bintray.gpg.password")
            }
        }
    }
}


```


### dependencies.gradle

这个文件主要用来保存一些项目信息。

```
project.ext {

    bintrayRepo = 'maven'
    bintrayName = 'statusbar'			//与bintray中新建的项目名称相同

    publishedGroupId = 'org.chen.statusbar'
    libraryName = 'statusbar'       	//确保与android studio里面library名称相同
    artifact = 'statusbar'				//确保与android studio里面library名称相同

    libraryDescription = 'Android System Status Bar'

    siteUrl = 'https://github.com/czhzero/AndroidSystemStatusBar'
    gitUrl = 'https://github.com/czhzero/AndroidSystemStatusBar.git'

    libraryVersion = '1.0.0'

    developerId = 'czhzero'
    developerName = 'chen'
    developerEmail = 'czhpxl007@163.com'

    licenseName = 'The Apache Software License, Version 2.0'
    licenseUrl = 'http://www.apache.org/licenses/LICENSE-2.0.txt'
    allLicenses = ["Apache-2.0"]

}
```


### build.gradle

在library中的build.gradle文件下，增加下面两句引用。

```

apply from: 'install.gradle'
apply from: 'bintray.gradle'


```

### local.properties

在local.properties文件中，增加如下字段。因为这两个字段是敏感信息，所以建议将这些信息放到项目本地文件中。

```
bintray.user=你的用户名,不是邮箱
bintray.apikey=你的bintray账号的apikey

```


## 执行命令打包并上传到Bintray

```
//进入项目根目录
> ./gradlew :模块名:install  
> ./gradlew :模块名:bintrayUpload  

或者

> gradlew :模块名:install  
> gradlew :模块名:bintrayUpload  

```

命令执行成功后，会返回 `build successful` 字样。


## 同步library文件到jcenter

前面所有步骤走完之后实际上只是上传了你的项目到Bintray而已，并没有被包含在jcenter中，要想提交到jcenter中还需要Bintray的审核。

登入Bintray网站，进入个人中心，在右侧的Owned Repositories区域点击Maven的图标，进入你的Maven项目列表。

如果已经上传成功了，在这里就能看到你的项目，进入项目详情，在右下角的Linked To区域点击Add to JCenter，然后在Comments输入框里随便填写下信息，最后点Send提交请求即可

![](http://o7y1sf21i.bkt.clouddn.com/blog/012/%E7%B2%98%E8%B4%B4%E5%9B%BE%E7%89%877.png)

一般情况下审核需要4到5个小时，耐心等待就行了，审核通过后会给你发邮件通知你，并且以后更新项目就不需要再审核了。


## 修改library库代码, 升级库版本

升级库版本代码，只需要修改代码后，然后修改`dependencies.gradle`里面版本号。

重新执行如下命令即可。

```
//进入项目根目录
> ./gradlew :模块名:install  
> ./gradlew :模块名:bintrayUpload  

或者

> gradlew :模块名:install  
> gradlew :模块名:bintrayUpload  


```




## 常见问题

- `message:Unable to upload files: Maven group, artifact or version defined in the pom file do not match the file path`

这个问题一般都是你的module的名字和你配置的artifactId不一致导致的。

- `Error : cause android.compileSdkVersion is missing `

```
apply from: 'install.gradle'
apply from: 'bintray.gradle'

```
在module模块的build.gradle添加上述两句话时，这两句话要写到末尾，不然会提示找不到`compileSdkVersion`

## 参考文章

[如何使用Android Studio把自己的Android library分享到jCenter和Maven Central](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0623/3097.html#commettop)
[【Android】5分钟发布Android Library项目到JCenter](http://www.jianshu.com/p/0e7b8e14f0cd)