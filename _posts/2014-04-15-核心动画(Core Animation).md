---
layout: post
title: "核心动画(Core Animation)"
date: 2014-04-15
comments: false
categories: UI
---


#Core Animation(核心动画)
开发步骤:

1. 首先得有CALayer
2. 初始化一个CAAnimation对象，并设置一些动画相关属性
3. 通过调用CALayer的addAnimation:forKey:方法，增加CAAnimation对象到CALayer中，这样就能开始执行动画了
4. 通过调用CALayer的removeAnimationForKey:方法可以停止CALayer中的动画

##CAAnimation——简介
- 是所有动画对象的父类，负责控制动画的持续时间和速度，是个抽象类，不能直接使用，应该使用它具体的子类

- 属性说明：(红色代表来自CAMediaTiming协议的属性)

	- `duration`：动画的持续时间

	- `repeatCount`：重复次数，无限循环可以设置HUGE_VALF或者MAXFLOAT

	- `repeatDuration`：重复时间

	- `removedOnCompletion`：默认为YES，代表动画执行完毕后就从图层上移除，图形会恢复到动画执行前的状态。如果想让图层保持显示动画执行后的状态，那就设置为NO，不过还要设置fillModekCAFillModeForwards

	- `fillMode`：决定当前对象在非active时间段的行为。比如动画开始之前或者动画结束之后

	- `beginTime`：可以用来设置动画延迟执行时间，若想延迟2s，就设置为CACurrentMediaTime()+2，CACurrentMediaTime()为图层的当前时间

	- `timingFunction`：速度控制函数，控制动画运行的节奏

	- `delegate`：动画代理
	- `autoreverses`: 自动恢复到原来的状态,动画怎么过来的就怎么回去

##CAAnimation——动画填充模式

**fillMode属性值**（要想fillMode有效，最好设置removedOnCompletion = NO）

`kCAFillModeRemoved` 这个是默认值，也就是说当动画开始前和动画结束后，动画对layer都没有影响，动画结束后，layer会恢复到之前的状态

`kCAFillModeForwards` 当动画结束后，layer会一直保持着动画最后的状态
 
`kCAFillModeBackwards` 在动画开始前，只需要将动画加入了一个layer，layer便立即进入动画的初始状态并等待动画开始。

`kCAFillModeBoth` 这个其实就是上面两个的合成.动画加入后开始之前，layer便处于动画初始状态，动画结束后layer保持动画最后的状态



##CAAnimation——速度控制函数
- 速度控制函数(CAMediaTimingFunction) 

`kCAMediaTimingFunctionLinear（线性）`：匀速，给你一个相对静态的感觉

`kCAMediaTimingFunctionEaseIn（渐进）`：动画缓慢进入，然后加速离开

`kCAMediaTimingFunctionEaseOut（渐出）`：动画全速进入，然后减速的到达目的地

`kCAMediaTimingFunctionEaseInEaseOut（渐进渐出）`：动画缓慢的进入，中间加速，然后减速的到达目的地。这个是默认的动画行为。

##CAAnimation——动画代理方法
CAAnimation在分类中定义了代理方法


```objc
@interface NSObject (CAAnimationDelegate)

// 动画开始时调用
- (void)animationDidStart:(CAAnimation *)anim;

// 动画结束时调用
- (void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag;

@end
```
##CALayer上动画的暂停和恢复
- 暂停CALayer的动画

```objc
#pragma mark 暂停CALayer的动画
-(void)pauseLayer:(CALayer*)layer
{
    CFTimeInterval pausedTime = [layer convertTime:CACurrentMediaTime() fromLayer:nil];

    // 让CALayer的时间停止走动
      layer.speed = 0.0;
    // 让CALayer的时间停留在pausedTime这个时刻
    layer.timeOffset = pausedTime;
}
```
- 恢复CALayer的动画

```objc
#pragma mark 恢复CALayer的动画
-(void)resumeLayer:(CALayer*)layer
{
    CFTimeInterval pausedTime = layer.timeOffset;
    // 1. 让CALayer的时间继续行走
      layer.speed = 1.0;
    // 2. 取消上次记录的停留时刻
      layer.timeOffset = 0.0;
    // 3. 取消上次设置的时间
      layer.beginTime = 0.0;    
    // 4. 计算暂停的时间(这里也可以用CACurrentMediaTime()-pausedTime)
    CFTimeInterval timeSincePause = [layer convertTime:CACurrentMediaTime() fromLayer:nil] - pausedTime;
    // 5. 设置相对于父坐标系的开始时间(往后退timeSincePause)
      layer.beginTime = timeSincePause;
}
```
##CAPropertyAnimation
- 是CAAnimation的子类，也是个抽象类，要想创建动画对象，应该使用它的两个子类：
	- CABasicAnimation
	- CAKeyframeAnimation

- 属性说明：
	- `keyPath`：通过指定CALayer的一个属性名称为keyPath（NSString类型），并且对CALayer的这个属性的值进行修改，达到相应的动画效果。比如，指定@“position”为keyPath，就修改CALayer的position属性的值，以达到平移的动画效果

##CABasicAnimation——基本动画

- 基本动画，是CAPropertyAnimation的子类

- 属性说明:
	- `keyPath`：通过指定CALayer的一个属性名称为keyPath（NSString类型），并且对CALayer的这个属性的值进行修改，达到相应的动画效果。
	- `fromValue`：keyPath相应属性的初始值
	- `toValue`：keyPath相应属性的结束值

- 动画过程说明：
	- 随着动画的进行，在长度为duration的持续时间内，keyPath相应属性的值从fromValue渐渐地变为toValue
	- keyPath内容是CALayer的可动画Animatable属性
	- 如果fillMode=kCAFillModeForwards同时removedOnComletion=NO，那么在动画执行完毕后，图层会保持显示动画执行后的状态。但在实质上，图层的属性值还是动画执行前的初始值，并没有真正被改变。

```objc
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event{
    // 创建动画对象
    CABasicAnimation *anim = [CABasicAnimation animation];
    
    // 设置动画属性
    //anim.keyPath = @"transform.translate";
    anim.keyPath = @"transform.scale";
    
    // 设置属性的改变值
	//anim.toValue = [NSValue valueWithCGPoint:CGPointMake(.5, .5)];
    anim.toValue = @0.5;
    
    // 设置动画时长
    anim.duration = 0.7;
    
    // 动画执行完毕之后不要移除
    anim.removedOnCompletion = NO;
    
    // 保持最新位置
    anim.fillMode = kCAFillModeForwards;
    
    // 重复次数
    anim.repeatCount = MAXFLOAT;
    
    // 给图层添加动画
    [_layer addAnimation:anim forKey:nil];
    
}
```


##CAKeyframeAnimation——关键帧动画
- 关键帧动画，也是CAPropertyAnimation的子类，与CABasicAnimation的区别是：
	- **CABasicAnimation只能从一个数值（fromValue）变到另一个数值（toValue）**
    - **而CAKeyframeAnimation会使用一个NSArray保存这些数值**


- 属性说明：

	- `values`：上述的NSArray对象。里面的元素称为“关键帧”(keyframe)。动画对象会在指定的时间（duration）内，依次显示values数组中的每一个关键帧
	- `keyPath`：通过指定CALayer的一个属性名称为keyPath（NSString类型），并且对CALayer的这个属性的值进行修改，达到相应的动画效果。
	- `path`：可以设置一个CGPathRef、CGMutablePathRef，让图层按照路径轨迹移动。path只对CALayer的anchorPoint和position起作用。如果设置了path，那么values将被忽略
    - `keyTimes`：可以为对应的关键帧指定对应的时间点，其取值范围为0到1.0，keyTimes中的每一个时间值都对应values中的每一帧。如果没有设置keyTimes，各个关键帧的时间是平分的

- CABasicAnimation可看做是只有2个关键帧的CAKeyframeAnimation

###图标抖动

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    
    UILongPressGestureRecognizer *longPress = [[UILongPressGestureRecognizer alloc] initWithTarget:self action:@selector(longPress:)];
    
    [_imageView addGestureRecognizer:longPress];
}


- (void)longPress:(UILongPressGestureRecognizer *)longPress{
    if (longPress.state == UIGestureRecognizerStateBegan) {
        CAKeyframeAnimation *anim = [CAKeyframeAnimation animation];
        
        anim.keyPath = @"transform.rotation";
        
        anim.values = @[
                        @(angle2radian(-5)),
                        @(angle2radian(5)),
                        @(angle2radian(-5))];
        anim.repeatCount = MAXFLOAT;
        
        anim.duration = 0.35;
        
        [_imageView.layer addAnimation:anim forKey:nil];
    }
}
```

##CAAnimationGroup——动画组
- 动画组，是CAAnimation的子类，可以保存一组动画对象，将CAAnimationGroup对象加入层后，组中所有动画对象可以同时并发运行

- 属性说明：
	- animations：用来保存一组动画对象的NSArray
默认情况下，一组动画对象是同时运行的，也可以通过设置动画对象的beginTime属性来更改动画的开始时间

##转场动画——CATransition
- CATransition是CAAnimation的子类，用于做转场动画，能够为层提供移出屏幕和移入屏幕的动画效果。iOS比Mac OS X的转场动画效果少一点
- UINavigationController就是通过CATransition实现了将控制器的视图推入屏幕的动画效果

- 动画属性:
	- `type`：动画过渡类型
	- `subtype`：动画过渡方向
	- `startProgress`：动画起点(在整体动画的百分比)
	- `endProgress`：动画终点(在整体动画的百分比)

![转场动画过度效果](https://dn-zhunjiee.qbox.me/Snip20150816_2.png)
###核心动画都是假象,不能改变layer的真实属性的值
---
##使用UIView动画函数实现转场动画——单视图
```objc
+ (void)transitionWithView:(UIView *)view duration:(NSTimeInterval)duration options:(UIViewAnimationOptions)options animations:(void (^)(void))animations completion:(void (^)(BOOL finished))completion;
```
- 参数说明：
	- duration：动画的持续时间
	- view：需要进行转场动画的视图
	- options：转场动画的类型
	- animations：将改变视图属性的代码放在这个block中
	- completion：动画结束后，会自动调用这个block

##使用UIView动画函数实现转场动画——双视图
```objc
+ (void)transitionFromView:(UIView *)fromView toView:(UIView *)toView duration:(NSTimeInterval)duration options:(UIViewAnimationOptions)options completion:(void (^)(BOOL finished))completion;
```
- 参数说明：
	- duration：动画的持续时间
	- options：转场动画的类型
	- animations：将改变视图属性的代码放在这个block中
	- completion：动画结束后，会自动调用这个block

##CADisplayLink
- CADisplayLink是一种以屏幕刷新频率触发的时钟机制，每秒钟执行大约60次左右
- CADisplayLink是一个计时器，可以使绘图代码与视图的刷新频率保持同步，而NSTimer无法确保计时器实际被触发的准确时间

- 使用方法：
	- 定义CADisplayLink并制定触发调用方法
	- 将显示链接添加到主运行循环队列





