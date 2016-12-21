---
title: 如何在Android7.0系统下通过Intent安装apk
date: 2016-12-21 10:14:49
categories: Android
tags: android7.0
---
 
 Android系统升级到7.0之后，安全性提高了不少，过去我们通常是使用这样的代码进行apk的安装操作。
 
 ```
	 Intent intent = new Intent(Intent.ACTION_VIEW);
	 intent.setDataAndType(Uri.fromFile(apkFile), "application/vnd.android.package-archive");
	 context.startActivity(intent);
	 
 ```
 
 但是在Android7.0的系统上，运行这段代码，会报如下错误。
 
 > Caused by: android.os.FileUriExposedException
 
 <!-- more -->
 

原因是，安卓官方为了提高私有文件的安全性，面向 Android 7.0 或更高版本的应用私有目录被限制访问　(0700)。此设置可防止私有文件的元数据泄漏，如它们的大小或存在性.
 
传递软件包网域外的 file:// URI 可能给接收器留下无法访问的路径。因此，尝试传递 file:// URI 会触发 FileUriExposedException。分享私有文件内容的推荐方法是使用 [FileProvider](https://developer.android.google.cn/reference/android/support/v4/content/FileProvider.html)。


## 1.定义一个FileProvider

```
<manifest>
    ...
    <application>
        ...
        <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="com.mydomain.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            ...
        </provider>
        ...
    </application>
</manifest>

```

## 2.添加可用权限的文件目录

在res目录下，增加xml文件夹，并新建一个名为 `file_paths.xml` 的文件。文件内容格式如下:

```
<paths xmlns:android="http://schemas.android.com/apk/res/android">
   		<files-path name="name1" path="test1" />
    	...
</paths>

```

<path>标签下面必须包含至少包含以下标签中的一个或者多个。

### files-path 

```
<files-path name="name1" path="test1" />

```

表示Context.getFilesDir()目录或者其子目录。

> 示例 : /data/data/com.chen.gradle/files/test1


### cache-path 

```
<cache-path name="name2" path="test2" />

```

表示Context.getCacheDir()目录或者其子目录。

> 示例 : /data/data/com.chen.gradle/cache/test2

### external-path

```
<external-path name="name3" path="test3" />

```

表示Environment.getExternalStorageDirectory()目录或者其子目录。

> 示例 : /storage/emulated/0/test3

### external-files-path  

```
<external-files-path name="name4" path="test4" />

```

表示Context.getExternalFilesDir(null)目录或者其子目录。

> 示例 : /storage/emulated/0/Android/data/com.chen.gradle/files/test4

### external-cache-path 

```
<external-cache-path name="name5" path="test5" />

```
表示Context.getExternalCacheDir()目录或者其子目录。

> 示例 : /storage/emulated/0/Android/data/com.chen.gradle/cache/test5

## 3.增加<path>到provider

通过`<meta-data>`标签将上面的filepath添加到provider当中。

```
<provider
    android:name="android.support.v4.content.FileProvider"
    android:authorities="com.mydomain.fileprovider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_paths" />
</provider>

```

## 4.通过provider生成Uri

```

File imagePath = new File(Context.getFilesDir(), "test1");
File newFile = new File(imagePath, "default_image.jpg");
Uri contentUri = FileProvider.getUriForFile(getContext(), "com.mydomain.fileprovider", newFile);

```

## 5.赋予临时权限给Uri

```
intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);

```

最终安装apk的代码变成这样:

```

public static void installApk(Context context, String apkPath) {
    if (context == null || TextUtils.isEmpty(apkPath)) {
        return;
    }

    
    File file = new File(apkPath);
    Intent intent = new Intent(Intent.ACTION_VIEW);

    //判读版本是否在7.0以上
    if (Build.VERSION.SDK_INT >= 24) {
        //provider authorities
        Uri apkUri = FileProvider.getUriForFile(context, "com.mydomain.fileprovider", file);
        //Granting Temporary Permissions to a URI
        intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
        intent.setDataAndType(apkUri, "application/vnd.android.package-archive");
    } else {
        intent.setDataAndType(Uri.fromFile(file), "application/vnd.android.package-archive");
    }

    context.startActivity(intent);
}


```


## 参考文献

[Android 7.0 行为变更](https://developer.android.google.cn/about/versions/nougat/android-7.0-changes.html)



