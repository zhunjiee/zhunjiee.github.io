---
layout: post
title: "iOS开发之 - JSON,XML与OC之间的转换"
date: 2014-05-20
comments: false
categories: 实用技术
---

##注意:
对于比较复杂的json数据,拿到数据后可能不会在tableview上显示,一定记得刷新数据

```objc
// 注意:拿到数据之后一定要刷新表格
[self.tableView reloadData];
```
---
##JSON数据（NSData） -> OC对象（Foundation Object）
- JSON和OC对象转换后对应数据类型
    + {} -> NSDictionary @{}
    + [] -> NSArray @[]
    + "jack" -> NSString @"jack"
    + 10 -> NSNumber @10
    + 10.5 -> NSNumber @10.5
    + true -> NSNumber @1
    + false -> NSNumber @0
    + null -> NSNull

- 转换方法

```objc
// 利用NSJSONSerialization类
+ (id)JSONObjectWithData:(NSData *)data options:(NSJSONReadingOptions)opt error:(NSError **)error;
```
例:
```objc
NSDictionary *dict = [NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingMutableContainers error:nil];
```
- NSJSONReadingOptions
    - NSJSONReadingMutableContainers = (1UL << 0)
        - 创建出来的数组和字典就是可变
    - NSJSONReadingMutableLeaves = (1UL << 1)
        - 数组或者字典里面的字符串是可变的, iOS7以后无效
    - NSJSONReadingAllowFragments
        - 允许解析出来的对象不是字典或者数组，比如直接是字符串或者NSNumber

## OC对象（Foundation Object）-> JSON数据（NSData）
- 转换方法

```objc
// 利用NSJSONSerialization类
+ (NSData *)dataWithJSONObject:(id)obj options:(NSJSONWritingOptions)opt error:(NSError **)error;
```
例:
```objc
NSData *data = [NSJSONSerialization dataWithJSONObject:dict options:NSJSONWritingPrettyPrinted error:nil];
```
- 格式化服务器返回的JSON数据
    + 在线格式化：http://tool.oschina.net/codeformat/json
    + 将服务器返回的字典或者数组写成plist文件

## 字典转模型框架
- Mantle
    - 所有模型都必须继承自MTModel
- JSONModel
    - 所有模型都必须继承自JSONModel
- MJExtension
    - 不需要强制继承任何其他类
```objc
    // 由于id是oc关键字,不适合做key,所以在字典转模型之前,告诉框架要将模型中的哪个属性和字典中的哪个key对应
    [HBWVideo setupReplacedKeyFromPropertyName:^NSDictionary *{
        return @{@"ID" : @"id"};
    }
    
    // 字典转模型
    self.videos = [HBWVideo objectArrayWithKeyValuesArray:dict[@"videos"]];
```

## 设计框架需要考虑的问题
- 侵入性
    - 侵入性大就意味着很难离开这个框架
- 易用性
    - 比如少量代码实现N多功能
- 扩展性
    - 很容易给这个框架增加新框架

## 利用苹果官方API播放视频
```objc
#import <MediaPlayer/MPMoviePlayerViewController.h>

// 创建视频播放器
MPMoviePlayerViewController *vc = [[MPMoviePlayerViewController alloc] initWithContentURL:[NSURL URLWithString:urlStr]];

// 显示视频
[self presentViewController:vc animated:YES completion:nil];
```
---
## XML的解析方式
- SAX
    - 大小文件都可以
    - NSXMLParser
- DOM
    - 最好是小文件
    - GDataXML

## NSXMLParser的用法
- 创建解析器来解析

```objc
// 创建XML解析器
NSXMLParser *parser = [[NSXMLParser alloc] initWithData:data];

// 设置代理
parser.delegate = self;

// 开始解析XML(parse方法是阻塞式的)
[parser parse];
```

- 代理对象要遵守NSXMLParserDelegate协议，实现代理方法

```objc
/**
 * 解析到某个元素的结尾（比如解析</videos>）
 */
- (void)parser:(NSXMLParser *)parser didEndElement:(NSString *)elementName namespaceURI:(NSString *)namespaceURI qualifiedName:(NSString *)qName
{

}

/**
 * 解析到某个元素的开头（比如解析<videos>）
 */
- (void)parser:(NSXMLParser *)parser didStartElement:(NSString *)elementName namespaceURI:(NSString *)namespaceURI qualifiedName:(NSString *)qName attributes:(NSDictionary *)attributeDict
{

}

/**
 * 开始解析XML文档
 */
- (void)parserDidStartDocument:(NSXMLParser *)parser
{

}

/**
 * 解析完毕
 */
- (void)parserDidEndDocument:(NSXMLParser *)parser
{
    
// 刷新UI
    [self.tableView reloadData];
}
```
## GDataXML
- 配置

![](https://dn-zhunjiee.qbox.me/Snip20150907_3.png)
![](https://dn-zhunjiee.qbox.me/Snip20150907_4.png)

- 设置非ARC标记

![](https://dn-zhunjiee.qbox.me/Snip20150907_1.png)

- 具体用法

    ```objc
    // 创建解析器
        GDataXMLDocument *doc = [[GDataXMLDocument alloc] initWithData:data options:kNilOptions error:nil];
        // 获取根元素
        GDataXMLElement *rootElement = doc.rootElement;
        // 从根元素中获取所有子元素
        NSArray *elements = [rootElement elementsForName:@"video"];
        // 遍历子元素,取出属性转换为模型
        for (GDataXMLElement *element in elements) {
            HBWVideo *video = [[HBWVideo alloc] init];
            video.ID = @([element attributeForName:@"id"].stringValue.integerValue);
            video.image = [element attributeForName:@"image"].stringValue;
            video.url = [element attributeForName:@"url"].stringValue;
            video.name = [element attributeForName:@"name"].stringValue;
            video.length = @([element attributeForName:@"length"].stringValue.integerValue);
            
            [self.videos addObject:video];
        }
        
        // 刷新表格
        [self.tableView reloadData];
    ```

---

##输出打印中文问题
- Xcode版本问题
- 重写字典的descriptionWithLocale方法
- 该方法系统自动调用,不需要我们手动调用

```objc
#import <Foundation/Foundation.h>

@implementation NSDictionary (LOG)


// 只要打印一个数组, 或者字典, 系统就会自动调用该方法
- (NSString *)descriptionWithLocale:(id)locale
{
    // 1.定义一个可变的字符串, 保存拼接结果
    NSMutableString *strM = [NSMutableString string];
    [strM appendString:@"{\n"];
    // 2.迭代字典中所有的key/value, 将这些值拼接到字符串中
    [self enumerateKeysAndObjectsUsingBlock:^(id key, id obj, BOOL *stop) {
        [strM appendFormat:@"\t%@ = %@,\n", key, obj];
    }];
    [strM appendString:@"}"];
    
    // 删除最后一个逗号
    if (self.allKeys.count > 0) {
        NSRange range = [strM rangeOfString:@"," options:NSBackwardsSearch];
        [strM deleteCharactersInRange:range];
    }
    
    // 3.返回拼接好的字符串
    return strM;
}
@end


@implementation NSArray (LOG)
- (NSString *)descriptionWithLocale:(id)locale
{
    // 1.定义一个可变的字符串, 保存拼接结果
    NSMutableString *strM = [NSMutableString string];
    [strM appendString:@"(\n"];
    // 2.迭代字典中所有的key/value, 将这些值拼接到字符串中
    [self enumerateObjectsUsingBlock:^(id obj, NSUInteger idx, BOOL *stop) {
        [strM appendFormat:@"\t%@,\n", obj];
    }];
    [strM appendString:@")\n"];
    
    // 删除最后一个逗号
    if (self.count > 0) {
        NSRange range = [strM rangeOfString:@"," options:NSBackwardsSearch];
        [strM deleteCharactersInRange:range];
    }
    
    // 3.返回拼接好的字符串
    return strM;
}
@end
```