---
layout: post
title: "UIScrollView-滚动视图使用简介"
date: 2014-03-20
comments: false
categories: UI
---

UIScrollView通常称为滚动视图,我们可以用它来实现很多的效果,如图片浏览,图片缩放,添加定时器后可以制作成图片轮播器等等,下面我们就一起看看UIScrollView的使用步骤吧.

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

