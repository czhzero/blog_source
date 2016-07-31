---
title: 解读Android6.0新功能和Api变更
date: 2016-07-31 18:59:35
categories: Android
tags: Android 6.0
---

从 Android A 到 Android M，Android 的每个甜品的变化的细节里都是新版本的改进。上一个大版本 Android 5.0（也就是 Android L）为我们带来了 Material Design 清爽但不失细节的设计风格，从此原生 Android 的颜值终于在舆论中得到了普遍赞誉；而在 Android 6.0 中，我们得到的更多是内在的提升。

新功能在使用体验上提升的效果是非常明显的，内在体验优化的意义不亚于全新的设计语言带来的新鲜感与友好度的提升。从外表到内在，我们看到的是一个更为成熟的 Android 系统。

> 如果你之前已经发布过 Android 应用程序，要意识到这些变更对应用程序的影响。
> 

<!-- more -->

# Runtime Permissions (运行时权限)

这个版本中引入了新的权限模型，现在用户可以在运行时直接管理应用程序的权限。这个模型基于用户对权限控制的更多可见性，同时为应用程序的开发者提供更流畅的应用安装和自动升级。用户可以为已安装的每个应用程序独立的授予或者取消权限。

在运行于目标版本 Android 6.0 (API 级别 23) 及以上的应用程序中，必须在运行时检查并请求权限。通过新的 checkSelfPermission() 方法来确定你的应用程序是否已经被授权。通过新的 requestPermissions() 方法来请求权限。即使你的应用程序运行的目标版本不是 Android 6.0 (API 级别 23)，你也应该在新的授权模型下来测试应用程序。

![解读Android6.0新功能和Api变更](http://o7y1sf21i.bkt.clouddn.com/blog/7/Android-m-4.jpg)


# Doze and App Standby(休眠与应用挂起)

这个版本为空闲的设备和应用程序引入了电源节能优化。这个特性将影响所有应用程序，故确保自己的应用程序在这些新模式下进行测试。

- 休眠：如果用户将设备拔下，并将其静置，关闭屏幕，经过一段时间，设备将进入休眠模式。这时候，设备试图让系统保持在一个睡眠的状态。这种模式下，设备周期性的恢复平常的操作，以便应用程序同步，系统则可以处理一些延时的操作。

- 应用挂起：应用挂起则允许系统当用户不再使用应用程序时，将其定义为空闲。当用户经过一段时间没有触摸应用程序时，系统可以做这个决定。设备被拔线时，系统禁用网络访问，停止应用程序的同步及操作，则被认为是空闲。


# Apache HTTP Client Removal (移除 Apache HTTP 客户端)

Android 6.0 发布版移除了对 Apache HTTP 客户端的支持。如果你的应用程序使用该客户端，并且目标运行版本为 Android 2.3 (API 级别9) 及以上，需要使用 HttpURLConnection 类来代替。这个 API 更加的高效，因为它通过对用户透明的压缩、响应缓存来减少网络开销，并最小化电量消耗。要继续使用 Apache HTTP 的 API，你需要在 build.gradle 文件中声明下面的编译期依赖：

```
android {
    useLibrary 'org.apache.http.legacy'
}

```

# BoringSSL

Android 从 OpenSSL 转移到了 BoringSSL 库。如果你在应用程序中使用 Android NDK，千万不要将加密库链接到除 NDK API 之外的任何库，如 libcrypto.so 和 libssl.so。这些库不是公开 API，有可能在没有收到通知的情况下在发布版和设备间发生变更或中断。这种情况你将把自己暴露在安全威胁下。你应该修改自己的本地代码来通过 JNI 调用 Java 加密 API 或者静态链接你选择的一个加密库。

# Access to Hardware Identifier (访问硬件标识符)

为了给用户更多的数据保护，从这个版本开始， Android 移除了通过 WiFi 和蓝牙 API 来在应用程序中可编程的访问本地硬件标示符。现在 WifiInfo.getMacAddress() 和 BluetoothAdapter.getAddress() 方法都将返回 02:00:00:00:00:00 常量。

要通过蓝牙和 WiFi 扫描来访问附近外部设备的硬件标示符，应用程序需要 ACCESS_FINE_LOCATION 和 ACCESS_COARSE_LOCATION 权限：

- WifiManager.getScanResults())
- BluetoothDevice.ACTION_FOUND
- BluetoothLeScanner.startScan())

> 注意：在一个运行 Android 6.0 (API 级别 23) 的设备初始化后台的 WiFi 或蓝牙扫描时，操作对于外部设备是可见的，且被赋予一个随机的 MAC 地址。
> 

# Notifications (通知)

这个版本移除了 Notification.setLatestEventInfo() 方法。使用 Notification.Builder 类来代替构造方法。要重复的更新通知，要重用 Notification.Builder 实例。调用 build() 方法来获取更新过的 Notification 实例。

adb shell dumpsys nnotification 命令不在答应通知文本。使用 adb shell dumpsys notification --noredcat 命令来在同志对象中打印文本。

# AudioManager Changes (AudioManager 变更)

通过 AudioManager 类来直接设置音量或者使流静音已经不再支持。 setStreamSolo() 方法被弃用，你需要调用 requestAudioFocus() 来代替。类似的， setStreamMute() 方法被弃用，替换为 adjustStreamVolume() 方法并传递方向值 ADJUST_MUTE 或 ADJUST_UNMUTE。

# Text Selection (文本选择)

当用户在应用程序中选择文本时，你可以在悬浮工具栏中显示文本选择工作，如剪切、复制、粘贴。用户交互实现与为独立视图启动上下文动作模式中描述的上下文动作栏类似。

为文本选择实现悬浮工具栏，需要在已存在的应用程序中做如下修改：

- 在 View 或者 Activity 对象中，通过修改 startActionMode(Callback) 为 startActionMode(Callback, ActionMode.TYPE_FLOATING) 来改变 ActionMode。

- 使已经存在的 ActionMode.Callback 的实现继承自 ActionMode.Callback2 。

- 重载 onGetContentRect() 方法来提供内容 Rect 对象(如文本选择矩形)在视图中的坐标。

- 如果矩形位置不在有效，并且这是需要刷新的唯一元素，则调用 invalidateContentRect() 方法。

![解读Android6.0新功能和Api变更](http://o7y1sf21i.bkt.clouddn.com/blog/7/text-selection.gif)


如果你在使用 Android 22.2 修订版的兼容包，悬浮工具栏不是向后兼容的，且使用默认的 ActionMode 对象。这导致悬浮工具栏无法显示。在 AppCompatActivity 中启用 ActionMode，首先调用 getDelegate() ，然后在返回的 AppCompatDelegate 对象中调用 setHandleNativeActionModesEnabled()，并设置输入参数为 false。这个调用为框架返回可控的 ActionMode 对象。在运行 Android 6.0 (API 级别 23) 的设备上，这允许框架支持 ActionBar 或悬浮工具栏模式。在 Android 5.1 (API 级别 22) 或更低版本，只有 ActionBar 是支持的。

# Browser Bookmark Changes (浏览器书签变更)

这个版本移除了对全局书签的支持。 android.provider.Browser.getAllBookmark() 和 android.provider.Browser.saveBookmark() 方法被移除。同样的， READ_HISTORY_BOOKMARKS 和 WRITE_HISTORY_BOOKMARKS 权限被移除。如果你的应用程序的目标运行版本为 Android 6.0 (API 级别 23) 或者更高，不要从全局提供者访问书签或者使用书签权限。现在，你的应用程序需要内部保存书签数据。

# Android Keystore Changes (密钥变更)

在这个版本中， Android Keystore 提供器 不在支持 DSA。 ECDSA 则仍然被支持。

当安全锁屏被禁用或重置时，不要求加密的 key 将不再被删除。要求加密的 key 则会在这些事件中被删除。

# Wi-Fi and Networking Changes (Wi-Fi和网络变更)

这个版本为 Wi-Fi 和 网络 API 引入了下面的行为变更：

- 应用程序只有在创建了 WifiConfiguration 对象以后，才能更变这些对象的状态。当 WifiConfiguration 被用户或者其他应用程序创建时，你将不允许修改和删除这些 WifiConfiguration 对象。

- 在之前的版本中，如果应用程序使用 enableNetwork() ，并设置 disableAllOthers=true 来强制设备连接到指定的 Wi-Fi 网络，设备将和其他网络断开。这个版本中，这些设备将不再和其他网络断开。如果应用程序的 targetSdkVersion 是 20 或者更低，则会连接被选中的 Wi-Fi 网络。如果应用程序的 targetSdkVersion 是 21 或者更高，使用多网络 API (如 openConnection()， bindSocket() 及新的 bindProcessToNetwork() 方法)来确保它的网络通信是发给被选中的网络。

# Camera Service Changes (摄像头服务变更)

在这个版本中，在摄像头服务中访问共享资源的模式发生了变更，不再是以前的”先到先得”，而是具备高优先级的将优先处理。服务行为的变更包括：

- 访问摄像头子系统的资源，包括打开和配置摄像设备，依赖于客户端应用进程的优先级。用户可见或者在前台活动的应用程序进程往往具备更高的优先级，使得摄像资源更易获得，也更加可靠。

- 当更高优先级的应用程序视图使用摄像头时，具有低权限的活动摄像头客户端应用程序可能被中断。在被弃用的 Camera API 中，错误会在被中断客户端的 onError() 中被调用。在 Camera2 API 中，错误结果则在 onDisconnected() 中被调用。

- 在具备适合摄像头硬件的设备上，不同的应用进程可以同时独立地打开和使用独立的摄像头设备。虽然如此，多进程使用时，同时访问摄像头会造成设备性能的急剧下降，这将被摄像头服务所检测到并不被允许。这个变更使得由于较低优先级而被中断的客户端，即使当没有其他应用直接访问时，也会试图访问一样的设备。

- 改变当前用户会导致之前用户账号的应用程序中活动那个的摄像头客户端被中断。对摄像头的访问是被当前设备用户所限制的。实际上，这意味着当用户切换到不同的账户下时，原来的”访客”账号所使用的摄像头子系统是不可能继续运行的。

# Runtime (运行时)

通过 newInstance() 方法 ART 运行时现在正确的实现了访问规则。这个变更修复了之前版本中 Dalvik 检查访问规则时的错误。如果你的应用程序使用 newInstance() 方法，且你想要覆盖访问检查，调用 setAccessible() 方便，并设置参数为 true。如果你的应用程序使用 v7 兼容库 和 v7 recyclerview 库。你需要升级应用程序中的相关库到最新版本。否则，需要确保 XML 中所引用的自定义类已经被升级，其构造方法是可访问的。

这个版本升级了动态链接器的行为。动态连接器现在可以理解库的 soname 和 它的路径之间的区别，且实现了通过 soname 来搜索。在加载时，之前可用的应用程序可能会被提示具有不可用的 DT_NEEDED 条目(通常是在构建机器文件系统中的绝对路径)。

dlopen(3) RTLD_LOCAL 标志现在被正确实现了。 RTLD_LOCAL 是默认的，因此调用 dlopen(3) 是不明确使用 RTLD_LOCAL 是有效的(除非应用之前有明确使用 RTLD_GLOBAL )。使用 RTLD_LOCAL ，标记在调用 dlopen(3) 加载库之前是不可用的(这与被 DT_NEEDED 条目引用恰恰相反)。

在之前版本的 Android，如果你的应用请求系统来加载包含文本重定位段的动态链接库，系统会显示警告，但允许继续加载库。从这个版本开始，如果你的目标运行 SDK 版本为 23 或以上，系统会拒绝这个库。为了辅助检测库是否被成功加载，应用程序需要为 dlopen(3) 失败添加日志，并在 dlerror(3) 返回值中包含问题的描述文本。学习更多关于如何处理文本重定位段，可以查看这个指南。


# APK 验证 (APK Validation)

Android 平台现在执行更加严格的 APK 验证。如果一个文件在清单文件中被声明，但在 APK 中却没有，那么这个 APK 被认为是无效的。如果任意的内容被移除， APK 需要重新签名。

# USB 连接 (USB Connection)

通过 USB 端口的设备连接现在默认被设置为充电模式。想要通过 USB 连接来访问设备和它的内容，用户需要为这些交互提供明确的授权。如果你的应用程序支持用户通过 USB 端口与设备交互，需要确保这些交互被明确启用。

# Android for Work Changes (Android for Work 变更)

这个版本包括下面的 Android for Work 行为变更：

- 个人上下文中的联系人。Google 拨号器通话记录现在可以在用户查看已通话记录时显示当前联系人。在 Google 拨号器中通过设置 setCrossProfileCallerIdDisabled() 为 true 来隐藏当前联系人。当设置 setBluetoothContactSharingDisabled() 为 false 时，当前联系人能通过蓝牙显示在设备联系人中。默认情况下，其设置为 true。

- 移除 Wi-Fi 配置：由外部拥有者添加(如通过 addNetworkd() 方法)的 Wi-Fi 配置现在在当前 profile 被删除时也将被移除。

- 紧闭 Wi-Fi 配置：如果 WIFI_DEVICE_OWNER_CONFIGS_LOCKDOWN 为非空时，任何由活动的设备拥有者创建的 Wi-Fi 配置将不能用户被修改和删除。用户可以创建和修改自己的 Wi-Fi 配置。活动的设备拥有者拥有编辑和移除任意 Wi-Fi 配置的特权，包括不是由他们创建的配置。

- 通过 Google 账号下载使用策略控制器：当一个要求 WPC 应用程序来管理的 Google 账号被添加到管理上下文之外的设备中时，添加账号流程会提示用户安装合适的 WPC。这些行为也可以应用到通过在初始设备创建向导中 设置 > 账号 添加的账号。

- 制定 DevicePolicyManager API 行为变更：

	- 调用 setCameraDisabled() 方法来影响当前调用用户的摄像头。
	
	- 此外， setKeyguardDisabledFeatures() 方法对 Profile 用户是可用的，与设备拥有者一样。
	
	- Profile 拥有者可以设置键盘守卫的约束：
		- KEYGUARD_DISABLE_TRUST_AGENTS 和 KEYGUARD_DISABLE_FINGERPRINT 将影响到 profile 父用户的键盘守卫设置。
		- KEYGUARD_DISABLE_UNREDACTED_NOTIFICATIONS 只会影响在 profile 中由应用程序生成的通知。

	- createAndInitializeUser() 和 createUser() 方法已经被弃用。

	- 当给定用户的应用程序在前台运行时， setScreenCaptureDisabled() 方法将阻塞辅助结构。
	SHA-256 的默认值为 
	
	- EXTRA_PROVISIONING_DEVICE_ADMIN_PACKAGE_CHECKSUM 。为保持兼容性， SHA-1 仍然被支持，但在将来会被移除。 EXTRA_PROVISIONING_DEVICE_ADMIN_SIGNATURE_CHECKSUM 则仅接受 SHA-256。
	
	- 原来存在的设备初始化 API 在 Android 6.0 (API 级别 23) 中被移除了。
	
	- EXTRA_PROVISIONING_RESET_PROTECTION_PARAMETERS 被移除，因此 NFC bump provisioning 无法通过编程的方式解锁重置被保护的设备。
	
	- 现在在被管理的设备中通过 NFC 可以使用 EXTRA_PROVISIONING_ADMIN_EXTRAS_BUNDLE 来传递数据给设备拥有者的应用程序。
	
	- Android M 上的 Android for Work API 是经过优化的，包括 Work 配置，辅助层及其他。新的 DevicePolicyManager 权限 API 不会影响 Android 之前版本的应用程序。
	
	- 当用户通过 ACTION_PROVISION_MANAGED_PROFILE 或者 ACTION_PROVISION_MANAGED_DEVICE 意图，从创建流程中的同步部分返回时，系统将返回 RESULT_CANCELED 结果码。

- 其他 API 变更：

	- 数据用法： android.app.usage.NetworkUsageStates 类重命名为 NetworkStats。

- 全局设置变更：

	- 下面的设置不能继续通过 setGlobalSettings() 方法设置：
		- BLUETOOTH_ON
		- DEVELOPMENT_SETTINGS_ENABLED
		- MODE_RINGER
		- NETWORK_PREFERENCE
		- WIFI_ON
		
	- 下面全局设置可以通过 setGlobalSettings() 方法设置：
		- WIFI_DEVICE_OWNER_CONFIGS_LOCKDOWN


# 参考文献
- [Android 6.0 Changes](https://developer.android.com/about/versions/marshmallow/android-6.0-changes.html#behavior-afw)
- [Android 6.0 新功能及主要 API 变更](http://www.linfuyan.com/android-m-changes/)
- [解读 Android 6.0 如何让你甜到心窝](http://www.ifanr.com/app/569615)
