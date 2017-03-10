---
title: Android自定义Dialog的Match_parent无效问题解决
date: 2017-03-10 11:12:53
categories: Android
tags: Dialog
---


Android 开发过程中，自定义dialog 用得很多，通常都是通过继承Dialog, 再加载自定义的xml来实现。

 <!-- more -->


```
//自定义Dialog
public class OrderFilterDialog extends Dialog {

    public OrderFilterDialog(Context context) {
        super(context);
        initView(this);
    }

    public OrderFilterDialog(Context context, int themeResId) {
        super(context, themeResId);
        initView(this);
    }


    public  void initView(Dialog dialog) {
        dialog.requestWindowFeature(Window.FEATURE_NO_TITLE);
        dialog.setContentView(R.layout.dialog_search_order);
        show();
        dialog.setCancelable(false);
        Window window = dialog.getWindow();
        WindowManager.LayoutParams lp = window.getAttributes();
        lp.width = WindowManager.LayoutParams.MATCH_PARENT;
        lp.height = WindowManager.LayoutParams.WRAP_CONTENT;
        lp.gravity = Gravity.TOP;
        window.setAttributes(lp);
        window.setWindowAnimations(R.style.DialogInOutTopAnimation);
    }
    
}
```

运行后，发现效果如下，


![这里写图片描述](http://img.blog.csdn.net/20160114170927743)

我们发现，match_parent属性根本没有生效。查看Dialog源码才发现，默认Dialog是有一个theme属性的，坑。

Dialog部分代码如下，

```
/**
140     * Creates a dialog window that uses the default dialog theme.
141     * <p>
142     * The supplied {@code context} is used to obtain the window manager and
143     * base theme used to present the dialog.
144     *
145     * @param context the context in which the dialog should run
146     * @see android.R.styleable#Theme_dialogTheme
147     */
148    public Dialog(@NonNull Context context) {
149        this(context, 0, true);   //默认传入0
150    }
151
152    /**
153     * Creates a dialog window that uses a custom dialog style.
154     * <p>
155     * The supplied {@code context} is used to obtain the window manager and
156     * base theme used to present the dialog.
157     * <p>
158     * The supplied {@code theme} is applied on top of the context's theme. See
159     * <a href="{@docRoot}guide/topics/resources/available-resources.html#stylesandthemes">
160     * Style and Theme Resources</a> for more information about defining and
161     * using styles.
162     *
163     * @param context the context in which the dialog should run
164     * @param themeResId a style resource describing the theme to use for the
165     *              window, or {@code 0} to use the default dialog theme
166     */
167    public Dialog(@NonNull Context context, @StyleRes int themeResId) {
168        this(context, themeResId, true);
169    }
170
171    Dialog(@NonNull Context context, @StyleRes int themeResId, boolean createContextThemeWrapper) {
172        if (createContextThemeWrapper) {
173            if (themeResId == 0) {
174                final TypedValue outValue = new TypedValue();
175                context.getTheme().resolveAttribute(R.attr.dialogTheme, outValue, true);             //若值为0，则默认设置dialogTheme样式
176                themeResId = outValue.resourceId;
177            }
178            mContext = new ContextThemeWrapper(context, themeResId);
179        } else {
180            mContext = context;
181        }
182
183        mWindowManager = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);
184
185        final Window w = new PhoneWindow(mContext);
186        mWindow = w;
187        w.setCallback(this);
188        w.setOnWindowDismissedCallback(this);
189        w.setWindowManager(mWindowManager, null, null);
190        w.setGravity(Gravity.CENTER);
191
192        mListenersHandler = new ListenersHandler(this);
193    }
```
其中

> if (themeResId == 0) {
>     final TypedValue outValue = new TypedValue();
>     context.getTheme().resolveAttribute(R.attr.dialogTheme, outValue, true);            
> 	themeResId = outValue.resourceId;
> }

从上述代码可以看出，若Dialog没有设置默认样式，系统会自动给你适配为R.attr.dialogTheme样式。
而查看themes.xml后，发现 dialogTheme样式是自带有Padding值的，所以match_parent自然是无效的。


那么，如何实现match_parent呢

(1) 设置padding值为0。

```
dialog.getWindow().getDecorView().setPadding(0, 0, 0, 0);
```

(2) 自定义一个Theme，覆盖原生Theme。

```
OrderFilterDialog dialog = new OrderFilterDialog(context, R.style.test);
```


运行效果如下，

![这里写图片描述](http://img.blog.csdn.net/20160114171122552)