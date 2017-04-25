---
title: ScrollView内部嵌套ListView的高度计算问题
date: 2017-04-25 15:55:25
categories: Android
tags: ListView嵌套
---


ScrollView嵌套ListView只显示一行的解决办法相信很多人都遇到过，原因是是因为ScrollView包着之后，ListView的onMeasure方法得到的父布局的mode为View.MeasureSpec.UNSPECIFIED,导致ListView计算的高度错误引起。

<!-- more -->

解决方案:

```

/**
 * Created by chenzhaohua on 17/4/25.
 */

public class CustomListView extends ListView {

    public CustomListView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public CustomListView(Context context) {
        super(context);
    }

    public CustomListView(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
    }

    @Override
    public void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {

        int expandSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE >> 2,
                MeasureSpec.AT_MOST); //将模式改为AT_MOST,并且将size设置为很大的数值。
        super.onMeasure(widthMeasureSpec, expandSpec);
    }

}


```