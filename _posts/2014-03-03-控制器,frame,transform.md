---
layout: post
title: "控制器,frame,transform"
date: 2014-03-03
comments: false
categories: UI
---


## 控制器

- 概念：凡是继承自UIViewController的对象，都叫做控制器
- 注意：每一个控制器都会专门管理一个软件界面
- 作用：负责处理软件界面的各种事件、负责软件界面的创建和销毁

## IBAction

- 只能修饰方法的返回值类型
- 被IBAction修饰的方法
    - 能拖线到storyboard中
    - 返回值类型实际是void
- 使用格式

```objc
- (IBAction)buttonClick
{

}
```

## IBOutlet
- 通过IBOutlet修饰的成员属性虽然外部是weak修饰的,但IBOutlet内部隐藏一个强指针
- 在对象被销毁时,并不会立马被清空,只有当发现没有别的指针只用该成员属性时IBoutlet内部隐藏的强指针才会释放,对象才会被销毁

- 只能修饰属性
- 被IBOutlet修饰的属性
    - 能拖线到storyboard中
- 使用格式

```objc
@property (nonatomic, weak) IBOutlet UILabel *label;
```

## 关于IBAction、IBOutlet前缀IB的解释
- 全称：Interface Builder
- 以前的UI界面开发模式：Xcode3 + Interface Builder
- 从Xcode4开始，Interface Builder已经整合到Xcode中了


##tag
1.使用tag根据父控件能取出当前内部子控件
2.使用tag能判断点击的哪个按钮

```objc
- (UIView *)viewWithTag:(NSInteger)tag;
```


#frame / bounds / center

- frame:
    + 1.能设置尺寸
    + 2.能设置位置
- bounds:
    + 只能设置尺寸不能设置位置
- (CGPoint)center:
    + 只能设置位置不能设置尺寸
- transform:
    + 能修改大小,位置,旋转角度

```objc
@property(nonatomic) CGRect            frame;
@property(nonatomic) CGRect            bounds;
@property(nonatomic) CGPoint           center;

struct CGRect {
  CGPoint origin;
  CGSize size;
};
typedef struct CGRect CGRect;


```
- **CGRectGetMaxX**(_mainView.frame) : 获取view的x的值+view的宽度的值

- **CGRectGetMinX**(_mainView.frame) : 获取view的x值



#transform
- `CGAffineTransformMakeTranslation`:每次从零开始形变,每次都是相对于最开始的位置形变
- `CGAffineTransformTranslate`:从当前位置开始形变,下一次继续

- `CGAffineTransformRotate`:旋转
- `CGAffineTransformScale`:缩放, 比例
- `CGAffineTransformIdentity`:复位

```objc
/***********<transform>实现移动效果***************/

- (IBAction)transRotate:(id)sender {
    [UIView animateWithDuration:2 animations:^{
        self.header.transform =  CGAffineTransformRotate(self.header.transform, M_PI_4); // 顺时针移动45°,前面加 "-" 代表逆时针旋转
    }];
}

- (IBAction)transMove:(id)sender {
    self.header.transform = CGAffineTransformTranslate(self.header.transform, 0, -50); // 向上移动
}

- (IBAction)transBig:(id)sender {
    // 缩放 比例
    self.header.transform = CGAffineTransformScale(self.header.transform, 1.5, 1.5);
}

- (IBAction)transSmall:(id)sender {
    self.header.transform = CGAffineTransformScale(self.header.transform, 0.5, 0.5);
}

- (IBAction)transform:(id)sender {
    [UIView animateWithDuration:3 animations:^{
        self.header.transform = CGAffineTransformTranslate(self.header.transform, 0, -50); // 向上移动
        self.header.transform =  CGAffineTransformRotate(self.header.transform, M_PI_4); // 顺时针移动45°,前面加 "-" 代表逆时针旋转
        self.header.transform = CGAffineTransformScale(self.header.transform, 1.5, 1.5);

    }];
}


// 复位
- (IBAction)goBack:(id)sender {
    self.header.transform = CGAffineTransformIdentity;
}
```

## 小语法点
###不能直接修改OC对象的结构体属性的成员
- 下面的写法是错误的

```objc
imageView.frame.size = imageView.image.size;
```
- 正确写法

```objc
CGRect tempFrame = imageView.frame;
tempFrame.size = imageView.image.size;
imageView.frame = tempFrame;
```

---
