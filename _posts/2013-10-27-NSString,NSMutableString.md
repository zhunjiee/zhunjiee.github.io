---
layout: post
title: "NSString,NSMutableString"
date: 2013-10-27
comments: false
categories: ObjC
---


#NSString
---
##1.创建字符串
```objc
NSString *str = @"zhunjiee";
NSString *str = [NSString stringWithFormat:@"zhunjiee"];
```
**读取文件内容**
```objc
NSString *str = [NSString stringWithContentsOfFile:@"/Users/zhunjiee/Desktop/zhunjiee.txt" encoding:NSUTF8StringEncoding error:nil];
```
**写入文件**
```objc
[str writeToFile:@"/Users/zhunjiee/Desktop/zhunjiee.txt" atomically:YES encoding:NSUTF8StringEncoding error:nil];
```
##2.URL的创建
```objc
NSURL *url = [NSURL URLWithString:@"http://www.baidu.com/zhunjiee"];
NSURL *url = [NSURL fileURLWithPath:@"Users/zhunjiee/Desktop/zhunjiee.txt"];
```
**从URL读取内容**

```objc
NSString *str = [NSString stringWithContentsOfURL:url encoding:NSUTF8StringEncoding error:nil];
```
**写入URL**

```objc
[str writeToURL:[NSURL URLWithString:@"/Users/why/Desktop/why.txt"] atomically:YES encoding:NSUTF8StringEncoding error:nil];
```
---

#字符串比较
##1.NSString大小写处理
- 全部字符转为大写字母
    + \- (NSString \*)**uppercaseString**;

- 全部字符转为小写字母
    + \- (NSString \*)**lowercaseString**;

- 首字母变大写，其他字母都变小写
    + \- (NSString \*)**capitalizedString**;

---


##2.NSString比较
- \- (BOOL)**isEqualToString**:(NSString *)aString;

    - 两个字符串的内容相同就返回YES, 否则返回NO

```objc
    NSString *str1 = @"why";
    NSString *str2 = [NSString stringWithFormat:@"why"];
    if ([str1 isEqualToString:str2]) {
        NSLog(@"字符串内容一样");
    }

    if (str1 == str2) {
        NSLog(@"字符串地址一样");
    }
```
- \- (NSComparisonResult)**compare**:(NSString *)string;

    + 这个方法可以用来比较两个字符串内容的大小
    + 比较方法: 逐个字符地进行比较ASCII值，返回NSComparisonResult作为比较结果
    + NSComparisonResult是一个枚举，有3个值:
        * 如果左侧   > 右侧,返回NSOrderedDescending,
        * 如果左侧   < 右侧,返回NSOrderedAscending,
        * 如果左侧  == 右侧返回NSOrderedSame

```objc
    NSString *str1 = @"abc";
    NSString *str2 = @"abd";
    switch ([str1 compare:str2]) {
        case NSOrderedAscending:
            NSLog(@"后面一个字符串大于前面一个");
            break;
        case NSOrderedDescending:
            NSLog(@"后面一个字符串小于前面一个");
            break;
        case NSOrderedSame:
            NSLog(@"两个字符串一样");
            break;
    }
    输出结果: 后面一个字符串大于前面一个

```
---

#字符串搜索
##1.字符串搜索
- \- (BOOL)**hasPrefix**:(NSString *)aString;
    + 是否以aString开头

- \- (BOOL)**hasSuffix**:(NSString *)aString;
    + 是否以aString结尾

- \- (NSRange)**rangeOfString**:(NSString *)aString;
    + 用来检查字符串内容中是否包含了aString
    + 如果包含, 就返回aString的范围
    + 如果不包含, NSRange的location为NSNotFound, length为0


---

##2.NSRange基本概念
- NSRange是Foundation框架中比较常用的结构体, 它的定义如下:

```objc
typedef struct _NSRange {
    NSUInteger location;
    NSUInteger length;
} NSRange;
// NSUInteger的定义
typedef unsigned int NSUInteger;
```

- NSRange用来表示事物的一个范围,通常是字符串里的字符范围或者数组里的元素范围

- NSRange有2个成员
    + NSUInteger location : 表示该范围的起始位置
    + NSUInteger length : 表示该范围内的长度

- 比如@“I love OC”中的@“OC”可以用location为7，length为2的范围来表示


---


##3.NSRange的创建
- 有3种方式创建一个NSRange变量
- 方式1

```objc
NSRange range;
range.location = 7;
range.length = 3;
```
- 方式2

```objc
NSRange range = {7, 3};
或者
NSRange range = {.location = 7,.length = 3};
```

- 方式3 : 使用NSMakeRange函数

```objc
NSRange range = NSMakeRange(7, 3);
```
---
# 字符串截取

##1.字符串的截取
- \- (NSString \*)**substringFromIndex**:(NSUInteger)from;
    + 从指定位置from开始(包括指定位置的字符)到尾部

```objc
    NSString *str = @"<head>小码哥</head>";
    str = [str substringFromIndex:7];
    NSLog(@"str = %@", str);

输出结果: 小码哥</head>

```

- \- (NSString \*)**substringToIndex**:(NSUInteger)to;
    + 从字符串的开头一直截取到指定的位置to，但不包括该位置的字符

```objc
    NSString *str = @"<head>小码哥</head>";
    str = [str substringToIndex:10];
    NSLog(@"str = %@", str);

输出结果: <head>小码哥
```

- \- (NSString \*)**substringWithRange**:(NSRange)range;
    + 按照所给出的NSRange从字符串中截取子串

```objc
    NSString *str = @"<head>小码哥</head>";
    NSRange range;
    /*
    range.location = 6;
    range.length = 3;
    */
    range.location = [str rangeOfString:@">"].location + 1;
    range.length = [str rangeOfString:@"</"].location - range.location;
    NSString *res = [str substringWithRange:range];
    NSLog(@"res = %@", res);
输出结果: 小码哥
```
---
###将字符串切割成数组
- \- (NSArray \*)**componentsSeparatedByString**:(NSString *)separator;

```objc
例:
    NSArray *subPaths = [subPath componentsSeparatedByString:@"?"];

```

# 字符串替换

###1.字符串的替换函数
- \- (NSString \*)**stringByReplacingOccurrencesOfString**:(NSString *)target **withString**:(NSString *)replacement;
    + 用replacement替换target

```objc
    NSString *str = @"http:**520it.com*img*why.gif";
    NSString *newStr = [str stringByReplacingOccurrencesOfString:@"*" withString:@"/"];
    NSLog(@"newStr = %@", newStr);

输出结果: http://www.520it.com/img/why.gif
```

---
# 字符串与路径

##1.NSString与路径
- \- (BOOL)**isAbsolutePath**;
    + 是否为绝对路径

```objc
     // 其实就是判断是否以/开头
//    NSString *str = @"/Users/Apple/Desktop/why.txt";
    NSString *str = @"Users/Apple/Desktop/why.txt";
    if ([str isAbsolutePath]) {
        NSLog(@"是绝对路径");
    }else
    {
        NSLog(@"不是绝对路径");
    }
```
- \- (NSString \*)**lastPathComponent**;
    + 获得最后一个目录

```objc
    // 截取最后一个/后面的内容
    NSString *str = @"/Users/Apple/Desktop/why.txt";
    NSString *component = [str lastPathComponent];
    NSLog(@"component = %@", component);
```
- \- (NSString \*)stringByDeletingLastPathComponent;
    + 删除最后一个目录

```objc
    // 其实就是上次最后一个/和之后的内容
    NSString *str = @"/Users/Apple/Desktop/why.txt";
    NSString *newStr = [str stringByDeletingLastPathComponent];
    NSLog(@"newStr = %@", newStr);
```
- \- (NSString \*) **stringByAppendingPathComponent** :(NSString \*)str;
    + 在路径的后面拼接一个目录
(也可以使用stringByAppendingString:或者stringByAppendingFormat:拼接字符串内容)

```objc
// 其实就是在最后面加上/和要拼接得内容
    // 注意会判断后面有没有/有就不添加了, 没有就添加, 并且如果有多个会替换为1个
//    NSString *str = @"/Users/Apple/Desktop";
    NSString *str = @"/Users/Apple/Desktop/";
    NSString *newStr = [str stringByAppendingPathComponent:@"why"];
    NSLog(@"newStr = %@", newStr);
```
---

##2.NSString与文件拓展名
- \- (NSString \*)pathExtension;
    + 获得拓展名

```objc
    // 其实就是从最后面开始截取.之后的内容
//    NSString *str = @"why.txt";
    NSString *str = @"abc.why.txt";
    NSString *extension = [str pathExtension];
    NSLog(@"extension = %@", extension);
```
- \- (NSString \*)stringByDeletingPathExtension;
    + 删除尾部的拓展名

```objc
    // 其实就是上次从最后面开始.之后的内容
//    NSString *str = @"why.txt";
    NSString *str = @"abc.why.txt";
    NSString *newStr = [str stringByDeletingPathExtension];
    NSLog(@"newStr = %@", newStr);
```
- \- (NSString \*)stringByAppendingPathExtension:(NSString *)str;
    + 在尾部添加一个拓展名

```objc
// 其实就是在最后面拼接上.和指定的内容
    NSString *str = @"why";
    NSString *newStr = [str stringByAppendingPathExtension:@"gif"];
    NSLog(@"newStr = %@", newStr);
```
---
# 字符串与基本数据类型转换

##1.字符串长度及索引处字符
- \- (NSUInteger)length;
    + 返回字符串的长度(有多少个文字)

- \- (unichar)characterAtIndex:(NSUInteger)index;
    + 返回index位置对应的字符

---

##2.字符串和其他数据类型转换
- 转为基本数据类型
    + \- (double)**doubleValue**;
    + \- (float)**floatValue**;
    + \- (int)**intValue**;

```objc
    NSString *str1 = @"110";
    NSString *str2 = @"10";
    int res = str1.intValue + str2.intValue;
    NSLog(@"res = %i", res);
```

```objc
    NSString *str1 = @"110";
    NSString *str2 = @"10.1";
    double res = str1.doubleValue + str2.doubleValue;
    NSLog(@"res = %f", res);
```
- 转为C语言中的字符串
    + \- (char \*)**UTF8String**;

```objc
    NSString *str = @"abc";
    const char *cStr = [str UTF8String];
    NSLog(@"cStr = %s", cStr);
```

```objc
    char *cStr = "abc";
    NSString *str = [NSString stringWithUTF8String:cStr];
    NSLog(@"str = %@", str);
```
---



# NSMutableString常用方法

##1.NSMutableString常用方法
- \- (void)**appendString**:(NSString *)aString;
    + 拼接aString到最后面


- \- (void)**appendFormat**:(NSString *)format, ...;
    + 拼接一段格式化字符串到最后面

```objc
    NSMutableString *strM = [NSMutableString string];
    NSLog(@"strM = %@", strM);
    [strM appendString:@"why"];
    NSLog(@"strM = %@", strM);
```
- \- (void)**deleteCharactersInRange**:(NSRange)range;
    + 删除range范围内的字符串

```objc
    NSMutableString *strM = [NSMutableString stringWithString:@"http://www.520it.com"];
     // 一般情况下利用rangeOfString和deleteCharactersInRange配合删除指定内容
     NSRange range = [strM rangeOfString:@"http://"];
     [strM deleteCharactersInRange:range];
     NSLog(@"strM = %@", strM);
```
- \- (void)**insertString**:(NSString \*)aString **atIndex**:(NSUInteger)loc;
    + 在loc这个位置中插入aString

```objc
    NSMutableString *strM = [NSMutableString stringWithString:@"www.520it.com"];
    [strM insertString:@"http://" atIndex:0];
    NSLog(@"strM = %@", strM);

```
- \- (void)**replaceCharactersInRange**:(NSRange)range **withString**:(NSString \*)aString;
    + 使用aString替换range范围内的字符串

```objc
    NSMutableString *strM = [NSMutableString stringWithString:@"http://www.520it.com/why.png"];
    NSRange range = [strM rangeOfString:@"why"];
    [strM replaceOccurrencesOfString:@"why" withString:@"jjj" options:0 range:range];
    NSLog(@"strM = %@", strM);
```

---

##2.字符串使用注意事项
- @”zhunjiee”这种方式创建的字符串始终是NSString,不是NSMutalbeString.所以下面的代码创建的还是NSString,此时使用可变字符串的函数,无法操作字符串。

```objc
NSMutalbeString *s1 = @”why”;
// 会报错
[strM insertString:@"my name is " atIndex:0];
```
---
##NSData类型转NSString类型
```objc
// NSData类型转NSString类型
NSString *str = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
```

