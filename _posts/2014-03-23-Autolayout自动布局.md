---
layout: post
title: "Autolayout自动布局"
date: 2014-03-23
comments: false
categories: UI
---


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