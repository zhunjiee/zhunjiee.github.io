---
layout: post
title: "autorelease基本使用"
date: 2013-10-16
comments: false
categories: ObjC
---


##1.autorelease基本概念
- autorelease是一种支持引用计数的内存管理方式,只要给对象发送一条autorelease消息,会将对象放到一个自动释放池中,当自动释放池被销毁时，会对池子里面的`所有对象做一次release操作`
    + > 注意,这里只是发送release消息,如果当时的引用计数(reference-counted)依然不为0,则该对象依然不会被释放。

- autorelease方法会返回对象本身

```
Person *p = [Person new];
p = [p autorelease];
```

- 调用完autorelease方法后，对象的计数器不变

```
Person *p = [Person new];
p = [p autorelease];
NSLog(@"count = %lu", [p retainCount]); // 1
```

- autorelease的好处
    + 不用再关心对象释放的时间
    + 不用再关心什么时候调用release

- autorelease的原理
    + autorelease实际上只是把对release的调用延迟了,对于每一个autorelease,系统只是把该 Object放入了当前的autorelease pool中,当该pool被释放时,该pool中的所有Object会被调用Release。



##2.自动释放池
- 创建自动释放池格式:
- iOS 5.0前

```
NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init]; // 创建自动释放池
[pool release]; // [pool drain]; 销毁自动释放池
```
- iOS 5.0 开始

```
@autoreleasepool
{ //开始代表创建自动释放池

} //结束代表销毁自动释放池
```
- 在iOS程序运行过程中，会创建无数个池子。这些池子都是以栈结构存在（先进后出）

- 当一个对象调用autorelease方法时，会将这个对象放到栈顶的释放池



##3.autorelease基本使用

```
NSAutoreleasePool *autoreleasePool = [[NSAutoreleasePool alloc] init];

Person *p = [[[Person alloc] init] autorelease];

[autoreleasePool drain];
```

```
@autoreleasepool
{ // 创建一个自动释放池
        Person *p = [[Person new] autorelease];
} // 销毁自动释放池(会给池子中所有对象发送一条release消息)
```
---
# autorelease注意事项


##1.autorelease使用注意
- 并不是放到自动释放池代码中,都会自动加入到自动释放池

```
 @autoreleasepool {
    // 因为没有调用 autorelease 方法,所以对象没有加入到自动释放池
    Person *p = [[Person alloc] init];
    [p run];
}
```

- 在自动释放池的外部发送autorelease 不会被加入到自动释放池中
    + autorelease是一个方法,只有在自动释 放池中调用才有效。

```
 @autoreleasepool {
 }
 // 没有与之对应的自动释放池, 只有在自动释放池中调用autorelease才会放到释放池
 Person *p = [[[Person alloc] init] autorelease];
 [p run];

 // 正确写法
  @autoreleasepool {
    Person *p = [[[Person alloc] init] autorelease];
 }

 // 正确写法
 Person *p = [[Person alloc] init];
  @autoreleasepool {
    [p autorelease];
 }

```

- 自动释放池的嵌套使用
    + 自动释放池是以栈的形式存在
    + 由于栈只有一个入口, 所以调用autorelease会将对象放到栈顶的自动释放池
    + >栈顶就是离调用autorelease方法最近的自动释放池
```
@autoreleasepool { // 栈底自动释放池
        @autoreleasepool {
            @autoreleasepool { // 栈顶自动释放池
                Person *p = [[[Person alloc] init] autorelease];
            }
            Person *p = [[[Person alloc] init] autorelease];
        }
    }
```

- 自动释放池中不适宜放占用内存比较大的对象
    + 尽量避免对大内存使用该方法,对于这种延迟释放机制,还是尽量少用
    + 不要把大量循环操作放到同一个 @autoreleasepool 之间,这样会造成内存峰值的上升

```
// 内存暴涨
    @autoreleasepool {
        for (int i = 0; i < 99999; ++i) {
            Person *p = [[[Person alloc] init] autorelease];
        }
    }
```

```
// 内存不会暴涨
 for (int i = 0; i < 99999; ++i) {
        @autoreleasepool {
            Person *p = [[[Person alloc] init] autorelease];
        }
    }
```
---

##2.autorelease错误用法
- 不要连续调用autorelease

```
 @autoreleasepool {
 // 错误写法, 过度释放
    Person *p = [[[[Person alloc] init] autorelease] autorelease];
 }
```
- 调用autorelease后又调用release

```
 @autoreleasepool {
    Person *p = [[[Person alloc] init] autorelease];
    [p release]; // 错误写法, 过度释放
 }
```
---
# autorelease 的应用场景


##1.快速创建对象的方法
- 一般可以为类添加一个快速创建对象的类方法
    + 外界调用[Person person]就可以获得和使用新建的Person对象，根本不用考虑在什么时候释放Person对象

```
@implementation Person
+ (instancetype)person
{
    return [[[self alloc] init] autorelease];
}
- (instancetype)initWithAge:(int)age
{
    if (self = [super init]) {
        _age = age;
    }
    return self;
}

+ (instancetype)personWithAge:(int)age
{
    Person *p = [[[self alloc] initWithAge:age] autorelease];
    return p;
}
@end
```

- 一般来说,除了alloc、new或copy之外的方法创建的对象都被声明了autorelease
    + 比如下面的对象都已经是autorelease的，不需要再release

```
NSNumber *nb = [NSNumber numberWithInt:100];
NSString *sb = [NSString stringWithFormat:@"why"];
NSString *xsb = @"rose";

```

---
