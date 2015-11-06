---
layout: post
title: "UINavgationController使用详解"
date: 2014-04-09
comments: false
categories: UI
---
UINavgationController---导航控制器，在我们iOS控制器里非常常见，可以说每个APP都会有导航栏，这也是主流控制器的重要组成部分，关于主流控制器的搭建我有一篇博客有专门讲[10分钟搭建主流APP框架](http://www.zhunjiee.com/ios/2014/04/19/10%E5%88%86%E9%92%9F%E6%90%AD%E5%BB%BAAPP%E4%B8%BB%E6%B5%81%E6%A1%86%E6%9E%B6.html)

##UINavigationController的使用步骤
- 初始化UINavigationController
- 设置UIWindow的rootViewController为UINavigationController
- 根据具体情况，通过push方法添加对应个数的子控制器

```objc
// 启动完成的时候调用
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    // 1.创建窗口
    self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];

    // 2.设置窗口的根控制器
    // 2.1创建导航控制器的根控制器
    OneViewController *vc = [[OneViewController alloc] init];

    // 3.创建导航控制器,需要根控制器
    UINavigationController *nav = [[UINavigationController alloc] initWithRootViewController:vc];
    // initWithRootViewContorller:底层调用了push方法,不再需要实现下面的方法了
    // 添加子控件
    //[nav pushViewController:vc animated:YES];

    // 4.设置根目录
    self.window.rootViewController = nav;

    // 5.显示窗口
    [self.window makeKeyAndVisible];


    return YES;
}
```
##UINavigationController的子控制器
- UINavigationController以栈的形式保存子控制器

```objc
@property(nonatomic,copy) NSArray *viewControllers;
@property(nonatomic,readonly) NSArray *childViewControllers;
```
- 使用push方法能将某个控制器压入栈,不显示的子控制器不会销毁

```objc
- (void)pushViewController:(UIViewController *)viewController animated:(BOOL)animated;
```
- 使用pop方法可以移除控制器,退回到前一个控制器
- 将栈顶的控制器移除,会销毁子控制器

```objc
- (UIViewController *)popViewControllerAnimated:(BOOL)animated;
```
- 回到指定的子控制器

```objc
- (NSArray *)popToViewController:(UIViewController *)viewController animated:(BOOL)animated;
```
- 回到根控制器（栈底控制器）

```objc
- (NSArray *)popToRootViewControllerAnimated:(BOOL)animated;
```

##如何修改导航栏的内容

- 导航栏的内容由栈顶控制器的navigationItem属性决定

```objc
// 栈顶导航控制器里设置
self.navigationItem.title = @"小码哥";
```
- 导航条上面的子控件的位置(x, y)由系统决定,我们只能控制控件的尺寸size(width, height)
    + 如果以后看到系统的类带有Item,就是苹果提供的模型

####`UINavigationBar`:整个导航条
####`UINavigationItem`:设置导航条内容 模型
####`UIBarButtonItem`:设置导航条上按钮的内容 模型
![](https://dn-zhunjiee.qbox.me/Snip20151104_2.png)

- `UIBarButtonItem *item2 = [[UIBarButtonItem alloc] initWithCustomView:btn];`

---
- UINavigationItem有以下属性影响着导航栏的内容

```objc
- 左上角的返回按钮
@property(nonatomic,retain) UIBarButtonItem *backBarButtonItem;

- 中间的标题视图
@property(nonatomic,retain) UIView          *titleView;

- 中间的标题文字
@property(nonatomic,copy)   NSString        *title;

- 左上角的视图
@property(nonatomic,retain) UIBarButtonItem *leftBarButtonItem;

- 右上角的视图
@property(nonatomic,retain) UIBarButtonItem *rightBarButtonItem;
```


- ios7以后,导航条上的按钮图片默认会被渲染成蓝色

```objc
    // 左边的内容
    UIBarButtonItem *item = [[UIBarButtonItem alloc] initWithTitle:@"返回" style:UIBarButtonItemStyleDone target:self action:@selector(back)];

    // 在iOS7之后,默认会导航条上按钮的图片渲染成蓝色
    // 获取图片
    UIImage *image = [UIImage imageNamed:@"navigationbar_friendsearch"];
    // 告诉这个图片不要渲染
    // 返回一个没有渲染的图片给你
    image = [image imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal];

    UIBarButtonItem *item1 = [[UIBarButtonItem alloc] initWithImage:image style:UIBarButtonItemStyleDone target:nil action:nil];


    // 显示多张图片,不同状态,用按钮
    UIButton *btn = [UIButton buttonWithType:UIButtonTypeCustom];
    [btn setImage:[UIImage imageNamed:@"navigationbar_friendsearch"] forState:UIControlStateNormal];

    [btn setImage:[UIImage imageNamed:@"navigationbar_friendsearch_highlighted"] forState:UIControlStateHighlighted];

    // 按钮自适应,根据图片计算尺寸
    [btn sizeToFit];


    UIBarButtonItem *item2 = [[UIBarButtonItem alloc] initWithCustomView:btn];

//    self.navigationItem.leftBarButtonItem = item;

    // 右边内容
    self.navigationItem.rightBarButtonItems = @[item,item1,item2];

```

**在iOS7之后,默认会给导航控制器里第一个ScrollView类型的view顶部添加额外的滚动区域64**

```objc
// 设置不需要添加额外滚动区域
self.automaticallyAdjustsScrollViewInsets = NO;
```
为啥会是64呢？

- 因为导航控制器高度 20 + 44，其中20为statusBar的高度，44为导航控制器的真实高度，所以就是避免导航控制器盖住了ScrollView的内容


##修改返回按钮
- 必须在父控件里修改子控件的返回按钮

```objc
// ">back" 改为 "返回"
// 必须在父控件里修改子控件的返回按钮
UIBarButtonItem *item = [[UIBarButtonItem alloc] initWithTitle:@"返回" style:UIBarButtonItemStyleDone target:nil action:nil];
self.navigationItem.backBarButtonItem = item;
```
##修改返回按钮颜色
```objc
// 设置返回按钮文字颜色
self.navigationController.navigationBar.tintColor = [UIColor blackColor];
// 设置整个导航栏颜色
self.navigationController.navigationBar.barTintColor = [UIColor orangeColor];


```
---

#Modal
除了push之外，还有另外一种控制器的切换方式，那就是Modal

- 任何控制器都能通过Modal的形式展示出来

- Modal的默认效果：新控制器从屏幕的最底部往上钻，直到盖住之前的控制器为止

- 以Modal的形式展示控制器

```objc
- (void)presentViewController:(UIViewController *)viewControllerToPresent animated: (BOOL)flag completion:(void (^)(void))completion
```
- 关闭当初Modal出来的控制器

```objc
- (void)dismissViewControllerAnimated: (BOOL)flag completion: (void (^)(void))completion;
```

- 如果一个控制器的view显示到屏幕上,这个控制器一定不能被销毁



---


