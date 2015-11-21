---
layout: post
title: "UITabBarController"
date: 2014-04-11
comments: false
categories: UI
---


- UITabBarController(49)的使用步骤
    - 初始化UITabBarController
    - 设置UIWindow的rootViewController为UITabBarController
    - 根据具体情况，通过addChildViewController方法添加对应个数的子控制器

```objc
// 启动完成的时候调用
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    // 1.创建窗口
    self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];

    // 2.设置窗口的根控制器
    // 2.1创建UITabBarController的根控制器
    UITabBarController *tab = [[UITabBarController alloc] init];
    // 2.2.设置根控制器
    self.window.rootViewController = tab;

    // 3.创建子控制器
    UIViewController *vc = [[UIViewController alloc] init];

    vc.view.backgroundColor = [UIColor redColor];
    vc.tabBarItem.title = @"消息";
    vc.tabBarItem.image = [UIImage imageNamed:@"tab_recent_nor"];

    // 3.1添加子控制器
    [tab addChildViewController:vc];



    // 4.显示窗口
    [self.window makeKeyAndVisible];


    return YES;
}
```

###UITabBarController的子控制器
UITabBarController添加控制器的方式有2种
- 添加单个子控制器

```objc
- (void)addChildViewController:(UIViewController *)childController;
// UITabBarController默认显示第0个控制器的view
```
- 设置子控制器数组

```objc
@property(nonatomic,copy) NSArray *viewControllers;
```

###UITabBarButton
- UITabBarButton里面显示什么内容，由对应子控制器的tabBarItem属性决定

- UITabBarItem有以下属性影响着UITabBarButton的内容

```objc
- 标题文字
@property(nonatomic,copy) NSString *title;

- 图标
@property(nonatomic,retain) UIImage *image;

- 选中时的图标
@property(nonatomic,retain) UIImage *selectedImage;

- 提醒数字
@property(nonatomic,copy) NSString *badgeValue;

```


#主流框架搭建
- 先搭建UITabBarController,再添加导航控制器为UITabBarController的子控制器
- Hide Bottom Bar On Push : 跳转时隐藏底部控制器,且只能隐藏系统的TabBar