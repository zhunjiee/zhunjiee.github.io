---
layout: post
title: "单例(singleton)"
date: 2014-05-15
comments: false
categories: 实用技术
---
###什么是单例模式
- **单例模式** 是一个类在系统中只有一个实例对象。通过全局的一个入口点对这个实例对象进行访问。在iOS开发中，单例模式是非常有用的一种设计模式。

- **可以保证在程序运行过程，一个类只有一个实例**

---
###实现单例模式的条件

- 实现单例模式有三个条件:
	1. 类的构造方法是私有的
	2. 类提供一个类方法用于产生对象
	3. 类中有一个私有的自己对象

- 针对于这三个条件，OC中都是可以做到的
	1. 类的构造方法是私有的
我们只需要重写allocWithZone方法，让初始化操作只执行一次
	2. 类提供一个类方法产生对象
这个可以直接定义一个类方法
	3. 类中有一个私有的自己对象
我们可以在.m文件中定义一个属性即可

---


###应用场景及注意点

- **应用场景**
    + 某个类经常被使用(节约系统资源)
    + 定义工具类
    + 共享数据

- **注意点**
    + 不要继承单例类
        * 先创建子类永远是子类对象
        * 先创建父类永远是父类对象

- **单例模式:**
	+ **懒汉模式** : 第一次用到单例对象的时候再创建
	+ **饿汉模式** : 一进入程序就创建一个单例对象


##ARC

---
###懒汉模式
```objc
#import "Singleton.h"

@implementation Singleton
static id _instance;

/**
 *  alloc方法内部会调用这个方法
 */
+ (instancetype)allocWithZone:(struct _NSZone *)zone{
    if (_instance == nil) { // 防止频繁加锁
        @synchronized(self) {
            if (_instance == nil) { // 防止创建多次
                _instance = [super allocWithZone:zone];
            }
        }
    }
    return _instance;
}

+ (instancetype)sharedSingleton{
    if (_instance == nil) { // 防止频繁加锁
        @synchronized(self) {
            if (_instance == nil) { // 防止创建多次
                _instance = [[self alloc] init];
            }
        }
    }
    return _instance;
}

- (id)copyWithZone:(NSZone *)zone{
    return _instance;
}
@end
```
###饿汉模式(不常用)
```objc
#import "HMSingleton.h"

@implementation Singleton
static id _instance;

/**
 *  当类加载到OC运行时环境中（内存），就会调用一次（一个类只会加载1次）
 */
+ (void)load{
    _instance = [[self alloc] init];
}

+ (instancetype)allocWithZone:(struct _NSZone *)zone{
    if (_instance == nil) { // 防止创建多次
        _instance = [super allocWithZone:zone];
    }
    return _instance;
}

+ (instancetype)sharedSingleton{
    return _instance;
}

- (id)copyWithZone:(NSZone *)zone{
    return _instance;
}
@end
```

###GCD实现单例模式 - 开发常用方式
```objc
@implementation Singleton
static id _instance;

+ (instancetype)allocWithZone:(struct _NSZone *)zone{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _instance = [super allocWithZone:zone];
    });
    return _instance;
}

+ (instancetype)sharedSingleton{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _instance = [[self alloc] init];
    });
    return _instance;
}

- (id)copyWithZone:(NSZone *)zone{
    return _instance;
}
@end
```



---
##非ARC
在非ARC的环境下,需要再加上下面的方法:

- 重写release方法为空
- 重写retain方法返回自己
- 重写retainCount返回1
- 重写autorelease返回自己

```objc
- (oneway void)release { }
- (id)retain { return self; }
- (NSUInteger)retainCount { return 1;}
- (id)autorelease { return self;}

```

- **如何判断是否是ARC**

```objc
#if __has_feature(objc_arc)
//ARC环境
#else
//MRC环境
#endif
```



---

