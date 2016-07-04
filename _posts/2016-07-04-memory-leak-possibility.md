---
title: Android内存泄漏常见场景分析
date: 2016-07-04 22:59:00
categories: Android
tags: 性能优化
---


Java是垃圾回收语言的一种，其优点是开发者无需特意管理内存分配，降低了应用由于局部故障(segmentation fault)导致崩溃，同时防止未释放的内存把堆栈(heap)挤爆的可能，所以写出来的代码更为安全。

不幸的是，在Java中仍存在很多容易导致内存泄漏的逻辑可能(logical leak)。如果不小心，你的Android应用很容易浪费掉未释放的内存，最终导致内存用光的错误抛出(out-of-memory，OOM)。

<!-- more -->

一般内存泄漏(traditional memory leak)的原因是：当该对象的所有引用都已经释放了，对象仍未被释放。逻辑内存泄漏(logical memory leak)的原因是：当应用不再需要这个对象，当仍未释放该对象的所有引用。如果持有对象的强引用，垃圾回收器是无法在内存中回收这个对象。

在Android开发中，最容易引发的内存泄漏问题的是Context。比如Activity的Context，就包含大量的内存引用，例如View Hierarchies和其他资源。一旦泄漏了Context，也意味泄漏它指向的所有对象。Android机器内存有限，太多的内存泄漏容易导致OOM。

一般来说, 内存泄漏的主要问题可以分为以下几种类型：

- **静态变量引起的内存泄漏**

例如: Static Activities, Static Views , SensorManager等系统static对象

- **非静态内部类引起的内存泄漏**

例如: Inner Classes, Anonymous Classes, Handler。

- **资源未关闭引起的内存泄漏**

例如: BroadcastReceiver、Cursor、Bitmap

- **耗时线程引起的内存泄漏**

例如: Thread, TimeTask, AsyncTask

- **频繁创建对象引起的内存泄漏**

例如: 构造Adapter时，没有使用缓存的convertView


下面是常见的一些内存泄漏的例子。检查一下你有没有遇到下列的情况。


## 静态变量引起的内存泄漏

> 解决方案: 
> 1. 及时释放无用的静态变量
> 2. 寻找与该静态变量生命周期差不多的替代对象
> 3. 将强引用方式改成弱引用


### Static Activities

在类中定义了静态Activity变量，把当前运行的Activity实例赋值于这个静态变量。
如果这个静态变量在Activity生命周期结束后没有清空，就导致内存泄漏。因为static变量是贯穿这个应用的生命周期的，所以被泄漏的Activity就会一直存在于应用的进程中，不会被垃圾回收器回收。

代码示例:

**修正前**

```
 static Activity activity;

    void setStaticActivity() {
      activity = this;
    }

    View saButton = findViewById(R.id.sa_button);
    saButton.setOnClickListener(new View.OnClickListener() {
      @Override public void onClick(View v) {
        setStaticActivity();
        nextActivity();
      }
    });

```

***修正后***

```
static WeakReference<Activity> weakActivity;

    void setStaticActivity() {
        weakActivity = new WeakReference<Activity>(this);
    }
    
    View saButton = findViewById(R.id.sa_button);
    saButton.setOnClickListener(new View.OnClickListener() {
      @Override public void onClick(View v) {
        setStaticActivity();
        nextActivity();
      }
    });

```


### Static Views

类似的情况会发生在单例模式中，如果Activity经常被用到，那么在内存中保存一个实例是很实用的。正如之前所述，强制延长Activity的生命周期是相当危险而且不必要的，无论如何都不能这样做。

特殊情况：如果一个View初始化耗费大量资源，而且在一个Activity生命周期内保持不变，那可以把它变成static，加载到视图树上(View Hierachy)，像这样，当Activity被销毁时，应当释放资源。（译者注：示例代码中并没有释放内存，把这个static view置null即可，但是还是不建议用这个static view的方法）

代码示例:

**修正前**

```
static view;

    void setStaticView() {
      view = findViewById(R.id.sv_button);
    }

    View svButton = findViewById(R.id.sv_button);
    svButton.setOnClickListener(new View.OnClickListener() {
      @Override public void onClick(View v) {
        setStaticView();
        nextActivity();
      }
    });

```

***修正后***

```
static WeakReference<View> weakView;

    void setStaticView() {
      View view = findViewById(R.id.sv_button);
      weakView = new WeakReference<View>(view)
    }

    View svButton = findViewById(R.id.sv_button);
    svButton.setOnClickListener(new View.OnClickListener() {
      @Override public void onClick(View v) {
        setStaticView();
        nextActivity();
      }
    });

```

### SensorManager

最后，通过Context.getSystemService(int name)可以获取系统服务。这些服务工作在各自的进程中，帮助应用处理后台任务，处理硬件交互。如果需要使用这些服务，可以注册监听器，这会导致服务持有了Context的引用，如果在Activity销毁的时候没有注销这些监听器，会导致内存泄漏。

代码示例:

**修正前**

```
void registerListener() {
               SensorManager sensorManager = (SensorManager) getSystemService(SENSOR_SERVICE);
               Sensor sensor = sensorManager.getDefaultSensor(Sensor.TYPE_ALL);
               sensorManager.registerListener(this, sensor, SensorManager.SENSOR_DELAY_FASTEST);
        }

        View smButton = findViewById(R.id.sm_button);
        smButton.setOnClickListener(new View.OnClickListener() {
            @Override public void onClick(View v) {
                registerListener();
                nextActivity();
            }
        });


```

**修正后**

```
//增加unregist方法
sensorManager.unregisterListener(this);       

```



## 非静态内部类引起的内存泄漏


> 解决方案: 1. 将内部类变成静态内部类, 2. 如果有强引用Activity中的属性，则将该属性的引用方式改为弱引用, 3. 在业务允许的情况下，当Activity执行onDestory时，结束这些耗时任务


### Inner Classes

继续，假设Activity中有个内部类，这样做可以提高可读性和封装性。将如我们创建一个内部类，而且持有一个静态变量的引用，恭喜，内存泄漏就离你不远了（译者注：销毁的时候置空，嗯）。

内部类的优势之一就是可以访问外部类，不幸的是，导致内存泄漏的原因，就是内部类持有外部类实例的强引用。

代码示例:

**修正前**


```
 private static Object inner;

    void createInnerClass() {
        class InnerClass {
        }
        inner = new InnerClass();
    }

    View icButton = findViewById(R.id.ic_button);
    icButton.setOnClickListener(new View.OnClickListener() {
        @Override public void onClick(View v) {
            createInnerClass();
            nextActivity();
        }
    });

```

### Anonymous Classes

相似地，匿名类也维护了外部类的引用。所以内存泄漏很容易发生，当你在Activity中定义了匿名的AsyncTsk
。当异步任务在后台执行耗时任务期间，Activity不幸被销毁了（译者注：用户退出，系统回收），这个被AsyncTask持有的Activity实例就不会被垃圾回收器回收，直到异步任务结束。

代码示例:

**修正前**

```

void startAsyncTask() {
        new AsyncTask<Void, Void, Void>() {
            @Override protected Void doInBackground(Void... params) {
                while(true);
            }
        }.execute();
    }

    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    View aicButton = findViewById(R.id.at_button);
    aicButton.setOnClickListener(new View.OnClickListener() {
        @Override public void onClick(View v) {
            startAsyncTask();
            nextActivity();
        }
    });

```

**修正后**




### Handler

同样道理，定义匿名的Runnable，用匿名类Handler执行。Runnable内部类会持有外部类的隐式引用，被传递到Handler的消息队列MessageQueue中，在Message消息没有被处理之前，Activity实例不会被销毁了，于是导致内存泄漏。

代码示例:

**修正前**

```
 void createHandler() {
        new Handler() {
            @Override public void handleMessage(Message message) {
                super.handleMessage(message);
            }
        }.postDelayed(new Runnable() {
            @Override public void run() {
                while(true);
            }
        }, Long.MAX_VALUE >> 1);
    }


    View hButton = findViewById(R.id.h_button);
    hButton.setOnClickListener(new View.OnClickListener() {
        @Override public void onClick(View v) {
            createHandler();
            nextActivity();
        }
    });

```

## 耗时线程引起的内存泄漏

> 解决方案:1.将内部类变成静态内部类 , 2.如果有强引用Activity中的属性，则将该属性的引用方式改为弱引用, 3.在业务允许的情况下，当Activity执行onDestory时，结束这些耗时任务


### Threads

我们再次通过Thread和TimerTask来展现内存泄漏。

代码示例:

**修正前**

```

void spawnThread() {
        new Thread() {
            @Override public void run() {
                while(true);
            }
        }.start();
    }

    View tButton = findViewById(R.id.t_button);
    tButton.setOnClickListener(new View.OnClickListener() {
      @Override public void onClick(View v) {
          spawnThread();
          nextActivity();
      }
    });

```

### TimerTask

只要是匿名类的实例，不管是不是在工作线程，都会持有Activity的引用，导致内存泄漏。

代码示例:

**修正前**


```

void scheduleTimer() {
        new Timer().schedule(new TimerTask() {
            @Override
            public void run() {
                while(true);
            }
        }, Long.MAX_VALUE >> 1);
    }

    View ttButton = findViewById(R.id.tt_button);
    ttButton.setOnClickListener(new View.OnClickListener() {
        @Override public void onClick(View v) {
            scheduleTimer();
            nextActivity();
        }
    });


```

## 资源未关闭引起的内存泄漏


> 解决方案: 在资源使用完成后，记得关闭资源。


### Cursor

资源性对象比如(Cursor，File文件等)往往都用了一些缓冲，我们在不使用的时候，应该及时关闭它们，以便它们的缓冲及时回收内存。它们的缓冲不仅存在于 java虚拟机内，还存在于java虚拟机外。如果我们仅仅是把它的引用设置为null,而不关闭它们，往往会造成内存泄漏。因为有些资源性对象，比如 SQLiteCursor(在析构函数finalize(),如果我们没有关闭它，它自己会调close()关闭)，如果我们没有关闭它，系统在回收它时也会关闭它，但是这样的效率太低了。因此对于资源性对象在不使用的时候，应该调用它的close()函数，将其关闭掉，然后才置为null.在我们的程序退出时一定要确保我们的资源性对象已经关闭。

程序中经常会进行查询数据库的操作，但是经常会有使用完毕Cursor后没有关闭的情况。如果我们的查询结果集比较小，对内存的消耗不容易被发现，只有在常时间大量操作的情况下才会复现内存问题，这样就会给以后的测试和问题排查带来困难和风险。

代码示例:

**修正前**

```
Cursor cursor = getContentResolver().query(uri...); 
 
if (cursor.moveToNext()) { 
 
... ... 
 
} 
```

**修正后**

```
Cursor cursor = null; 
 
try { 
 
cursor = getContentResolver().query(uri...); 
 
if (cursor != null &&cursor.moveToNext()) { 
 
... ... 
 
} 
 
} finally { 
 
if (cursor != null) { 
 
try { 
 
cursor.close(); 
 
} catch (Exception e) { 
 
//ignore this 
 
} 
 
} 
 
} 

```



## 频繁创建对象引起的内存泄漏

> 解决方案: 复用对象，避免过多的重复创建


### Adapter View

初始时ListView会从BaseAdapter中根据当前的屏幕布局实例化一定数量的 view对象，同时ListView会将这些view对象缓存起来。当向上滚动ListView时，原先位于最上面的list item的view对象会被回收，然后被用来构造新出现的最下面的list item。这个构造过程就是由getView()方法完成的，getView()的第二个形参View convertView就是被缓存起来的list item的view对象(初始化时缓存中没有view对象则convertView是null)。由此可以看出，如果我们不去使用 convertView，而是每次都在getView()中重新实例化一个View对象的话，即浪费资源也浪费时间，也会使得内存占用越来越大。

代码示例:

**修正前**

```

public View getView(int position, ViewconvertView, ViewGroup parent) { 
 
View view = new Xxx(...); 
 
... ... 
 
return view; 
 
} 

```

**修正后**

```
public View getView(int position, ViewconvertView, ViewGroup parent) { 
 
View view = null; 
 
if (convertView != null) { 
 
view = convertView; 
 
populate(view, getItem(position)); 
 
... 
 
} else { 
 
view = new Xxx(...); 
 
... 
 
} 
 
return view; 
 
} 

```




- 参考文献

1. [Android内存泄漏的八种可能](http://www.jianshu.com/p/ac00e370f83d)
2. [Android内存泄漏终极解决篇](https://www.douban.com/note/542644739/)
