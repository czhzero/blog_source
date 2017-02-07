---
title: Android性能优化(三)－ 移动网络优化
date: 2017-02-07 22:17:48
categories: Android
tags: 性能优化
---

本篇文章介绍了移动端开发基本的网络优化方式，包括连接服务器策略，获取数据策略等等。

<!-- more -->

性能优化系列:

- [Android性能优化(一)－UI优化](http://www.czhzero.com/2017/02/07/performance-optimization-1/)
- [Android性能优化(一)－数据库优化](http://www.czhzero.com/2017/02/07/performance-optimization-2/)
- [Android性能优化(三)－ 移动网络优化](http://www.czhzero.com/2017/02/07/performance-optimization-3/)
- [Android性能优化(四)－ 代码优化](http://www.czhzero.com/2017/02/07/performance-optimization-4/)

## 连接服务器优化

- IP直连

不用域名，用IP直接连接，省去DNS解析时间。一般为了安全，这个ip会设置成动态Ip列表。

- 服务器合理部署

服务器多运营商多地部署，一般至少含三大运营商、南中北三地部署。

配合上面说到的动态 IP 列表，支持优先级，每次根据地域、网络类型等选择最优的服务器 IP 进行连接。

## 获取数据优化

- 连接复用

节省连接建立时间，如开启 keep-alive。

Http 1.1 默认启动了 keep-alive。对于 Android 来说默认情况下 HttpURLConnection 和 HttpClient 都开启了 keep-alive。只是 2.2 之前 HttpURLConnection 存在影响连接池的 Bug，具体可见：[Android HttpURLConnection 及 HttpClient 选择](http://www.trinea.cn/android/android-http-api-compare/)

- 请求合并

即将多个请求合并为一个进行请求，比较常见的就是网页中的 CSS Image Sprites。 如果某个页面内请求过多，也可以考虑做一定的请求合并。

具体业务的网络请求，能够合并的也尽量合并。

- 压缩数据

(1) 对于 POST 请求，Body 可以做 Gzip 压缩，如日志。[GZIP压缩与解压](http://www.cnblogs.com/tinyclear/p/6109792.html)。
 
(2) 对请求头进行压缩
这个 Http 1.1 不支持，SPDY 及 Http 2.0 支持。 Http 1.1 可以通过服务端对前一个请求的请求头进行缓存，后面相同请求头用 md5 之类的 id 来表示即可。

- CDN 缓存静态资源

缓存常见的图片、JS、CSS 等静态资源。

- 减小每次返回数据的大小

(1) 压缩
一般 API 数据使用 Gzip 压缩，下图是之前测试的 Gzip 压缩前后对比图。 

(2) 精简数据格式

如 JSON 代替 XML，WebP 代替其他图片格式。

(3) 对于不同的设备不同网络返回不同的内容 如不同分辨率图片大小。

(4) 增量更新

需要数据更新时，可考虑增量更新。如常见的服务端进行 bsdiff，客户端进行 bspatch。

(5) 大文件下载
支持断点续传，并缓存 Http Resonse 的 ETag 标识，下次请求时带上，从而确定是否数据改变过，未改变则直接返回 304。

- 数据缓存

Http缓存获取到的数据，在一定的有效时间内再次请求可以直接从缓存读取数据。

## 其他方案

- 预取
包括预连接、预取数据。
 
- 分优先级、延迟部分请求
将不重要的请求延迟，这样既可以削峰减少并发、又可以和后面类似的请求做合并。
 
- 多连接
对于较大文件，如大图片、文件下载可考虑多连接。 需要控制请求的最大并发量，毕竟移动端网络受限。

- wifi传输

wifi比蜂窝数据，包括2G(GPRS)、3G更省电)尽量在Wi-Fi下传输数据。


