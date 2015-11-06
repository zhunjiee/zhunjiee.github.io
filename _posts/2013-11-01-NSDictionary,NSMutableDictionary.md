---
layout: post
title: "NSDictionary,NSMutableDictionary"
date: 2013-11-01
comments: false
categories: ObjC
---

#NSDictionary

##1.NSDictionary基本概念
- 什么是NSDictionary
    + NSDictionary翻译过来叫做”字典”
    + 日常生活中,“字典”的作用:通过一个拼音或者汉字,就能找到对应的详细解释
    + NSDictionary的作用类似:通过一个key,就能找到对应的value
    + NSDictionary是不可变的, 一旦初始化完毕, 里面的内容就无法修改

---

##2.NSDictionary的创建

+ \+ (instancetype)dictionary;
+ \+ (instancetype)dictionaryWithObject:(id)object forKey:(id <NSCopying>)key;
+ \+ (instancetype)dictionaryWithObjectsAndKeys:(id)firstObject, ...;
+ \+ (id)dictionaryWithContentsOfFile:(NSString *)path;
+ \+ (id)dictionaryWithContentsOfURL:(NSURL *)url;


####NSDictionary创建简写

- 以前

```objc
NSDictionary *dict = [NSDictionary dictionaryWithObjectsAndKeys:@"why", @"name", @"12345678", @"phone", @"天朝", @"address", nil];
```
- 现在

```objc
NSDictionary *dict = @{@"name":@"why", @"phone":@"12345678", @"address":@"天朝"};
```

####NSDictionary获取元素简写

- 以前

```objc
[dict objectForKey:@"name”];
```
- 现在

```objc
dict[@"name”];
```


- 键值对集合的特点
    + 字典存储的时候,必须是"键值对"的方式来存储(同时键不要重复)
    + 键值对中存储的数据是"无序的".
    + 键值对集合可以根据键, 快速获取数据.

##3.NSDictionary的遍历
- \- (NSUInteger)count;
    + 返回字典的键值对数目

- \- (id)objectForKey:(id)aKey;
    + 根据key取出value


- 快速遍历

```objc
    NSDictionary *dict = @{@"name":@"why", @"phone":@"12345678", @"address":@"天朝"};
    for (NSString *key in dict) {
        NSLog(@"key = %@, value = %@", key, dict[key]);
    }
```
- Block遍历

```objc
    [dict enumerateKeysAndObjectsUsingBlock:^(NSString *key, NSString *obj, BOOL *stop) {
        NSLog(@"key = %@, value = %@", key, obj);
    }];

```
---
##4.NSDictionary文件操作
- 将字典写入文件中
    + \- (BOOL)writeToFile:(NSString *)path atomically:(BOOL)useAuxiliaryFile;
    + \- (BOOL)writeToURL:(NSURL *)url atomically:(BOOL)atomically;
    + 存结果是xml文件格式,但苹果官方推荐为plist后缀。

- 示例

```objc
    NSDictionary *dict = @{@"name":@"why", @"phone":@"12345678", @"address":@"天朝"};
    BOOL flag = [dict writeToFile:@"/Users/Apple/Desktop/dict.plist" atomically:YES];
    NSLog(@"flag = %i", flag);
```

- 从文件中读取字典

```objc
NSDictionary *newDict = [NSDictionary dictionaryWithContentsOfFile:@"/Users/Apple/Desktop/dict.plist"];
    NSLog(@"newDict = %@", newDict);
```
# NSMutableDictionary基本概念


---

##1.NSMutableDictionary 基本概念
- 什么是NSMutableDictionary
    + NSMutableDictionary是NSDictionary的子类
    + NSDictionary是不可变的,一旦初始化完毕后,它里面的内容就永远是固定的,不能删除里面的元素, 也不能再往里面添加元素
    + NSMutableDictionary是可变的,随时可以往里面添加\更改\删除元素

---


##2.NSMutableDictionary的常见操作
- \- (void)setObject:(id)anObject forKey:(id <NSCopying>)aKey;
    + 添加一个键值对(会把aKey之前对应的值给替换掉)

- \- (void)removeObjectForKey:(id)aKey;
    + 通过aKey删除对应的value

- \- (void)removeAllObjects;
    + 删除所有的键值对

##3.NSMutableDictionary的简写
####设置键值对
+ 以前

```
[dict setObject:@"Jack" forKey:@"name”];
```
+ 现在

```
dict[@"name"] = @"Jack";
```

---
##4.NSDictionary和NSArray对比
- NSArray和NSDictionary的区别
    + NSArray是有序的,NSDictionary是无序的
    + NSArray是通过下标访问元素,NSDictionary是通过key访问元素

####NSArray的用法
- 创建

```objc
@[@"Jack", @"Rose"] (返回是不可变数组)
```
- 访问

```objc
id d = array[1];
```
- 赋值

```objc
array[1] = @"jack";
```

####NSDictionary的用法

- 创建

```objc
@{ @"name" : @"Jack", @"phone" : @"10086" } (返回是不可变字典)
```
- 访问

```objc
id d = dict[@"name"];
```
- 赋值

```objc
dict[@"name"] = @"jack";
```



