---
layout: post
title: "UIApplication,UIWindow,控制器view的创建及生命周期"
date: 2014-04-07
comments: false
categories: UI
---

#项目常见文件

###项目结构区别(Xcode5和Xcode6)
- 1.xcode6没有Frameworks文件夹,xcode6内部会自动帮你导入一些常见的框架.
- 2.xcode6多了LaunchScreen.xib,用于设置启动界面,而且可以确定模拟器或者真机的真实尺寸,如果没有设置,默认4s的尺寸(320 x 480)
- 3.模拟器或者真机的真实尺寸是由启动界面确定的
- 4.xcode6没有pch文件

###info.plist
- Bundle name : 应用程序名称
- Bundle versions string,short : 应用程序版本号
- Bundle identifier : 应用程序唯一标识符,用于上传AppStore和推送

###.pch文件
- 原理:会把pch里面的所有内容导入到每个文件的头部
- 需要提前预编译
    - `TARGETS - Build Settings - All - 搜索 prefix header`

- 作用:
    - pch中存放一些公用的宏
    - 存放公用的头文件,pch头文件的内容能被项目中的其他所有源文件共享和访问
    - 可以自定义Log
        + 在pch文件中添加下列预处理指令，然后在项目中使用Log(…)来输出日志信息，就可以在发布应用的时候，一次性将NSLog语句移除（在调试模式下，才有定义DEBUG）
       
       ```objc
        #ifdef DEBUG // 表示当前为调试阶段
        #define Log(...) NSLog(__VA_ARGS__)
        #else // 发布阶段
        #define Log(...)
        #endif
        ```
- 注意点:
    + 需要判断当前文件是否是OC文件,是OC文件才可以导入
    + `所有的OC文件都会定义__OBJC__这个宏,苹果定义`
    + 使用pch文件的正确格式如下:
    
```objc
#ifdef __OBJC__

#ifdef DEBUG // 表示当前为调试阶段
#define Log(...) NSLog(__VA_ARGS__)
#else // 发布阶段
#define Log(...)
#endif

#endif
```


#UIApplication
###什么是UIApplication
- UIApplication对象是应用程序的象征

- 每一个应用都有自己的UIApplication对象，而且是单例的

- 通过[UIApplication sharedApplication]可以获得这个单例对象

- 一个iOS程序启动后创建的第一个对象就是UIApplication对象

- 利用UIApplication对象，能进行一些应用级别的操作

###UIApplication的常用属性

- 设置应用程序图标右上角的红色提醒数字
@property(nonatomic) NSInteger applicationIconBadgeNumber;

- 设置联网指示器的可见性
@property(nonatomic,getter=isNetworkActivityIndicatorVisible) BOOL networkActivityIndicatorVisible;

####openURL:
- UIApplication有个功能十分强大的openURL:方法

```objc
    - (BOOL)openURL:(NSURL*)url;
```
openURL:方法的部分功能有:

- 打电话

```objc
UIApplication *app = [UIApplication sharedApplication];
[app openURL:[NSURL URLWithString:@"tel://10086"]];
```
- 发短信

```objc
[app openURL:[NSURL URLWithString:@"sms://10086"]];
```
- 发邮件

```objc
[app openURL:[NSURL URLWithString:@"mailto://12345@qq.com"]];
```
- 打开一个网页资源

```objc
[app openURL:[NSURL URLWithString:@"http://ios.itcast.cn"]];
```
- 打开其他app程序

##UIApplicationDelegate

- 每次新建完项目，都有个带有“AppDelegate”字眼的类，它就是UIApplication的代理
- AppDelegate默认已经遵守了UIApplicationDelegate协议，已经是UIApplication的代理

```objc
// 应用程序的生命周期
// 应用程序启动完成的时候调用
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    NSLog(@"%s",__func__);
    return YES;
}

// 当我们应用程序即将失去焦点的时候调用
- (void)applicationWillResignActive:(UIApplication *)application {
    NSLog(@"%s",__func__);
}

// 当我们应用程序完全进入后台的时候调用
- (void)applicationDidEnterBackground:(UIApplication *)application{
   NSLog(@"%s",__func__);
}

// 当我们应用程序即将进入前台的时候调用
- (void)applicationWillEnterForeground:(UIApplication *)application {
  NSLog(@"%s",__func__);}

// 当我们应用程序完全获取焦点的时候调用
// 只有当一个应用程序完全获取到焦点,才能与用户交互.
- (void)applicationDidBecomeActive:(UIApplication *)application {
    NSLog(@"%s",__func__);
}

// 当我们应用程序即将关闭的时候调用
- (void)applicationWillTerminate:(UIApplication *)application {
   NSLog(@"%s",__func__);
}
```
---
#程序启动原理
一.`首先找到程序入口,执行main函数`

main -> UIApplicationMain

二.`UIApplicationMain底层做事情`

1.创建UIApplication对象

2.创建UIApplication的代理对象,而且给UIApplication对象代理属性赋值

3.开启主运行循环,作用接收事件,让程序一直运行

4.加载info.plist,判断下有木有指定main.storyboard,如果指定就会去加载

###UIApplicationMain
三.`函数介绍`:

```objc
int UIApplicationMain(int argc, char *argv[], NSString *principalClassName, NSString *delegateClassName);
```
- 变量说明
    - `argc、argv`：直接传递给UIApplicationMain进行相关处理即可

    - `principalClassName`：指定应用程序类名（app的象征），该类必须是UIApplication(或子类)。如果为nil,则用UIApplication类作为默认值

    - `delegateClassName`：指定应用程序的代理类，该类必须遵守UIApplicationDelegate协议

#UIWindow
- 添加UIView到UIWindow中两种常见方式:

```objc
- (void)addSubview:(UIView *)view;
```
- 直接将view添加到UIWindow中，但并不会理会view对应的UIViewController

```objc
@property(nonatomic,retain) UIViewController *rootViewController;
```
- 自动将rootViewController的view添加到UIWindow中，负责管理rootViewController的生命周期

###常用方法
`- (void)makeKeyWindow;`
- 让当前UIWindow变成keyWindow（主窗口）

`- (void)makeKeyAndVisible;`
- 让当前UIWindow变成keyWindow，并显示出来

###UIWindow的获得
- `[UIApplication sharedApplication].windows`
    - 在本应用中打开的UIWindow列表，这样就可以接触应用中的任何一个UIView对象
(平时输入文字弹出的键盘，就处在一个新的UIWindow中)

- `[UIApplication sharedApplication].keyWindow`
    - 用来接收键盘以及非触摸类的消息事件的UIWindow，而且程序中每个时刻只能有一个UIWindow是keyWindow。如果某个UIWindow内部的文本框不能输入文字，可能是因为这个UIWindow不是keyWindow

- `view.window`
    - 获得某个UIView所在的UIWindow
 
---

#控制器的View的创建
- **创建的六种方式**：
	1. 没有同名xib和storyboard情况下, 会自动创建一个空白的view做为控制器的veiw
	2. 通过 storyboard 创建,会创建箭头指向view做为控制器的veiw
	3. 有指定xib情况下创建
	4. 有同名xib情况
	5. 有同名去掉controll的情况
	6. 重写控制器的loadveiw方法

```objc
 #import "NJAppDelegate.h"
 #import "NJViewController.h"
 /*
  1.没有同名xib情况下
  2.通过 storyboard 创建
  3.有指定xib情况下创建
  4.有同名xib情况
  5.有同名去掉controll的情况
  6.loadveiw
  */
 @implementation NJAppDelegate
 
 - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
 {
     // 创建UIWindow
     self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
     self.window.backgroundColor = [UIColor whiteColor];
    

 1.第一种方式:没有xib和storyboard
      // (如果没有xib和storyboard, 会自动创建一个空白的view做为控制器的veiw)
      NJViewController *vc = [[NJViewController alloc] init];

     

2.通过 storyboard 创建
      // 如果通过storyboard创建, 会创建箭头指向view做为控制器的veiw
      
      // 如果重写了控制器的loadview方法, 就不会创建storyboard中描述的view作为控制器的view, 而是创建一个空白的veiw做为控制器的veiw
      UIStoryboard *storyboard = [UIStoryboard storyboardWithName:@"Test" bundle:nil];
      NJViewController *vc = [storyboard instantiateInitialViewController];


3.有指定xib情况下创建
      // 如果通过xib, 会创建xib中描述的veiw做为控制器的veiw
      NJViewController *vc = [[NJViewController alloc] initWithNibName:@"One" bundle:nil];

     

4.有同名xib情况
      // 如果有同名的xib, 会自动找到同名xib中描述的view做为控制器的veiw
      NJViewController *vc = [[NJViewController alloc] init];

     

5.有同名去掉Controller的 xib情况
      // 如果有有同名去掉Controller的xib, 会自动找到该xib的view做为控制器的view
      NJViewController *vc = [[NJViewController alloc] init];

     
6.重写控制器的loadveiw方法
     // 如果重写了控制器的loadview方法, 就不会去加载创建同名去掉controller的xib和同名的xib, 而是创建一个空白的veiw做为控制器的veiw
     NJViewController *vc = [[NJViewController alloc] init];
     
     // 设置控制器为window的根控制器
     self.window.rootViewController = vc;
     // 显示window
     [self.window makeKeyAndVisible];
     
     return YES;
 }
```
##创建控制器view的优先级

![](https://dn-zhunjiee.qbox.me/Snip20150808_1.png)
##控制器View的延迟加载

- 说明：
	- 控制器的view是延迟加载的：用到时再加载
	- 可以用isViewLoaded方法判断一个UIViewController的view是否已经被加载
	- 控制器的view加载完毕就会调用viewDidLoad方法

```objc
1 - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
 2 {
 3     // 1.创建UIWindow
 4     self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
 5     self.window.backgroundColor = [UIColor whiteColor];
 6     
 7     
 8     // 2.创建控制器
 9      NJViewController *vc = [[NJViewController alloc] init];
10     
11     // 其实是两步操作, 首先调用loadview方法, 创建控制器的veiw,然后再设置控制器的view的颜色为紫色, 也就是说后一次的颜色覆盖掉了前一次的颜色
12     vc.view.backgroundColor = [UIColor purpleColor];
13     
14     // 3.设置控制器为window的根控制器
15     self.window.rootViewController = vc;
16     
17     // 4.显示window(在这一行才用到了控制器的veiw)
18     [self.window makeKeyAndVisible];
19     
20     return YES;
21 }
```
- 主控制器文件中：

```objc
1 #import "NJViewController.h"
 2 
 3 @interface NJViewController ()
 4 
 5 @end
 6 
 7 @implementation NJViewController
 8 
 9 // 当控制器需要显示控制器的view的时候就会调用loadView
10 // 可以在loadView方法中创建view给控制器
11 // 该方法一般用于自定义控制器的view
12 - (void)loadView
13 {
14     // 什么时候调用loadveiw就代表什么时候加载控制器的veiw
15     NSLog(@"loadView");
16     
17     self.view = [[UIView alloc] init];
18     self.view.backgroundColor = [UIColor greenColor];
19 }
20 
21 - (void)viewDidLoad
22 {
23     [super viewDidLoad];
24     NSLog(@"viewDidLoad");
25 }
26 @end
```

## 如何创建一个控制器


###`一. 手动直接创建窗口`

- 什么时候创建?
    + 在加载info.plist文件之后,程序启动才完成,启动完成之后,就要显示窗口,因此在程序启动完成的时候创建窗口.

```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // 窗口显示的注意点:
    // 1.一定要强引用
    // 2.控件要想显示出来,必须要有尺寸

    // 1.创建窗口
    self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];

    // 2.创建根控制器,在设置窗口的根控制器
    UIViewController *vc = [[UIViewController alloc] init];
    vc.view.backgroundColor = [UIColor redColor];

    // 设置窗口的根控制器,底层会自动把根控制器的view添加到窗口上,并且让控制器的view有旋转功能
    self.window.rootViewController = vc;

    // 3.显示窗口
    // makeKeyAndVisible:让窗口成为应用程序的主窗口,并且显示窗口
    [self.window makeKeyAndVisible];

    return YES;
}

```


###`二. main.storyboard步骤`

- 什么时候创建
    - 加载info.plist,判断有没有指定main.storyboard,指定了main.storyboard,就会去加载main.storyboard,执行main.storyboard的时候创建.

```objc
2.1创建窗口
self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];

2.2加载控制器
//先加载storyboard文件（Test是storyboard的文件名）
UIStoryboard *storyboard = [UIStoryboard storyboardWithName:@"Test" bundle:nil];

//接着初始化storyboard中的控制器
//初始化“初始控制器”（箭头所指的控制器）
UIViewController *vc = [storyboard instantiateInitialViewController];

2.3设置窗口的根控制器,显示窗口
self.window.rootViewController = vc;

// 显示窗口
[self.window makeKeyAndVisible];
```

###`三. 指定xib方式创建`

```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    // 1.创建窗口
    self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];

    // 2.创建根控制器,在设置窗口的根控制器(Red为xib名字)
    ViewController *vc = [[ViewController alloc] initWithNibName:@"Red" bundle:nil];

    // 设置窗口的根控制器,底层会自动把根控制器的view添加到窗口上,并且让控制器的view有旋转功能
    self.window.rootViewController = vc;

    // 3.显示窗口
    // makeKeyAndVisible:让窗口成为应用程序的主窗口,并且显示窗口
    [self.window makeKeyAndVisible];

    return YES;
}
```



#控制器生命周期
- ARC:
    + viewDidLoad ---> viewWillAppear ---> viewWillLayoutSubviews ---> viewDidLayoutSubviews --->  viewDidAppear ---> viewWillDisappear ---> viewDidDisappear

- 非ARC:
    - viewWillUnload ---> viewDidUnload

---

###`窗口补充`

1.应用程序中那些控件属于窗口,1.状态栏 2.键盘

2.窗口层级关系
`UIWindowLevelAlert > UIWindowLevelStatusBar > UIWindowLevelNormal`

`设置窗口的层级,层级谁大就显示在最外面`

3.UITextField显示键盘

注意点:`如果一个键盘想要弹出来,必须把textField添加到一个控件上.`
