---
layout: post
title: "CocoaPods的简介与使用"
date: 2014-05-13
comments: false
categories: 实用技术
---


- CocoaPods 是什么?
    + CocoaPods 是开发 OS X 和 iOS 应用程序的一个第三方库的依赖管理工具。利用 CocoaPods,可以定义自己的依赖关系 (称作 pods),并且随着时间的变化,以 及在整个开发环境中对第三方库的版本管理非常方便

- CocoaPods 背后的理念主要体现在两个方面
    + 在工程中引入第三方代码 会涉及到许多内容。针对 Objective-C 初级开发者来说,工程文件的配置会让 人很沮丧
    + 在配置buildphases和linker flags过程中,会引起许多人为因素的 错误
    + CocoaPods 简化了这一切,它能够自动配置编译选项

- CocoaPods的原理
    + 它是将所有的依赖库都放到另一个名为Pods项目中,然后 让主项目依赖Pods项目,这样,源码管理工作都从主项目移到了Pods项目中
    + 1、Pods项目最终会编译成一个名为libPods.a的文件,主项目只需要依赖这个.a 文件即可。
    + 2、对于资源文件,CocoaPods提供了一个名为Pods-resources.sh的bash脚本, 该脚本在每次项目编译的时候都会执行,将第三方库的各种资源文件复制到目 标目录中。
    + 3、CocoaPods通过一个名为Pods.xcconfig的文件来在编译时设置所有的依赖和 参数。

---
- CocoaPods安装
    + 更新gem
        * sudo gem update --system
    + 更新ruby的软件源
        * gem sources --remove https://rubygems.org/
        * gem sources -a http://ruby.taobao.org/
        * gem sources -l
    + 安装CocoaPods
        * 10.11 cocoapods安装 sudo gem install -n /usr/local/bin cocoapods
        * sudo gem install cocoapods
    + 替换CocoaPods的镜像索引
        * pod repo remove master
        * pod repo add master http://git.oschina.net/akuandev/Specs.git  (二选一)
        *  pod repo add master https://gitcafe.com/akuandev/Specs.git
        * pod repo update
    + 设置 pod 仓库
        * pod setup
    + 测试
        * pod --version

- 卸载CocoaPods
    + sudo gem uninstall cocoapods

---

###终端使用CocoaPods技巧:



- 来到项目所在文件夹 :

    - `cd /Users/ZHUNJIEE/Documents/SourceTree/MyPrograms/抽屉效果`

- 在项目目录里新建一个名为**Podfile**的文件:

    - `touch Podfile`

- 打开并进入Podfile文件:

        - $ vim Podfile

        - (i 回车)进入插入模式开始写入

- 在打开的文件中输入以下指令:

    - `platform : ios`

    - `pod '框架名称'`

- 保存退出:

        - (ESC)

        - :wq

- 开始安装插件:    

    - `pod install`
    
---
- 注释事项
    + 1.利用CocoPods管理类库后, 以后打开项目就用xxxx.xcworkspace 打开,而不是 之前的.xcodeproj文件
    + 2.每次更改了Podfile文件,你需要重新执行一次pod update命令。
    + 3.CocoaPods在执行pod install和pod update时,会默认先更新一次CocoPods的 spec仓库索引。使用--no-repo-update参数可以禁止其做索引更新操作

```
pod install --no-repo-update
pod update --no-repo-update
```


## CocoaPods安装失败
- CocoaPods安装东西的时候它要找到Xcode的Developer文件夹, 如果找不到会报如下错误
![](https://dn-zhunjiee.qbox.me/Snip20150907_5.png)

- 解决方案
    + LNJ替换为你自己的用户名
```objc
sudo xcode-select --switch /Users/LNJ/Applications/Xcode.app/Contents/Developer
```

![](https://dn-zhunjiee.qbox.me/Snip20150907_6.png)
![](https://dn-zhunjiee.qbox.me/Snip20150907_7.png)



###Xcode中cocoapods插件
github中有大牛写的cocoapods相关的插件,非常diao(直接在github中搜索cocoapods第一个就是),可以更好地提高我们开发的效率,安装此cocoapods插件以后我们直接通过Xcode写入要安装的插件然后安装就可以了,再也不用打开终端进行麻烦的操作,具体使用步骤就不做介绍了,github里也有相应的介绍

关于10.11.xx 系统以后cocoapods插件的安装注意(更新): 

- 很多Mac OS X 系统升到10.11.xx以后的同事会发现  
    - 终端里:$ gem env
    - 找到SHELL PATH
    - 你会发现多了一个/usr/local/bin
    - 修改Xcode的cocoapods插件里GEM_PATH: `/usr/local/bin`,如下图所示

![](https://dn-zhunjiee.qbox.me/Snip20151213_1.png)
