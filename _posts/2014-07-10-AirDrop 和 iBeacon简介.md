---
layout: post
title: "AirDrop 和 iBeacon简介"
date: 2014-07-10
comments: false
categories: 知识囊
---
##AirDrop简介
- 苹果在`2010`推出的`OS X 10.7 Lion系统`中加入了全新的AirDrop功能，该功能`允许两台Mac机之间无线传输文件`。
区别于传统的局域网文件共享方式，AirDrop不要求两台机器在同一个网络内。
用户无需设置，只需要打开AirDrop文件夹即可查看到其他用户，分享文件变得非常便捷。

- AirDrop不需要基于（无线）路由器或者手动建立热点组网，它是利用Mac与Mac之间的点对点网络来进行会话传输。
这一切由系统在后台完成，无需断开当前WiFi网络，也不影响当前连接WiFi网络的通信，就可以与其他Mac通过内置特定信道通信。

- WWDC13上推出的iOS7也开始支持iOS设备之间使用AirDrop实现共享传输。
关于AirDrop的条件要求及内部机制，可参考[《为什么iOS 7 和 OS X 之间的AirDrop 不能互传？》](http://www.zhihu.com/question/21681429)。
WWDC14推出的OS X 10.10 Yosemite操作系统，终于打通了与iOS移动设备之间的[跨平台AirDrop传输](https://support.apple.com/zh-cn/HT204144)。
运行Mac OS X Yosemite 10.10版本的Mac设备（型号≥2012）和运行iOS 7及以上的iOS设备（≥iPhone5，≥iPad 4，iPad mini，≥iPod touch）之间才能实现跨平台文件传输。

- 根据官方资料显示，AirDrop基于蓝牙和WiFi实现（AirDrop does the rest using Wi-Fi and Bluetooth）。
具体来说，通过低功耗蓝牙技术（[BLE](http://www.warski.org/blog/2014/01/how-ibeacons-work/)）进行发现（Advertising/Browsing），使用[WiFi Direct（P2P WiFi）](http://www.xuebuyuan.com/539020.html)技术进行数据传输。
可参考《[iOS 7的AirDrop是利用什么信号来传输的？](http://www.zhihu.com/question/21189545)》《[What Is AirDrop? How Does It Work?](http://ipad.about.com/od/iPad_Guide/ss/What-Is-Airdrop-How-Does-It-Work.htm)》。

- 因此，开启AirDrop不要求双方必须联网或连接到同一局域网，但必须`同时打开WiFi和蓝牙`，且进行传输的两台设备必须保持在`9米`的范围之内。

## iBeacon简介
- iBeacon起源:苹果在WWDC2013上正式推出了iBeacon,并且在iOS7设备商配置了该功能
- iBeacon应用:苹果期望将其作为一种技术标准,这个标准允许移动App(包括iOS和Android设备)监听来自于iBeacon设备上的信号并作出响应.
- iBeacon设备:配备有BLE通信功能,并使用BLE向周围发送自己特有的ID,移动设备上的App在接收到该ID后可以作出相应的反应.比如,我们在店铺里设置iBeacon发射器,便可以让应用接收到信息并将这一信息通知给服务器,服务器向我们的App返回与该店铺相关的产品或折扣信息.
- 本质上讲,iBeacon技术允许App了解他们在某个局部范围内的位置,并向用户分发基于位置的超文本上下文内容.

