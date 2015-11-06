---
layout: post
title: "UIScrollView,代理"
date: 2014-03-20
comments: false
categories: UI
---


#UIScrollView

##UIScrollView的使用步骤
- 1.创建UIScrollView

```objc
UIScrollView *scrollView = [UIScrollView alloc] initWithFrame:CGRectMake(0, 0, 200, 200);
```
- 2.给UIScrollView添加子控件

```objc
UIImageView *imgView = [UIImageView alloc] initWithImage:[UIImage imageNamed:@"1.jpg"];
[scrollView addSubview:imgView];
```
- 3.设置UIScrollview的contentSize

```objc
scrollView.contentSize = imgView.image.size;
```

- **UIScrollView不能滚动的几种情况**
    + 1.没有设置contentSize
    + 2.scrollEnabled = NO
    + 3.userInteractionEnabled = NO
    + 注意点:userInteractionEnabled不是Disabled, 他们两有本质的区别
        * Disabled代表控件不能使用, 状态是Disabled状态
        * userInteractionEnabled代表控件是否可以交互
    + 注意点:在企业开发中, 千万不要使用subviews来获取UIScrollview中的子控件
        * 因为subviews中包含了UIScrollview中的滚动条, 而且滚动条的顺序是不确定的

---
##UIScrollview常见属性
- scrollView.**bounces** = YES;
    + 用于设置UIScrollview是否有回弹效果

- scrollView.**scrollEnabled** = YES;
    + 设置是否可以滚动
- scrollView.**userInteractionEnabled** = YES;
   + 设置是否可以与用户进行交互,此项设置为否,即使上面设置为可以滚动的也不能滚动
- scrollView.**showsHorizontalScrollIndicator** = NO;
   + 隐藏水平滚动条
- scrollView.**showsVerticalScrollIndicator** = YES;
    + 显示垂直滚动条
- scrollView.**alwaysBounceHorizontal** = YES;
    + 水平回滚效果
- scrollView.**alwaysBounceVertical** = YES;
   + 垂直回滚效果 应用场景:下拉刷新
- scrollView.**indicatorStyle** = UIScrollViewIndicatorStyleWhite; // 设置滚动条的样式

---
- **contentSize**
    + 用于设置scrollView的滚动区域
- **contentOffset**
    + 作用: 用于设置内容的滚动偏移位
    + 计算公式: 移动的距离 =  "控件的左上角" - "内容的左上角"
        * 最好先理解iOS的坐标系, 然后再理解公式
- 可以利用`setContentOffset:offset animated:YES`方法设置偏移量, 达到在点击按钮切换视图的目的


- **contentInset**
    + 作用: 在contentSize周围添加额外的滚动区域
    + 应用场景: 避免UIScrollview中的内容被遮挡
```objc
self.tableView.contentInset = UIEdgeInsetsMake( 100, 0, 0, 0);
// UIEdgeInsetsMake(CGFloat top, CGFloat left, CGFloat bottom, CGFloat right):上左下右
```


- 注意查看PPT中的对比图理解: contentSize/contentOffset/contentInset/frame
![](https://dn-zhunjiee.qbox.me/Snip20150809_1.png)
---
##常用方法
- 设置偏移时带有动画效果

```objc
- (void)setContentOffset:(CGPoint)contentOffset animated:(BOOL)animated;  // animate at constant velocity to new offset
```
---

- 如何监听一个控件
    + 首先需要查看该控件的头文件, 看它继承于谁
        * 如果继承于UIControl, 那么就可以通过addTarget来监听
        * 如果不是继承于UIControl, 那么就必须通过代理来监听

---
#代理
####使用代理的步骤:
+ 1.遵守代理协议
+ 2.实现代理方法
+ 3.将遵守了协议的对象设置为代理

####代理的规律:
+ 代理名称的规律:
    * 协议名称以类名开头, 后面跟上Delegate
+ 代理方法名称的规律
    * 方法名称以类名去掉前缀开头,并且谁触发这个方法就将谁传递出去
+ 代理属性一般是id
+ 代理属性一般是weak, 主要是为了避免循环引用
    * 因为一般情况下, 控件的代理都是控制器,而控件又是添加到控制器的view中的

##代理设计模式练习
- 学生通过中介找房子的过程,学生不知道怎么找所以让代理帮忙找

###student.h
```objc
#import <Foundation/Foundation.h>
@class Student;

//谁的代理就写在谁的头部
@protocol StudentDelegate <NSObject>

- (void)helpToFindRoom:(Student *)student;

@end


@interface Student : NSObject

@property (nonatomic, copy) NSString *name;
@property (nonatomic, weak) id<StudentDelegate> delegate;

- (void)goToGuangZhou;

@end

```
###sudent.m

```objc
#import "Student.h"

@implementation Student

- (void)goToGuangZhou
{
    if ([_delegate respondsToSelector:@selector(helpToFindRoom:)]) {
        [_delegate helpToFindRoom:self];
    }
}
@end
```
###teacher.h
```objc
#import <Foundation/Foundation.h>
#import "Student.h"

@interface Teacher : NSObject <StudentDelegate>

@end
```
###teacher.m
```objc
#import "Teacher.h"

@implementation Teacher

- (void)helpToFindRoom:(Student *)student
{
    NSLog(@"帮%@同学找房子", student.name);
}
@end
```
###main.m

```objc
#import <Foundation/Foundation.h>
#import "Student.h"
#import "Teacher.h"

/*
 学生来广州找代理帮他找房子
 */
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Student *stu = [[Student alloc] init];
        Teacher *tea = [[Teacher alloc] init];
        stu.name = @"zhangsan";
        stu.delegate = tea;
        [stu goToGuangZhou];
    }
    return 0;
}

```

####代理的应用场景
+ 1.当A对象想监听B对象的变化时, 就可以使用代理, 让A成为B的代理
+ 2.当B对象想通知A对象的时候, 就可以使用代理, 让A成为B的代理

---
##如何监听UIScrollView停止滚动

- 如何监听UIScrollview停止滚动
    + scrollViewDidEndDragging
        * 只要用户松手就会调用
        * 停止拖拽并不代表停止滚动, 也就是说UIScrollView滚动是有惯性的
    + scrollViewDidEndDecelerating
        * 只要UIScrollview有惯性就会调用, 如果没有惯性就不会调用
    + 想要监听UIScrollview停止滚动必须同时实现这两个方法

```objc
- (void)scrollViewDidEndDragging:(UIScrollView *)scrollView willDecelerate:(BOOL)decelerate
{
    NSLog(@"%s = %d", __func__, decelerate);

    // 1.判断是否有惯性, 如果没有惯性手动调用scrollViewDidEndDecelerating告知已经完全停止滚动
    if (decelerate == NO) {
        [self scrollViewDidEndDecelerating:scrollView];
    }
}
- (void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView
{
    NSLog(@"%s", __func__);
    NSLog(@"停止滚动");
    self.iv.alpha = 1.0;
}

```
