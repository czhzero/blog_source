---
title: Android分享机制总结
date: 2017-03-10 11:01:34
categories: Android
tags: 分享
---

Android应用分享功能是一般应用所必不可少到功能。
一般有以下三种方式。

 - 调用系统Activity进行分享 
 - 根据第三方App的包名和类名直接启动 
 - 注册第三方app账号，集成sdk

<!-- more -->


###1.调用系统Activity进行分享

这种方式最为简单，但是有些分享软件的高级功能无法使用。
分享一般分为图片，文字，或者图片文字混合的分享。

 - 分享文字

```
Intent intent= new Intent(Intent.ACTION_SEND);
intent.setType("text/plain");
intent .putExtra(Intent.EXTRA_SUBJECT, "subject");
intent.putExtra(Intent.EXTRA_TEXT,"my share content");
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
startActivity(Intent.createChooser(intentItem, "share"));
```

 - 分享图片

```
Intent shareIntent = new Intent(); 
shareIntent.setAction(Intent.ACTION_SEND); 
File file=new File(imgPath);
Uri uri=Uri.fromFile(file);  
shareIntent.putExtra(Intent.EXTRA_STREAM, uri); 
shareIntent.putExtra(Intent.EXTRA_TEXT, "share content");
shareIntent.setType("image/*"); 
startActivity(Intent.createChooser(shareIntent,"image share "));
```

 - 分享多张图片
 

```
ArrayList<Uri> imageUris = new ArrayList<Uri>();
imageUris.add(uri_1); 
imageUris.add(uri_2);
Intent shareIntent = new Intent();
shareIntent.setAction(Intent.ACTION_SEND_MULTIPLE);
shareIntent.putParcelableArrayListExtra(Intent.EXTRA_STREAM, imageUris);
shareIntent.setType("image/*");
startActivity(Intent.createChooser(shareIntent, "image share"));
```

 - 同时分享文字和图片

```
Intent intent = new Intent(Intent.ACTION_SEND);
if(imgPath == null || imgPath.equals("")) {
	intent.setType("text/plain");
} else {
		File f = new File(imgPath);
		if (f != null && f.exists() && f.isFile()) {
	       intent.setType("image/jpg");
	       Uri u = Uri.fromFile(f);
	       intent.putExtra(Intent.EXTRA_STREAM, u);
	   }
}
intent.putExtra(Intent.EXTRA_SUBJECT, msgTitle);
intent.putExtra(Intent.EXTRA_TEXT, msgText);
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
startActivity(Intent.createChooser(intent, activityTitle)); 
```

###2.根据第三方App的包名和类名直接启动

根据第三方包名直接启动，可以自定义界面，同时也不需要任何sdk，节省apk空间大小。缺点是，万一第三方app更新类名或者增加了某些特殊限制，这一方法也将会失效。


- 获取可分享app的包名和activity名
	 

```
List<ResolveInfo> mApps = new ArrayList<ResolveInfo>();
Intent intent = new Intent(Intent.ACTION_SEND, null);
intent.addCategory(Intent.CATEGORY_DEFAULT);
intent.setType("text/plain");
PackageManager pManager = context.getPackageManager();
mApps = pManager.queryIntentActivities(intent, PackageManager.COMPONENT_ENABLED_STATE_DEFAULT);
if(mApps!=null)
{
 for (ResolveInfo resolveInfo : mApps) {
 Log.v("czh","packageName="+resolveInfo.activityInfo.packageName);
 Log.v("czh","activityName="+resolveInfo.activityInfo.name);
 }
}
```

 - 传递数据到第三方app 

```
try {

	Intent intent = new Intent();
	ComponentName componentName = new ComponentName(
			"com.renren.mobile.android",
			"com.renren.mobile.android.publisher.UploadPhotoEffect");
	intent.setComponent(componentName);
	intent.setAction(Intent.ACTION_SEND);
	intent.setType("image/*");
	intent.putExtra(Intent.EXTRA_TEXT, mSelectedString);
	startActivity(intent);
} catch (Exception e) {
	e.printStackTrace();
}
```

###3.注册第三方app账号，集成sdk

这种是最常用的方法，参考官方sdk即可。虽然麻烦了点，但是是最可靠的方法。