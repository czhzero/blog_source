---
title: Android性能优化(二)－ 数据库性能优化
date: 2017-02-07 21:50:36
categories: Android
tags: 性能优化
---

本文为性能优化的第二篇——数据库性能优化，主要包括索引，事务，异步线程等几个方面。

<!-- more -->

性能优化系列:

- [Android性能优化(一)－UI优化](http://www.czhzero.com/2017/02/07/performance-optimization-1/)
- [Android性能优化(一)－数据库优化](http://www.czhzero.com/2017/02/07/performance-optimization-2/)
- [Android性能优化(三)－ 移动网络优化](http://www.czhzero.com/2017/02/07/performance-optimization-3/)
- [Android性能优化(四)－ 代码优化](http://www.czhzero.com/2017/02/07/performance-optimization-4/)

## 索引

简单的说，索引就像书本的目录，目录可以快速找到所在页数，数据库中索引可以帮助快速找到数据，而不用全表扫描，合适的索引可以大大提高数据库查询的效率。

优点是加快了数据库点检索速度，但是缺点也很明显，索引但创建和维护存在消耗，索引会占用物理空间，且随着数据量的增加而增加。在对数据库进行增删改时需要维护索引，所以会对增删改的性能存在影响。


## 事务

使用事务的两大好处是原子提交和更优性能。

Sqlite默认会为每个插入、更新操作创建一个事务，并且在每次插入、更新后立即提交。
这样如果连续插入100次数据实际是创建事务->执行语句->提交这个过程被重复执行了100次。如果我们显示的创建事务->执行100条语句->提交会使得这个创建事务和提交这个过程只做一次，通过这种一次性事务可以使得性能大幅提升。尤其当数据库位于sd卡时，时间上能节省两个数量级左右。

```
public void insertWithOneTransaction() {
    SQLiteDatabase db = sqliteOpenHelper.getWritableDatabase();
    // Begins a transaction
    db.beginTransaction();
    try {
        // your sqls
        for (int i = 0; i < 100; i++) {
            db.insert(yourTableName, null, value);
        }

        // marks the current transaction as successful
        db.setTransactionSuccessful();
    } catch (Exception e) {
        // process it
        e.printStackTrace();
    } finally {
        // end a transaction
        db.endTransaction();
    }
}

```

## 异步线程

Android中数据不多时表查询可能耗时不多，不会导致anr，不过大于100ms时同样会让用户感觉到延时和卡顿，可以放在线程中运行，但sqlite在并发方面存在局限，多线程控制较麻烦，这时候可使用单线程池，在任务中执行db操作，通过handler返回结果和ui线程交互，既不会影响UI线程，同时也能防止并发带来的异常。

```

ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();
singleThreadExecutor.execute(new Runnable() {
 
	@Override
	public void run() {
		// db operetions, u can use handler to send message after
		db.insert(yourTableName, null, value);
		handler.sendEmptyMessage(xx);
	}
});

```

## 其他优化

- 有些能用文件操作的，尽量使用文件操作，文件操作的速度比数据库的操作要快10倍左右。

- Cursor的使用，管理好cursor，不要每次打开关闭cursor，因为打开关闭cursor非常耗时。Cursor.require用于刷cursor。同时由于SQLiteDatabase对象较为耗费资源，所以我们在使用完SQLiteDatabase对象之后，必须立即关闭它，避免它继续占用资源，否则我们继续程序可能会导致OOM或者其他异常。

- 查询时返回更少的结果集及更少的字段。

- 少用cursor.getColumnIndex(可以在建表的时候用static变量记住某列的index，直接调用相应index而不是每次查询。)

- 优化sql语句字符串等，语句的拼接使用StringBuilder代替String。



## 参考文献

- [性能优化之数据库优化](http://www.trinea.cn/android/database-performance/)
- [Android性能优化 浅析](http://blog.csdn.net/wtyvhreal/article/details/44172125)

