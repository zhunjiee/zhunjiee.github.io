---
layout: post
title: "NSArray,NSMutableArray"
date: 2013-10-30
comments: false
categories: ObjC
---

#NSArray
---
##1.NSArray的基本概念
- 什么是NSArray?
    + NSArray是OC中的数组类,开发中建议尽量使用NSArray替代C语言中的数组
    + C语言中数组的弊端
        * int array[4] = {10, 89, 27, 76};
        * 只能存放一种类型的数据.(类型必须一致)
        * 不能很方便地动态添加数组元素、不能很方便地动态删除数组元素(长度固定)

- NSArray的使用注意
    + 只能存放任意OC对象, 并且是有顺序的
    + 不能存储非OC对象, 比如int\float\double\char\enum\struct等
    + 它是不可变的,一旦初始化完毕后,它里面的内容就永远是固定的, 不能删除里面的元素, 也不能再往里面添加元素

---

##2.NSArray的创建方式
- \+ (instancetype)array;
- \+ (instancetype)arrayWithObject:(id)anObject;
- \+ (instancetype)arrayWithObjects:(id)firstObj, ...;
- \+ (instancetype)arrayWithArray:(NSArray *)array;

- \+ (id)arrayWithContentsOfFile:(NSString *)path;
- \+ (id)arrayWithContentsOfURL:(NSURL *)url;

---

##3.NSArray 的使用注意事项
- NSArray直接使用NSLog()作为字符串输出时是小括号括起来的形式。

- NSArray中不能存储nil,因为NSArray认为nil是数组的结束(nil是数组元素结束的标记)。nil就是0。0也是基本数据类型,不能存放到NSArray中。

```objc
    NSArray *arr = [NSArray arrayWithObjects:@"why", nil ,@"lmj",@"jjj", nil];
    NSLog(@"%@", arr);
输出结果:
(
    why
)
```
---

##4.NSArray的常用方法
- \- (NSUInteger)count;
    + 获取集合元素个数

- \- (id)objectAtIndex:(NSUInteger)index;
    + 获得index位置的元素

- \- (BOOL)containsObject:(id)anObject;
    + 是否包含某一个元素

- \- (id)lastObject;
    + 返回最后一个元素

- \- (id)firstObject;
    + 返回最后一个元素

- \- (NSUInteger)indexOfObject:(id)anObject;
    + 查找anObject元素在数组中的位置(如果找不到，返回-1)

- \- (NSUInteger)indexOfObject:(id)anObject inRange:(NSRange)range;
    + 在range范围内查找anObject元素在数组中的位置

---

##5.NSArray的简写形式
- 自从2012年开始, Xcode的编译器多了很多自动生成代码的功能, 使得OC代码更加精简

- **一. 数组的创建**
- 之前

```objc
[NSArray arrayWithObjects:@"Jack", @"Rose", @"Jim", nil];
```
- 现在

```objc
@[@"Jack", @"Rose", @"Jim"];
```

- **二. 数组元素的访问**

- 之前

```objc
[array objectAtIndex:0];
```
- 现在

```objc
array[0];
```
---
# NSArray 遍历
---


##1.NSArray的下标遍历

```objc
    NSArray *arr = @[p1, p2, p3, p4, p5];
    for (int i = 0; i < arr.count; ++i) {
        Person *p = arr[i];
        [p say];
    }
```
---

##2.NSArray的快速遍历

```objc
    NSArray *arr = @[p1, p2, p3, p4, p5];
   for (Person *p in arr) {
        [p say];
    }
```
---


##3.NSArray 使用block进行遍历

```objc
    [arr enumerateObjectsUsingBlock:^(id obj, NSUInteger idx, BOOL *stop) {
        NSLog(@"obj = %@, idx = %lu", obj, idx);
        Person *p = obj;
        [p say];
    }];
```
---

##4.NSArray给所有元素发消息
- 让集合里面的所有元素都执行aSelector这个方法
    + \- (void)makeObjectsPerformSelector:(SEL)aSelector;
    + \- (void)makeObjectsPerformSelector:(SEL)aSelector withObject:(id)argument;

```
    // 让数组中所有对象执行这个方法
    // 注意: 如果数组中的对象没有这个方法会报错
//    [arr makeObjectsPerformSelector:@selector(say)];
    [arr makeObjectsPerformSelector:@selector(eat:) withObject:@"bread"];
```
# NSArray排序

---

##1.NSArray排序
- Foundation自带类排序

```objc
NSArray *arr = @[@(1), @(9), @(5), @(2)];
NSArray *newArr = [arr sortedArrayUsingSelector:@selector(compare:)];
```
- 自定义类排序

```objc
    NSArray *arr = @[p1, p2, p3, p4, p5];
    //    默认按照升序排序
    NSArray *newArr = [arr sortedArrayWithOptions:NSSortConcurrent usingComparator:^NSComparisonResult(Person *obj1, Person *obj2) {
        return obj1.age > obj2.age;
    }];
    NSLog(@"%@", newArr);
```
---
# NSArray文件读写

##1.NSArray数据写入到文件中

```objc
    NSArray *arr = @[@"why", @"lmj", @"jjj", @"xcq"];
    BOOL flag = [arr writeToFile:@"/Users/Apple/Desktop/persons.plist" atomically:YES];
    NSLog(@"%i", flag);
```
---

##2.从文件中读取数据到NSArray中

```
    NSArray *newArr = [NSArray arrayWithContentsOfFile:@"/Users/Apple/Desktop/persons.xml"];
    NSLog(@"%@", newArr);
```
---

# NSArray 与字符串

---

##1.把数组元素链接成字符串
- \- (NSString *)componentsJoinedByString:(NSString *)separator;
    + 这是NSArray的方法, 用separator作拼接符将数组元素拼接成一个字符串

```objc
    NSArray *arr = @[@"lmj", @"lnj", @"yz", @"why"];
    NSString *res = [arr componentsJoinedByString:@"*"];
    NSLog(@"res = %@", res);
输出结果:
lmj*lnj*yz*why
```
---

##2.字符串分割方法
- \- (NSArray *)componentsSeparatedByString:(NSString *)separator;
    + 这是NSString的方法,将字符串用separator作为分隔符切割成数组元素

```objc
    NSString *str = @"lmj-lnj-yz-why";
    NSArray *arr = [str componentsSeparatedByString:@"-"];
    NSLog(@"%@", arr);

输出结果:
(
    lmj,
    lnj,
    yz,
    why
)

```
---


# NSMutableArray基本概念
---

##1.NSMutableArray介绍
- 什么是NSMutableArray
    + NSMutableArray是NSArray的子类
    + NSArray是不可变的,一旦初始化完毕后,它里面的内容就永远是固定的, 不能删除里面的元素, 也不能再往里面添加元素
    + NSMutableArray是可变的,随时可以往里面添加\更改\删除元素

---

##2.NSMutableArray基本用法
- 创建空数组

```objc
NSMutableArray *arr = [NSMutableArray array];
```
- 创建数组,并且指定长度为5,此时也是空数组

```objc
NSMutableArray *arr2 = [[NSMutableArray alloc] initWithCapacity:5];
```
- 创建一个数组,包含两个元素

```objc
NSMutableArray *arr3 = [NSMutableArray arrayWithObjects:@"1",@"2", nil];
```
- 调用对象方法创建数组

```objc
NSMutableArray *arr4 = [[NSMutableArray alloc] initWithObjects:@"1",@"2", nil];
```

- \- (void)addObject:(id)object;
    + 添加一个元素

- \- (void)addObjectsFromArray:(NSArray *)array;
    + 添加otherArray的全部元素到当前数组中

- \- (void)insertObject:(id)anObject atIndex:(NSUInteger)index;
    + 在index位置插入一个元素

- \- (void)removeLastObject;
    + 删除最后一个元素

- \- (void)removeAllObjects;
    + 删除所有的元素

- \- (void)removeObjectAtIndex:(NSUInteger)index;
    + 删除index位置的元素

- \- (void)removeObject:(id)object;
    + 删除特定的元素

- \- (void)removeObjectsInRange:(NSRange)range;
    + 删除range范围内的所有元素

- \- (void)replaceObjectAtIndex:(NSUInteger)index withObject:(id)anObject;
    + 用anObject替换index位置对应的元素

- \- (void)exchangeObjectAtIndex:(NSUInteger)idx1 withObjectAtIndex:(NSUInteger)idx2;
    + 交换idx1和idx2位置的元素


---

##3.NSMutableArray 错误用法
- 不可以使用@[]创建可变数组

```objc
NSMutableArray *array = @[@"lmj", @"lnj", @"why"];
// 报错, 本质还是不可变数组
[array addObject:@“Peter”];
```
---

