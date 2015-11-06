---
layout: post
title: "initWithFrame/initWithCoder/awakeFromNib的区别,自定义控件,instancetype/id的区别"
date: 2014-03-16
comments: false
categories: UI
---
###initWithFrame
- 通过代码创建视图内容时,需要用initWithFrame初始化


###initWithCoder 
- 用于视图加载nib文件,从nib中加载对象实例时
- 当有连线时,initWithCoder会在连线之前加载完毕,所以无法获取连线的属性

```objc
- (instancetype)initWithCoder:(NSCoder *)coder{
	if(self = [superinitWithCoder:coder]){
		// 初始化代码
	}
	return self;
}
```


###awakeFromNib
- 通过xib或storyboard初始化时调用,只调用一次
- 可以获取连线的属性


###layoutSubviews:布局子控件
+ 1.只要frame有值就会调用
+ 2.但是可能第一次进来frame是没有值的,那么下次改变self.frame的时候还是调用

###layoutIfNeeded
- 只要有需要就布局，通常用于自动重新布局子控件

```objc
[self.view layoutIfNeeded];
```
---
####注：通过xib自定义的控件，通常情况我们会在initWithCoder中初始化控件，在awakeFromNib中设置属性，在layoutSubviews中设置尺寸

---

###类前缀
- 防止项目中的类名冲突
    + NSString NS  UILabel UI
    + ShopModel XJShopModel  LDHShopModel



## 通过纯代码自定义控件
- 继承自系统自带的控件，写一个属于自己的控件
- 目的：封装控件内部的细节，不让外界关心
- 步骤
    - 新建一个继承`UIView`的类
    - 在`initWithFrame:`方法中添加子控件
    - 在`layoutSubviews`方法中设置子控件的frame
        - 一定要调用`[super layoutSubviews]`;
    - 提供一个模型属性，重写模型属性的set方法
        - 在set方法中取出模型属性，给对应的子控件赋值

---

## 通过xib自定义控件


- 新建一个xib文件（xib的文件名最好跟控件类名一样）
    - 添加子控件、设置子控件属性
    - 修改最外面那个控件的class为控件类名
    - 将子控件进行连线


- 新建一个继承`UIView`的类
- XXShopView.h

```objc
#import <UIKit/UIKit.h>
@class XXShop;

@interface XXShopView : UIView
@property (strong, nonatomic) XXShop *shop; // 商品模型
+ (instancetype)shopViewWithShop:(XXShop *)shop;

// 下边的方法更简洁
+ (instancetype)shopView;
@end
```

- 提供模型属性，重写模型的set方法(模型参见上面模型章节)
    - 在set方法中给子控件设置数据
- XXShopView.m

```objc
#import "XXShopView.h"
#import "XXShop.h"

@implementation XXShopView

+ (instancetype)shopViewWithShop:(XXShop *)shop
{
    XXShopView *shopView = [[NSBundle mainBundle] loadNibaNamed:NSStringFromClass(self) owner:nil options:nil] lastObject];
    shopView.shop = shop;
    return shopView;
}


// 下边的方法更简洁
+ (instancetype)shopView{
    return [[[NSBundle mainBundle] loadNibaNamed:NSStringFromClass(self) owner:nil options:nil] lastObject];
}


// 重写模型的set方法
- (void)setShop:(XMGShop *)shop
{
    _shop = shop;

    UIImageView *iconView = (UIImageView *)[self viewWithTag:1];
    iconView.image = [UIImage imageNamed:shop.icon];

    UILabel *nameLabel = (UILabel *)[self viewWithTag:2];
    nameLabel.text = shop.name;
}

```

##instancetype / id 的区别
- instancetype 返回的是一个真实类型,是哪个类调用的就直接返回那个类
- id 返回的一个任意类型
- id 可以使用在参数的位置 instancetype 只能使用在方法的返回类型

