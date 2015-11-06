---
layout: post
title: "@property各参数含义解释,@class"
date: 2013-10-15
comments: false
categories: ObjC
---

#@property各参数含义解释

##1.控制setter方法的内存管理
- retain ： release旧值，retain新值（用于OC对象）
- assign ： 直接赋值，不做任何内存管理(默认，用于非OC对象类型)
- copy   ： release旧值，copy新值（一般用于NSString *）

```objc
.h文件
@interface BWString : NSObject
/** retain */
@property (nonatomic, retain) NSString *name;
/** copy */
@property (nonatomic, copy) NSString *name1;
@end

.m 文件
#import "BWString.h"
@implementation BWString
- (void)setName:(NSString *)name{
    if (name != _name) {
        [_name release];
        _name = [name retain];
    }
}

- (void)setName1:(NSString *)name1{
    if (name1 != _name1) {
        [_name1 release];
        _name1 = [name1 copy];
    }
}
@end
```

##2.控制需不需生成setter方法
- readwrite ：同时生成setter方法和getter方法（默认）
- readonly  ：只会生成getter方法

##3.多线程管理
- atomic    ：性能低（默认）
- nonatomic ：性能高

##4.控制setter方法和getter方法的名称
- setter ： 设置setter方法的名称，一定有个冒号:
- getter ： 设置getter方法的名称


- 注意: 不同类型的参数可以组合在一起使用

---

#@class

##1.@class基本概念
- 作用
    + 可以简单地引用一个类

- 简单使用
    + @class Dog;
    + 仅仅是告诉编译器:Dog是一个类;并不会包含Dog这个类的所有内容

- 具体使用
    + 在.h文件中使用@class引用一个类
    + 在.m文件中使用#import包含这个类的.h文件


##2.@class其它应用场景
- 对于循环依赖关系来说，比方A类引用B类，同时B类也引用A类
- 这种嵌套包含的代码编译会报错

```
#import "B.h"
@interface A : NSObject
{
    B *_b;
}
@end

#import “A.h"
@interface B : NSObject
{
    A *_a;
}
@end

```

- 当使用@class在两个类相互声明，就不会出现编译报错

```
@class B;
@interface A : NSObject
{
    B *_b;
}
@end

@class A;
@interface B : NSObject
{
    A *_a;
}
@end
```


##3.@class和#import
- 作用上的区别
    + #import会包含引用类的所有信息(内容),包括引用类的变量和方法
    + @class仅仅是告诉编译器有这么一个类, 具体这个类里有什么信息, 完全不知

- 效率上的区别
    + 如果有上百个头文件都#import了同一个文件，或者这些文件依次被#import,那么一旦最开始的头文件稍有改动，后面引用到这个文件的所有类都需要重新编译一遍 , 编译效率非常低
    + 相对来讲，使用@class方式就不会出现这种问题了

---


# 循环retian

- 循环retain的场景
    + 比如A对象retain了B对象，B对象retain了A对象

- 循环retain的弊端
    + 这样会导致A对象和B对象永远无法释放

- 循环retain的解决方案
    + 当两端互相引用时，应该一端用retain、一端用assign

---
