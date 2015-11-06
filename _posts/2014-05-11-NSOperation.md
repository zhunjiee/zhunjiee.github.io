---
layout: post
title: "NSOperation"
date: 2014-05-11
comments: false
categories: 实用技术
---

NSOperation是个抽象类，并不具备封装操作的能力，必须使用它的子类

###NSInvocationOperation
- 如果直接执行NSInvocationOperation中的操作, 那么默认会在主线程中执行

```objc
// 创建队列
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    
// 创建操作
NSInvocationOperation *operation = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(download) object:nil];
// operation直接调用start，是同步执行（在当前线程执行操作）
// [operation start];
    
// 添加操作到队列中，会自动异步执行
[queue addOperation:operation];
```
###NSBlockOperation
+ 如果只封装了一个操作, 那么默认会在主线程中执行
+ 果封装了多个操作, 那么除了第一个操作以外, 其它的操作会在子线程中执行

```objc
NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"1- %@", [NSThread currentThread]);
    }];
[op1 start];
```

###自定义Operation

```objc
@implementation XMGOperation

- (void)main
{
    NSLog(@"%s, %@", __func__,[NSThread currentThread]);
}
@end
```

---

##NSOperationQueue
- GCD队列和NSOperationQueue对比
    + GCD
        * 串行: 自己创建, 主队列
        * 并发: 自己创建, 全局
    + NSOperationQueue
        * 自己创建: alloc/init
        * 主队列  : mainQueue

- NSOperationQueue特点
    + 任务添加到`自己创建`队列中会开启新线程
        * 默认是并发: maxConcurrentOperationCount -1
        * 串行 : maxConcurrentOperationCount = 1
    + 任务添加到`mainQueue`队列中不会开启新线程

- Invocation(了解)

```objc
    // 1.创建队列
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    // 2.封装任务
    NSInvocationOperation *op1 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(demo) object:nil];
    // 3.将任务添加到队列中
    [queue addOperation:op1];
```
- block

```objc
    // 1.创建队列
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    // 2.封装任务
     NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
     NSLog(@"1 = %@", [NSThread currentThread]);
     }];
     // 3.将任务添加到队列中
     [queue addOperation:op1];
```

```objc
    // 1.创建队列
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];

    // addOperationWithBlock方法会做两件事情
    // 1.根据传入的block, 创建一个NSBlockOperation对象
    // 2.将内部创建好的NSBlockOperation对象, 添加到队列中

    // 2.将任务添加到队列中
    [queue addOperationWithBlock:^{
        NSLog(@"1 = %@", [NSThread currentThread]);
    }];
    [queue addOperationWithBlock:^{
        NSLog(@"2 = %@", [NSThread currentThread]);
    }];
```
- 自定义

```objc
    // 1.创建队列
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    // 2.封装任务
    XMGOperation *op1 = [[XMGOperation alloc] init];

    // 3.将任务添加到队列中
    [queue addOperation:op1];
```

- **暂停-恢复**
    + 不会暂停当前正在执行的任务
    + 会从第一个未执行的任务恢复执行

```objc
// 如果是YES, 代表需要暂停
// 如果是NO ,代表恢复执行
self.queue.suspended = YES;
```
- **取消**
    + 不会取消当前正在执行的任务
    + 取消后任务不能恢复
    + 耗时操作应该没执行一段判断一次

```objc
// 内部会调用所有任务的cancel方法
[self.queue cancelAllOperations];
```

- 线程间通信

```objc
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
// 开启子线程
[queue addOperationWithBlock:^{
        // 回到主线程
        [[NSOperationQueue mainQueue] addOperationWithBlock:^{
        }];

}];
```

- **依赖和监听**
    + 只有被依赖的任务完成, 才会执行当前任务
    + 可以跨队列依赖
	+ 不可以相互依赖
```objc
    [operationB addDependency:operationA]; // 操作B依赖于操作A
```

```objc
    op1.completionBlock = ^{
        NSLog(@"第一张图片下载完毕");
    };
    op2.completionBlock = ^{
        NSLog(@"第二张图片下载完毕");
    };
```
---
##从其他线程回到主线程的方式
1. perform
2. GCD
3. NSOperationQueue

```objc
---perform...
[self performSelectorOnMainThread:(SEL) withObject:(id) waitUnitlDone:(BOOL)];

---GCD
dispatch_async(dispatch_get_main_queue(),^{

});

---NSOperationQueue
[NSOperationQueue mainQueue] addOperationWithBlock:^{

}];
```
