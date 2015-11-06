---
layout: post
title: "NSNumber,NSValue,NSFileManager"
date: 2013-11-02
comments: false
categories: ObjC
---

# NSNumber
---
##1.NSNumber基本概念
- NSArray\NSDictionary中只能存放OC对象,不能存放int\float\double等基本数据类

- 如果真想把基本数据(比如int)放进数组或字典中,需要先将基本数据类型包装成OC对象

![](https://dn-zhunjiee.qbox.me/Snip20150708_1.png)
- NSNumber可以将基本数据类型包装成对象,这样就可以间接将基本数据类型存进NSArray\NSDictionary中

![](https://dn-zhunjiee.qbox.me/Snip20150709_1.png)

---

##2.NSNumber的创建
- 以前
    + \+ (NSNumber *)numberWithInt:(int)value;
    + \+ (NSNumber *)numberWithDouble:(double)value;
    + \+ (NSNumber *)numberWithBool:(BOOL)value;

- 现在
    + @10;
    + @10.5;
    + @YES;
    + @(num);

---

##3.从NSNumber对象中的到基本类型数据
- \- (char)charValue;

- \- (int)intValue;

- \- (long)longValue;

- \- (double)doubleValue;

- \- (BOOL)boolValue;

- \- (NSString *)stringValue;

- \- (NSComparisonResult)compare:(NSNumber *)otherNumber; - \- (BOOL)isEqualToNumber:(NSNumber *)number;

---
# 常见的结构体

---

##1.NSPoint和CGPoint
- CGPoint和NSPoint是同义的

```objc
typedef CGPoint NSPoint;

CGPoint的定义
struct CGPoint {
  CGFloat x;
  CGFloat y;
};
typedef struct CGPoint CGPoint;
typedef double CGFloat;
```
- CGPoint代表的是二维平面中的一个点
    + 可以使用CGPointMake和NSMakePoint函数创建CGPoint

---


##2.NSSize和CGSize
- CGSize和NSSize是同义的

```
typedef CGSize NSSize;

CGSize的定义
struct CGSize {
  CGFloat width;
  CGFloat height;
};
typedef struct CGSize CGSize;
```

- CGSize代表的是二维平面中的某个物体的尺寸(宽度和高度)
    + 可以使用CGSizeMake和NSMakeSize函数创建CGSize

---


##3.NSRect和CGRect
- CGRect和NSRect是同义的

```objc
typedef CGRect NSRect;

CGRect的定义
struct CGRect {
  CGPoint origin;
  CGSize size;
};
typedef struct CGRect CGRect;
```

- CGRect代表的是二维平面中的某个物体的位置和尺寸
    + 可以使用CGRectMake和NSMakeRect函数创建CGRect

---

##4.常见的结构体使用注意
-   苹果官方推荐使用CG开头的:
    + CGPoint
    + CGSize
    + CGRect

----


# NSValue

---

##1.NSValue基本概念
- NSNumber是NSValue的子类, 但NSNumber只能包装数字类型

- NSValue可以包装任意值
    + 因此, 可以用NSValue将结构体包装后,加入NSArray\NSDictionary中

---

##2. 常见结构体的包装
- 为了方便 结构体 和NSValue的转换,Foundation提供了以下方法
- 将结构体包装成NSValue对象

    + \+ (NSValue *)valueWithPoint:(NSPoint)point;
    + \+ (NSValue *)valueWithSize:(NSSize)size;
    + \+ (NSValue *)valueWithRect:(NSRect)rect;

- 从NSValue对象取出之前包装的结构体
    + \- (NSPoint)pointValue;
    + \- (NSSize)sizeValue;
    + \- (NSRect)rectValue;

---


# NSFileManager

---

##1.NSFileManager介绍
- 什么是NSFileManager
    + 顾名思义, NSFileManager是用来管理文件系统的
    + 它可以用来进行常见的文件\文件夹操作

- NSFileManager使用了单例模式
    + 使用defaultManager方法可以获得那个单例对象
```objc
[NSFileManager defaultManager]
```

---

##2.NSFileManager用法
- \- (BOOL)fileExistsAtPath:(NSString *)path;
    + path这个文件\文件夹是否存在

```objc
    NSFileManager *manager = [NSFileManager defaultManager];
    // 可以判断文件
    BOOL flag = [manager fileExistsAtPath:@"/Users/Apple/Desktop/why.txt"];
    NSLog(@"flag = %i", flag);
    // 可以判断文件夹
    flag = [manager fileExistsAtPath:@"/Users/Apple/Desktop/未命名文件夹"];
    NSLog(@"flag = %i", flag);
```
- \- (BOOL)fileExistsAtPath:(NSString *)path isDirectory:(BOOL *)isDirectory;
    + path这个文件\文件夹是否存在, isDirectory代表是否为文件夹

```objc
    NSFileManager *manager = [NSFileManager defaultManager];
    BOOL directory = NO;
    BOOL flag = [manager fileExistsAtPath:@"/Users/Apple/Desktop/未命名文件夹" isDirectory:&directory];
    NSLog(@"flag = %i, directory = %i", flag, directory);
```
- \- (BOOL)isReadableFileAtPath:(NSString *)path;
    + path这个文件\文件夹是否可读

- \- (BOOL)isWritableFileAtPath:(NSString *)path;
    + path这个文件\文件夹是否可写
    + >系统目录不允许写入

- \- (BOOL)isDeletableFileAtPath:(NSString *)path;
    + path这个文件\文件夹是否可删除
    + >系统目录不允许删除


---

##3.NSFileManager的文件访问
- \- (NSDictionary *)attributesOfItemAtPath:(NSString *)path error:(NSError **)error;
    + 获得path这个文件\文件夹的属性

```
    NSFileManager *manager = [NSFileManager defaultManager];
    NSDictionary *dict = [manager attributesOfItemAtPath:@"/Users/Apple/Desktop/why.txt" error:nil];
    NSLog(@"dit = %@", dict);
```

- \- (NSArray *)contentsOfDirectoryAtPath:(NSString *)path error:(NSError **)error;
    + 获得path的当前子路径

- \- (NSData *)contentsAtPath:(NSString *)path;
    + 获得文件内容

```objc
    NSFileManager *manager = [NSFileManager defaultManager];
    NSArray *paths = [manager contentsOfDirectoryAtPath:@"/Users/Apple/Desktop/" error:nil];
    NSLog(@"paths = %@", paths);
```

- \- (NSArray *)subpathsAtPath:(NSString *)path;
- \- (NSArray *)subpathsOfDirectoryAtPath:(NSString *)path error:(NSError **)error;
    + 获得path的所有子路径

```objc
    NSFileManager *manager = [NSFileManager defaultManager];
    NSArray *paths = [manager subpathsAtPath:@"/Users/Apple/Desktop/"];
    NSLog(@"paths = %@", paths);
```
---


##4.NSFileManager的文件操作
- \- (BOOL)copyItemAtPath:(NSString *)srcPath toPath:(NSString *)dstPath error:(NSError **)error;
    + 拷贝

- \- (BOOL)moveItemAtPath:(NSString *)srcPath toPath:(NSString *)dstPath error:(NSError **)error;
    + 移动(剪切)

- \- (BOOL)removeItemAtPath:(NSString *)path error:(NSError **)error;
    + 删除

- \- (BOOL)createDirectoryAtPath:(NSString *)path withIntermediateDirectories:(BOOL)createIntermediates attributes:(NSDictionary *)attributes error:(NSError **)error;
    + 创建文件夹(createIntermediates为YES代表自动创建中间的文件夹)

```objc
    NSFileManager *manager = [NSFileManager defaultManager];
    BOOL flag = [manager createDirectoryAtPath:@"/Users/Apple/Desktop/test" withIntermediateDirectories:YES attributes:nil error:nil];
    NSLog(@"flag = %i", flag);
```
- \- (BOOL)createFileAtPath:(NSString *)path contents:(NSData *)data attributes:(NSDictionary *)attr;
    + 创建文件(NSData是用来存储二进制字节数据的)

```objc
    NSString *str = @"why";
    NSData  *data = [str dataUsingEncoding:NSUTF8StringEncoding];
    NSFileManager *manager = [NSFileManager defaultManager];
    BOOL flag = [manager createFileAtPath:@"/Users/Apple/Desktop/abc.txt" contents:data attributes:nil];
    NSLog(@"flag = %i", flag);
```



