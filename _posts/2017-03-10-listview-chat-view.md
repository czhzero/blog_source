---
title: ListView聊天窗口与输入法键盘冲突解决方法(聊天框在viewpager里)
date: 2017-03-10 10:31:18
categories: Android
tags: ListView
---

在使用listView显示聊天窗口时，弹出输入法，我们期待的效果是输入框上移动，listview自动定位到最后的聊天内容。
但是项目中遇到了这样的情况，

<!-- more -->

![这里写图片描述](http://img.blog.csdn.net/20160514114854249)

聊天框底下有tab,我们就需要解决三件事。

> 1.输入法弹出时候ListView聊天内容不被遮挡
> 
> 2.输入法弹出的时候，底部tab隐藏
> 
> 3.输入法弹出的时候，聊天内容自动定位到最后一行。


###1.在manifest的activity中设置输入法属性。

android:windowSoftInputMode

这个是Android1.5后的一个新特性。activity主窗口与软键盘的交互模式，可以用来避免输入法面板遮挡问题，
这个属性能影响两件事情：
【一】当有焦点产生时，软键盘是隐藏还是显示
【二】是否减少活动主窗口大小以便腾出空间放软键盘


各值的含义：

- 【A】stateUnspecified：软键盘的状态并没有指定，系统将选择一个合适的状态或依赖于主题的设置
- 【B】stateUnchanged：当这个activity出现时，软键盘将一直保持在上一个activity里的状态，无论是隐藏还是显示
- 【C】stateHidden：用户选择activity时，软键盘总是被隐藏
- 【D】stateAlwaysHidden：当该Activity主窗口获取焦点时，软键盘也总是被隐藏的
- 【E】stateVisible：软键盘通常是可见的
- 【F】stateAlwaysVisible：用户选择activity时，软键盘总是显示的状态
- 【G】adjustUnspecified：默认设置，通常由系统自行决定是隐藏还是显示
- 【H】adjustResize：该Activity总是调整屏幕的大小以便留出软键盘的空间
- 【I】adjustPan：当前窗口的内容将自动移动以便当前焦点从不被键盘覆盖和用户能总是看到输入内容的部分


分别尝试了下面两种组合，

```
android:windowSoftInputMode="adjustPan|stateAlwaysHidden" 
```

![这里写图片描述](http://img.blog.csdn.net/20160514114956946)

```
android:windowSoftInputMode="adjustResize|stateAlwaysHidden"
```

![这里写图片描述](http://img.blog.csdn.net/20160514115015384)

第一种情况，底部tab是看不见了，但是顶部的标题栏也被顶掉了，
第二种情况是标题栏还在，但是底部tab也可以看见。
针对第一情况，可以采取当标题被顶掉时，弹出一个popupwindow代替，但是这种方法需要考虑，弹出位置和tab直接之间切换的影响，比较麻烦。因此本人针对第二钟情况隐藏底部tab。

###2.监听键盘弹出事件，隐藏底部tab

因为resize模式会改变布局的大小，所以我们可以监听布局尺寸的变化,从而监听键盘是否弹出。

 - 方法一：重新onSizeChanged

```
 @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        //计算尺寸，判断变大还是变小。
    }
```
  

 - 方法二：增加addOnGlobalLayoutListener


```
vpContent.getViewTreeObserver().addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
	    @Override
	    public void onGlobalLayout() {
	        if(vpContent.getRootView().getHeight() - vpContent.getHeight() > 500) {
	            //隐藏tab
	        } else {
	            //显示tab
	        }
	    }
	});
```
	


###3.键盘弹出时，listview定位到最后一行

 - 方法一：增加listview的transcriptMode属性

```
	android:transcriptMode="alwaysScroll"
```

 - 方法二：调用setSelection()方法 (这种方法不会有屏幕跳动) 

```
    listview.setSelection(mData.size()-1);
```
     
   不过，隐藏底部tab时，listview会重绘，出现了数据抖动效果，这个问题还有待解决。
      
    


