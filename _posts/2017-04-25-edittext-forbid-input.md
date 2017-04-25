---
title: EditText的针对输入法的强行显示与隐藏
date: 2017-04-25 16:23:24
categories: Android
tags: EditText
---

EditText是Android中最常用的控件之一，很多时候我们需要对这个控件输入与否进行控制。

<!-- more -->

## 自动弹出输入法

### 方法一

```
<activity android:name=".ui.login"
   android:configChanges="orientation|keyboardHidden|locale"
   android:screenOrientation="portrait"
   android:windowSoftInputMode="stateVisible|adjustPan" > 
</activit

```

设置stateVisible属性。不过这个方法有时候并不管用。

### 方法二(推荐)

```
/**
 * 强行弹出输入法
 *
 * @param activity
 * @param editText
 */
public static void showSoftInputFromWindow(Activity activity, EditText editText) {
    editText.setFocusable(true);
    editText.setFocusableInTouchMode(true);
    editText.requestFocus();
    activity.getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_STATE_ALWAYS_VISIBLE);
}


```

## 首次进入禁止弹出输入法

### 方法一

```
<activity android:name=".Main"
android:label="@string/app_name"
android:windowSoftInputMode="adjustUnspecified|stateHidden"
android:configChanges="orientation|keyboardHidden">

</activity>

```

设置android:windowSoftInputMode为stateHidden。


### 方法二(推荐)

在对应的xml文件里面，添加一个看不见的view,设置焦点。

```
<View
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:focusable="true"
    android:focusableInTouchMode="true"></View>
    
```


## 永久禁止弹出输入法

### 方法一 (推荐)

```
/**
 * 永久隐藏softinput
 *
 * @param editText
 */
public static void hiddenSoftInputFromWindow(Activity activity, EditText editText) {
    if (android.os.Build.VERSION.SDK_INT <= 10) {//4.0以下
        editText.setInputType(InputType.TYPE_NULL);
    } else {
        activity.getWindow().setSoftInputMode(
                WindowManager.LayoutParams.SOFT_INPUT_STATE_ALWAYS_HIDDEN);
        try {
            Class<EditText> cls = EditText.class;
            Method setShowSoftInputOnFocus;
            setShowSoftInputOnFocus = cls.getMethod("setShowSoftInputOnFocus",
                    boolean.class);
            setShowSoftInputOnFocus.setAccessible(true);
            setShowSoftInputOnFocus.invoke(editText, false);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

### 方法二

```
     <EditText
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:inputType="none"/>

```

这种方法，无法显示光标。

## 无输入法的插入与删除

### EditText最大长度

输入法被禁止后，光标跳转需要通过requestFocus进行手动跳转。

通常当输入内容达到输入框最长长度时，进行自动跳转

获取EditText最长输入长度方法如下。

```
/**
 * 获取edittext的最大长度
 * @param et
 * @return
 */
public static int getEditTextMaxLength(EditText et) {
    int length = 0;
    try {
        InputFilter[] inputFilters = et.getFilters();
        for (InputFilter filter : inputFilters) {
            Class<?> c = filter.getClass();
            if (c.getName().equals("android.text.InputFilter$LengthFilter")) {
                Field[] f = c.getDeclaredFields();
                for (Field field : f) {
                    if (field.getName().equals("mMax")) {
                        field.setAccessible(true);
                        length = (Integer) field.get(filter);
                    }
                }
            }
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
    return length;
}


```

### 插入字符

```
/**
 * 插入字符
 * @param et
 * @param str
 */
public static boolean insertEditText(EditText et, String str) {
    int position = et.getSelectionStart();

    if(position >= getEditTextMaxLength(et)) {
        return false; //插入失败
    } else {
        Editable editable = et.getText();
        editable.insert(position, str);
        return true;
    }

}

```

### 删除字符

```
/**
 * 删除字符
 *
 * @param et
 */
public static boolean deleteEditText(EditText et) {
    int position = et.getSelectionStart();
    Editable editable = et.getText();
    if (position > 0) {
        editable.delete(position - 1, position);
        return true;
    } else {
        return false; //删除失败
    }
}

```