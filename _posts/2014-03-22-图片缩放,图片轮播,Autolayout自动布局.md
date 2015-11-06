---
layout: post
title: "图片缩放,图片轮播,Autolayout自动布局"
date: 2014-03-22
comments: false
categories: UI
---


#UIScrollView实现图片缩放

- **步骤**:
    + 1.成为UIScrollview的代理, 通过代理方法告诉UIScrollview要缩放哪一个子控件
        * < UIScrollViewDelegate >
    + 2.设置子控件最大和最小的缩放比例
        * maximumZoomScale
        * mininumZoomScale
    + 3.viewForZoomingInScrollView :返回需要缩放的子控件

```objc
@interface ViewController () <UIScrollViewDelegate>

@property (weak, nonatomic) UIScrollView *scrollView;
@property (weak, nonatomic) UIImageView *imgView;

@end

- (void)viewDidLoad {
    [super viewDidLoad];

    UIScrollView *scrollView = [[UIScrollView alloc] initWithFrame:CGRectMake(0, 0, 100, 100)];
    [self.view addSubview:scrollView];
    self.scrollView = scrollView;

    UIImageView *imgView = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"27003623"]];
    [self.scrollView addSubview:imgView];
    self.imgView = imgView;
    // 设置滚动范围
    self.scrollView.contentSize = imgView.image.size;
    // 设置代理
    self.scrollView.delegate = self;
    // 设置缩放比例
    self.scrollView.maximumZoomScale = 2.0;
    self.scrollView.minimumZoomScale = 0.1;
}

// 返回需要缩放的子控件
- (UIView *)viewForZoomingInScrollView:(UIScrollView *)scrollView{
    return self.imgView;
}
```
- **常用方法**
    + viewForZoomingInScrollView : 返回需要缩放的子控件
    + scrollViewDidZoom: 只要子控件被缩放了就会调用(调用频率非常高)
    + scrollViewDidEndZooming: 缩放完毕时掉用

- **如何保证被缩放的子控件永远居中**
    + 只要子控件的contentSize小于UIScrollview的宽高时才需要缩放

- 第一种

```objc
- (void)scrollViewDidZoom:(UIScrollView *)scrollView{
 // 1.判断contentSize的宽高, 是否小于UIScrollView的frame的size
    if ((scrollView.contentSize.width < scrollView.bounds.size.width) || (scrollView.contentSize.height < scrollView.bounds.size.height)){

        self.imgView.center = CGPointMake(scrollView.bounds.size.width * 0.5, scrollView.bounds.size.height * 0.5);

    }

}
```

- 第二种

```objc
- (void)scrollViewDidZoom:(UIScrollView *)scrollView{
 // 1.判断contentSize的宽高, 是否小于UIScrollView的frame的size
    if (scrollView.contentSize.width < scrollView.bounds.size.width) {

        // 计算X的值
        CGFloat x =  (scrollView.contentSize.width * 0.5) + ((scrollView.bounds.size.width - scrollView.contentSize.width) * 0.5);

        // 重新设置center
        self.iv1.center = CGPointMake(x, self.iv1.center.y);
    }

    if (scrollView.contentSize.height < scrollView.bounds.size.height) {

        // 计算Y的值
        CGFloat y =  (scrollView.contentSize.height * 0.5) + ((scrollView.bounds.size.height - scrollView.contentSize.height) * 0.5);

        // 重新设置center
        self.iv1.center = CGPointMake(self.iv1.center.x, y);
    }
}
```

- 第三种

```objc
- (void)scrollViewDidZoom:(UIScrollView *)scrollView{
 CGFloat x = scrollView.contentSize.width < scrollView.bounds.size.width ? (scrollView.bounds.size.width - scrollView.contentSize.width) * 0.5 : 0.0;
    CGFloat y = scrollView.contentSize.height < scrollView.bounds.size.height ? (scrollView.bounds.size.height - scrollView.contentSize.height) * 0.5 : 0.0;
    self.iv1.center = CGPointMake(scrollView.contentSize.width * 0.5 + x, scrollView.contentSize.height * 0.5 + y);
}
```

---
# UIPageControl

---
## 图片轮播器
- 如何分页 : **pagingEnabled = YES**
- 分页的原理: 是根据UIScrollview的宽度或者高度来分页

##UIPageControl
- numberOfPages : 设置总页码
- currentPage: 设置当前页码
- pageIndicatorTintColor: 设置其它页码的颜色
- currentPageIndicatorTintColor : 设置当前页码的颜色
- hidesForSinglePage : 如果只有一页,隐藏页码

- pageControl.enabled = NO;  --- 设置pagecontrol为不可点击
- pageControl.userInteractionEnabled = NO; --- 设置pageControl与用户不可交互

---
- **自定义页码**
    + 利用KVC给UIPageControl设置页码图片

```objc
[pageControl setValue:[UIImage imageNamed:@"current"] forKeyPath:@"_currentPageImage"];
[pageControl setValue:[UIImage imageNamed:@"other"] forKeyPath:@"_pageImage"];
```

- 监听UIPageControl的点击
    + 由于UIPageControl继承于UIControl, 所以通过addTargt来监听

```objc
[pageControl addTarget:self action:@selector(nextPage) forControlEvents:UIControlEventValueChanged];
```

- 切换页码
    + 滚动完毕之后再切换

```objc
- (void)scrollViewDidEndDragging:(UIScrollView *)scrollView willDecelerate:(BOOL)decelerate
{
    // 判断是否有惯性, 如果没有就手动调用scrollViewDidEndDecelerating
    if (NO == decelerate) {
        [self scrollViewDidEndDecelerating:scrollView];
    }
}

// 只有有惯性才会调用
- (void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView
{
    // 1.动态计算当前的页码
    // 页码 = UIScrollView的偏移位 / UIScrollView的宽度
    int page = scrollView.contentOffset.x / scrollView.bounds.size.width;
    NSLog(@"page = %d", page);
    // 2.设置当前的页码
    self.pageControl.currentPage = page;

}
```
    + 实时切换

```objc
// 只要滚动就会调用
- (void)scrollViewDidScroll:(UIScrollView *)scrollView
{
//    NSLog(@"%s", __func__);
    // 1.动态计算当前的页码
    // 页码 = UIScrollView的偏移位 / UIScrollView的宽度
    CGFloat page = scrollView.contentOffset.x / scrollView.bounds.size.width;

    NSLog(@"page = %f", page + 0.5);
    // 2.设置当前的页码
    self.pageControl.currentPage = (int)(page + 0.5);
}

```

---

#NSTimer


- 作用: 可以让系统每隔一段时间执行指定对象的指定方法
- 注意:
    + 间隔时间是不准确的
    + 只要通过scheduledTimerWithTimeInterval创建出来的Timer,就会被RUNLOOP强引用, 所以如果通过属性保存使用weak
    + 只要调用了NSTimer的invalidate方法, 那么定时器就不能使用了, 想要使用必须重新创建
    + 如何主线程正在处理其它操作, 那么NSTimer不会执行
        * 默认NSTimer是NSDefaultRunLoopMode模式
        * 要想在主线程处理其它操作的时候也处理NSTimer, 那么必须把NSTiemr在RunLoop中的模式改为NSRunLoopCommonModes

```objc

// 注意: 只要通过scheduledTimerWithTimeInterval方法, 创建一个NSTimer, 那么系统就会自动将NSTimer添加到RunLoop中, RunLoop会对NSTimer进行强引用, 所以NSTimer不会被释放
// 面试题:通过属性保存NSTimer, 用Strong还是weak. 用weak
#warning 注意: NSTimer定时器是不准确的
// NSTimer是在主线程中执行的, 而主线程同一时刻只能执行一个操作
// 要想解决这个问题, 就必须告诉线程再忙也要分一点时间来照顾我
    // TimeInterval : 间隔几秒
    // target : 将来调用谁的方法
    // selector: 调用什么方法
    // userInfo: 调用方法时需要传递的参数, 可以为nil
    // repeats: 是否重复
    self.timer = [NSTimer scheduledTimerWithTimeInterval:2.0 target:self selector:@selector(nextPage) userInfo:nil repeats:YES];

    [[NSRunLoop mainRunLoop] addTimer:self.timer forMode:NSRunLoopCommonModes];
```


- **如何暂停和启动定时器**

```objc
    // 暂停定时器
    // 设置timer的开始时间为 遥远的未来 4001
//    NSLog(@"%@", [NSDate distantFuture]);
    [self.timer setFireDate:[NSDate distantFuture]];
```

```objc
    // 开启定时器
    // 设置定时器的开始时间为过去的某一个时间, 也就意味着立刻开始
//    [self.timer setFireDate:[NSDate distantPast]];
// 设置定时器从当前时间之后两秒才开始执行
    [self.timer setFireDate:[NSDate dateWithTimeIntervalSinceNow:2.0]];
```

---
- 封装
    + 只要发现控制器知道得太多,就要考虑重构代码
    + 只要发现一个效果很多地方都需要使用, 就要考虑封装

---

#Autolayout


###Autoresizing

imgView.**autoresizingmask** =

- UIViewAutoresizingFlexible`Left`Margin   = 1 << 0,
    - 距离父控件`左边`的间距是伸缩的
- UIViewAutoresizingFlexible`Right`Margin  = 1 << 2,
    - 距离父控件`右边`的间距是伸缩的
- UIViewAutoresizingFlexible`Top`Margin    = 1 << 3,
    - 距离父控件`上边`的间距是伸缩的
- UIViewAutoresizingFlexible`Bottom`Margin = 1 << 5
    - 距离父控件`下边`的间距是伸缩的
- UIViewAutoresizingFlexible`Width`        = 1 << 1,
    - `宽度`跟随父控件`宽度`进行伸缩
- UIViewAutoresizingFlexible`Height`       = 1 << 4,
    - `高度`跟随父控件`高度`进行伸缩

---
### 2个核心概念
- 约束
    - 尺寸约束
        - width约束
        - height约束
    - 位置约束
        - 间距约束（上下左右间距）

- 参照
    - 所添加的约束跟哪个控件有关（相对于哪个控件来说）

### 常见单词
- Leading -> Left -> 左边
- Trailing -> Right -> 右边

### UILabel实现包裹内容
- 设置宽度约束为 <= 固定值
- 设置位置约束
- 不用去设置高度约束

##代码实现Autolayout的步骤
- 利用NSLayoutConstraint类创建具体的约束对象
- 添加约束对象到相应的view上
```objc
\- (void)addConstraint:(NSLayoutConstraint *)constraint;
\- (void)addConstraints:(NSArray *)constraints;
```
- 代码实现Autolayout的注意点
    + 要先禁止autoresizing功能，设置view的下面属性为NO
    + view.translatesAutoresizingMaskIntoConstraints = NO;
    + 添加约束之前，一定要保证相关控件都已经在各自的父控件上
    + 不用再给view设置frame

## NSLayoutConstraint
- 一个NSLayoutConstraint对象就代表一个约束

创建约束对象的常用方法
```objc
+(id)constraintWithItem:(id)view1 attribute:(NSLayoutAttribute)attr1 relatedBy:(NSLayoutRelation)relation toItem:(id)view2 attribute:(NSLayoutAttribute)attr2 multiplier:(CGFloat)multiplier constant:(CGFloat)c;
```
- view1 ：要约束的控件
- attr1 ：约束的类型（做怎样的约束）
- relation ：与参照控件之间的关系
- view2 ：参照的控件
- attr2 ：约束的类型（做怎样的约束）
- multiplier ：乘数
- c ：常量

##自动布局的核心计算公式
`obj1.property1 =（obj2.property2 * multiplier）+ constant value`

---
##VFL示例
- H:[cancelButton(72)]-12-[acceptButton(50)]
    + canelButton宽72，acceptButton宽50，它们之间间距12

- H:[wideView(>=60@700)]
    + wideView宽度大于等于60point，该约束条件优先级为700（优先级最大值为1000，优先级越高的约束越先被满足）

- V:[redBox][yellowBox(==redBox)]
    + 竖直方向上，先有一个redBox，其下方紧接一个高度等于redBox高度的yellowBox

- H:|-10-[Find]-[FindNext]-[FindField(>=20)]-|
    + 水平方向上，Find距离父view左边缘默认间隔宽度，之后是FindNext距离Find间隔默认宽度；再之后是宽度不小于20的FindField，它和FindNext以及父view右边缘的间距都是默认宽度。（竖线“|” 表示superview的边缘）

##VFL的使用

使用VFL来创建约束数组
+ (NSArray *)constraintsWithVisualFormat:(NSString *)format options:(NSLayoutFormatOptions)opts metrics:(NSDictionary *)metrics views:(NSDictionary *)views;
format ：VFL语句
opts ：约束类型
metrics ：VFL语句中用到的具体数值
views ：VFL语句中用到的控件

创建一个字典（内部包含VFL语句中用到的控件）的快捷宏定义
NSDictionaryOfVariableBindings(...)