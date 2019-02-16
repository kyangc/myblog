title: ADB(Android Debug Bridge)学习笔记
date: 2015-07-15 20:00:00
categories: 技术
tags: [Android,adb] 
description: 活了半辈子终于把ADB搞明白了……
---

ADB=`Andorid Debug Bridge`。看到过很多，平时也是在用，但是没有系统的了解过，这里稍微深入的了解下。特别是之后可能会用到ADB进行设备文件的修改，因此认真学习一下还是很有必要的。

<!-- more -->

主要内容还是在[Android Debug Bridge](http://developer.android.com/intl/zh-cn/tools/help/adb.html)官方文档里，这里只是个大概的记录。

`ADB`包含三个部分：

> Client，运行在电脑上，可以通过adb命令开启或使用ADT或DDMS开启。
> Server，运行在电脑上，处于后台，用于与手机或模拟器上的ADB客户端进行通信。
> Daemon，一个运行在模拟器或真机上的后台程序

当你开启ADB的时候，客户端首先将检查ADB Server是否已经运行，如果没有则将开始ADB Server，该Server会与本地的TCP端口5037进行绑定，并且监听来自于ADB Client、经过5037端口与设备进行通信的ADB命令。

Server之后开始扫描从5555~5585的**奇数**端口找到所有链接到的模拟器或真机，在那里可以找到设备或模拟器上的Daemon后台程序，而后Server与之建立双重连接——奇数端口建立与Daemon的链接，比该奇数端口小1的偶数端口与控制台Console链接。为了使用ADB，必须在手机上将允许USB调试打开。

ADB的命令格式如下：

```bash
adb [-d|-e|-s <serialNumber>] <command>
```

>`-d`意味着直接连接到唯一链接到电脑的设备。
>`-e`意味着直接连接到唯一连接到电脑的模拟器。
>`-s <serial number>`意味着直接连接到指定serial number的设备。（如emulator-5556）

ADB的命令有很多，下面简单的列举一下：

>`devices`列举连接设备
>`help`打印帮助信息
>`version`查看adb版本
>`logcat [option] [filter-specs]`打印log
>`bugreport`打印`dumpsys`、`dumpstate`、`logcat`
>`install <path-to-apk>`安装apk到设备
>`pull <remote> <local>`将特定的文件从设备复制到本地
>`push <local-path> <remote-path>`将特定的文件推送到设备中
>`forward <local-path> <remote-path>`在特定的本地、远端端口建立socket链接
>`get-serialno`获得adb客户端的serial number
>`get-state`获得adb客户端的adb状态
>`start-server`检查本地的adb server是否开启，如果没有则开启
>`kill-server`杀掉本地的adb server
>`shell [shell command]`在远端开启shell并且执行shell命令

ADB可以通过Wifi链接设备，步骤简单记录如下：

>*	保证设备和电脑在同一个Wifi连接中。
>*	将设备链接电脑
>*	保证设备以USB模式链接：

```bash
$ adb usb
restarting in USB mode

$ adb devices
List of devices attached
######## device
```

>*	以tcpip模式重启adb

```bash
$ adb tcpip 5555
restarting in TCP mode port: 5555
```

>*	找到设备的IP地址#.#.#.#，让adb链接到该设备

```bash
$ adb connect #.#.#.#
connected to #.#.#.#:5555
```

>* 拔出数据线，检查是否连接成功

```bash
$ adb devices
List of devices attached
#.#.#.#:5555 device
```

>如果链接丢失，那么回到第5步再次执行命令即可。