---
title: WindowManager悬浮窗如何监听Home键和返回键
date: 2016-08-26 17:06:21
categories: Android
tags: WindowManager
---


使用WindowManager可以在其他应用最上层，甚至手机桌面最上层显示窗口。
调用的是WindowManager继承自基类的addView方法和removeView方法来显示和隐藏窗口。但是WindowManager中的View并不能直接地监听到Home和Back的KeyEvent。

通常我们会通过`setOnKeyListener`方法进行监听，不过，很不幸这种方式监听不到事件。

<!-- more -->


```

View view = LayoutInflater.from(context).inflate(R.layout.popupwindow,
                null);
                
view.setOnKeyListener(new OnKeyListener() {

            @Override
            public boolean onKey(View v, int keyCode, KeyEvent event) {
                switch (keyCode) {
                case KeyEvent.KEYCODE_BACK:
                    //dosomething
                    return true;
                case KeyEvent.KEYCODE_HOME
                    //dosomething
                    return true;
                default:
                    return false;
                }
            }
        });


```

所以我们要通过其他方式进行监听，

- 监听Home键事件

```
    /**
     * 自定义广播
     */
    private final static BroadcastReceiver homeListenerReceiver = new BroadcastReceiver() {
        final String SYSTEM_DIALOG_REASON_KEY = "reason";

        final String SYSTEM_DIALOG_REASON_HOME_KEY = "homekey";

        @Override
        public void onReceive(Context context, Intent intent) {
            String action = intent.getAction();
            if (action.equals(Intent.ACTION_CLOSE_SYSTEM_DIALOGS)) {
                String reason = intent.getStringExtra(SYSTEM_DIALOG_REASON_KEY);
                if (reason != null && reason.equals(SYSTEM_DIALOG_REASON_HOME_KEY)) {
                    // 处理自己的逻辑
                }
            }
        }
    };
    
    
    //在创建View时注册Receiver
    IntentFilter homeFilter = new IntentFilter(Intent.ACTION_CLOSE_SYSTEM_DIALOGS);
	context.registerReceiver(homeListenerReceiver, homeFilter);
	
	//在View消失时反注册Receiver
	if (homeListenerReceiver != null) {
   context.unregisterReceiver(homeListenerReceiver);
	}
    


```


- 监听Back键事件

```
	//重写dispatchKeyEvent
    @Override
    public boolean dispatchKeyEvent(KeyEvent event) {
        switch (event.getKeyCode()) {
            case KeyEvent.KEYCODE_BACK:
            case KeyEvent.KEYCODE_MENU:
                // 处理自己的逻辑break;
            default:
                break;
        }
        return super.dispatchKeyEvent(event);
    }

```