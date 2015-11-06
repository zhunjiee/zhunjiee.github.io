---
layout: post
title: "NSInvocation"
date: 2014-05-25
comments: false
categories: 实用技术
---

- **作用**: 用来包装方法和对应的对象,它可以存储方法的名称,对应的对象,对应的参数
  
- **Signature签名**: 在创建NSInvocation的时候, 必须传递一个签名对象
    - 签名对象的作用: 用于获取参数的个数和方法的返回值

- **注意**: 设置参数的索引时不能从0开始, 因为0已经被self占用, 1已经被_cmd占用


- **使用步骤**:
    - 创建签名(创建签名的时候不是使用NSMethodSignature类来创建,而是方法属于谁就用谁来创建)
    - 创建一个NSInvocation对象
    - 设置保存方法所属的对象(target)
    - 设置保存方法名称(selector)
    - 设置参数(setArgument:& atIndex:)
        - 第一个参数需要接收一个指针,也就是传递值的时候需要传递地址;
        - 第二个参数至少从2位置开始
    - 调用NSInvocation对象的invoke方法
        - 只要调用invocation的invoke方法,就代表要执行NSInvocation对象中指定对象的指定方法,并且传递指定的参数

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    // NSInvocation: 用来包装方法和对应的对象, 它可以存储方法的名称, 对应的对象,对应的参数
    
    // 0. 创建签名对象
    // 方法属于谁就用谁调用
    NSMethodSignature *signature = [ViewController instanceMethodSignatureForSelector:@selector(sendMessageWithNumber:andContent:)];
    // 1. 创建一个NSInvocation对象
    NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:signature];
    
    invocation.target = self; // 保存方法所属的对象
    // 给invocation设置的方法,必须和签名中的方法一致
    invocation.selector = @selector(sendMessageWithNumber:andContent:); // 保存方法名称
    
    
    // 第一个参数: 需要给指定方法传递的值
    //      + 第一个参数需要接收一个指针, 也就是传递值的时候需要传递地址
    // 第二个参数: 需要给指定方法的第几个参数传值
    NSString *number = @"10086";
    // 注意: 设置参数的索引时不能从0开始, 因为0已经被self占用, 1已经被_cmd占用
    [invocation setArgument:&number atIndex:2];
    NSString *content = @"hahaha";
    [invocation setArgument:&content atIndex:3];
    
    
    // 2. 调用NSInvocation对象的invoke方法
    [invocation invoke];
}

//sendMessageWithNumber:andContent:
- (void)sendMessageWithNumber:(NSString *)number andContent:(NSString *)content
{
    NSLog(@"发信息给%@, 内容是%@", number, content);
}
```