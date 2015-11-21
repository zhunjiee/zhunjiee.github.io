---
layout: post
title: "block内指针分析与__block的使用详解"
date: 2013-11-10
comments: false
categories: 知识囊
---


##block内指针分析

- block内部如果通过外面声明的**强引用**使用某个对象,那么block内部就会额外产生一个**强引用**指向该对象
- block内部如果通过外面声明的**弱引用**使用某个对象,那么block内部就会额外产生一个**弱引用**指向该对象

默认情况下block是放在**栈**里面的,不会产生上述现象,
但通过copy创建的block放在**堆**里面,会产生上述现象.

- 为了防止在block里用self导致有强引用不能及时释放的问题,我们在block外做如下定义:
`__weak typeof(weakSelf) weakSelf = self;`

- 但有时候我们还会看到在block里又会有像下面一样的定义:
`__strong typeof(strongSelf) strongSelf = weakSelf;`

例:

```
    XMGStudent *stu = [[XMGStudent alloc] init];
    __weak XMGStudent *weakStu = stu;
    
    stu.block = ^{
        NSLog(@"begin - block");
        XMGStudent *strongStu = weakStu;
        
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            [strongStu study];
        });
    };
    stu.block();
```

为什么要这样写呢?

- 我们根据以上那个代码分析:

	- 1.直接用外部声明的变量会导致block内部产生额外的强引用,代码执行完后额外的强指针不会被销毁,导致不能释放的问题
	
	- 2.全部用弱指针(即在block内再声明局部**弱指针**),dispatch_after这个block内部调用该弱指针对象,会额外产生指向该对象的弱引用.在代码由上到下执行时,定义是里面的内容会被dispatch_after保存起来,因为里面的对象是弱指针引用的,所以对象会被销毁,所以输出的结果会有问题
	
	- 3.用局部的强指针(即在block内再声明局部**强指针**),dispatch_after这个block内部调用该强指针对象,会额外产生指向该对象的强引用.在代码由上到下执行时,定义是里面的内容会被dispatch_after保存起来,因为有强指针引用里面的对象,所以对象不会销毁,等到dispatch_after执行完毕后block销毁,里面的对象才销毁

##总结:block开发使用注意
1. block原理: block会把代码块里的所有强指针强引用
2. block里面不要使用self,可能会造成循环引用
3. block里尽量不要引用下划线的成员属性
4. 解决循环引用

```objc
// typeof:获取类型
__weak typeof(self) weakSelf = self;
```
---

##__block详解
- ﻿__block 的标记的作用是告诉编译器，这个变量在 block 里面需要做特殊处理。

- 一般来说，在 block 中用的变量值是被复制过来的，所以对于变量本身的修改并不会影响这个变量的真实值。而当我们用 __block 标记的时候，表示在 block 中的修改对于 block 外也是有效地。

```objc
extern NSInteger CounterGlobal;
static NSInteger CounterStatic;
 
{
    NSInteger localCounter = 42;
    __block char localCharacter;
 
    void (^aBlock)(void) = ^(void) {
        ++CounterGlobal;
        ++CounterStatic;
        CounterGlobal = localCounter; // localCounter fixed at block creation
        localCharacter = 'a'; // sets localCharacter in enclosing scope
    };
 
    ++localCounter; // unseen by the block
    localCharacter = 'b';
 
    aBlock(); // execute the block
    // localCharacter now 'a'
}
```
- 在上面的代码里，localConter 和 localCharacter 都在 block 中有所修改，但是在 block 里面，只有 localCharacter 的修改是有效的，原因是 __block 标记起了作用。而在 block 中对于 localCharacter 的修改在 block 外也是可见的。