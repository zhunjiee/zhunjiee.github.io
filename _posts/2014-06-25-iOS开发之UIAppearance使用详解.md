---
layout: post
title: "iOS开发之UIAppearance使用详解"
date: 2014-06-25
comments: false
categories: 知识囊
---

苹果从iOS5开始提供了一个比较强大的工具**UIAppearance**，我们可以通过UIAppearance设置一些UI的全局效果，这样就可以很方便的实现UI的自定义效果又能最简单的实现统一界面风格。
###方法介绍
1. `+ (id)appearance`

- 这个方法是统一全部改，比如你设置UINavBar的tintColor，你可以这样写：

```
[[UINavigationBar appearance] setTintColor:myColor];
```
<br>
<br>

2. `+ (id)appearanceWhenContainedIn:(Class <>)ContainerClass,...`

- 这个方法可设置某个类的改变：例如：设置UIBarButtonItem 在UINavigationBar、UIPopoverController、UITabbar中的效果。就可以这样写

```
[[UIBarButtonItem appearanceWhenContainedIn:[UINavigationBar class], [UIPopoverController class],[UITabbar class] nil] setTintColor:myPopoverNavBarColor];
```
###使用注意
- iOS8以后又添加了一些方法,这里就不做介绍了,我们开发中最长用到的还是第1种方法

- 我们在查看方法或者属性的时候,若是看到方法或属性的后面是UI_APPEARANCE_SELECTOR的,我们就可以通过appearance统一设置相关的属性

- 注意 : 使用appearance设置UI效果最好采用全局的设置，在所有界面初始化前开始设置，否则可能失效。


###代码示例

统一设置TabBarItem的文字属性

```
/**
 *  统一设置TabBar按钮的文字属性
 */
- (void)setUpTabBarItemAttrs{
    UITabBarItem *item = [UITabBarItem appearance];
    
    NSMutableDictionary *normalAttrs = [NSMutableDictionary dictionary];
    normalAttrs[NSFontAttributeName] = [UIFont systemFontOfSize:15];
    normalAttrs[NSForegroundColorAttributeName] = [UIColor grayColor];
    [item setTitleTextAttributes:normalAttrs forState:UIControlStateNormal];
    
    NSMutableDictionary *selectedAttrs = [NSMutableDictionary dictionary];
    selectedAttrs[NSForegroundColorAttributeName] = [UIColor blackColor];
    [item setTitleTextAttributes:selectedAttrs forState:UIControlStateSelected];
}
```