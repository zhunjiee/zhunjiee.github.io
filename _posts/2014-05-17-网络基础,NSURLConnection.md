---
layout: post
title: "网络基础,NSURLConnection"
date: 2014-05-17
comments: false
categories: 实用技术
---

##基本概念
- 在网络编程中，有几个必须掌握的基本概念 
    - 客户端（Client）：移动应用（iOS、android等应用） 
    - 服务器（Server）：为客户端提供服务、提供数据、提供资源的机器 
    - 请求（Request）：客户端向服务器索取数据的一种行为 
    - 响应（Response）：服务器对客户端的请求做出的反应，一般指返回数据给客户端

##URL
- 什么是URL 
    - URL的全称是Uniform Resource Locator（统一资源定位符） 
    - 通过1个URL，能找到互联网上唯一的1个资源 
    - URL就是资源的地址、位置，互联网上的每个资源都有一个唯一的URL 
    - URL的基本格式 = 协议://主机地址/路径
    - 例:http://www.baidu.com/zhunjiee/com.jpg

##URL中常见协议
- HTTP
	- 超文本传输协议，访问的是远程的网络资源，格式是http://
	- http协议是在网络开发中最常用的协议

- file
	- 访问的是本地计算机上的资源，格式是file://（不用加主机地址）

- mailto
	- 访问的是电子邮件地址，格式是mailto:

- FTP
	- 访问的是共享主机的文件资源，格式是ftp://

##GET和POST的选择

选择GET和POST的建议
- 如果要传递大量数据，比如文件上传，只能用POST请求
- GET的安全性比POST要差些，如果包含机密\敏感信息，建议用POST
- 如果仅仅是索取数据（数据查询），建议使用GET
- 如果是增加、修改、删除数据，建议使用POST

##iOS中发送HTTP请求的方案
- 苹果原生
    + NSURLConnection:过时
    + NSURLSession:2013年推出,iOS7开始
- 第三方框架:`AFNetworking`

##NSURLConnection
- 使用NSURLConnection发送请求的简单步骤:
    - 创建一个NSURL对象，设置请求路径
    - 传入NSURL创建一个NSURLRequest对象，设置请求头和请求体
    - 使用NSURLConnection发送请求


- **NSURLRequest**
    + 用于保存请求地址/请求头/请求体
    + 默认情况下NSURLRequest会自动给我们设置好请求头
    + request默认情况下就是GET请求

---  

- 同步请求(sendSynchronousRequest)
    + 如果是调用NSURLConnection的同步方法, 会阻塞当前线程

```objc
    // 1.创建一个URL
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login?username=520it&pwd=520it&type=JSON"];

    // 2.根据URL创建NSURLRequest对象
    // 默认情况下NSURLRequest会自动设置好请求头
    // request默认情况下就是GET请求
    NSURLRequest *request = [NSURLRequest requestWithURL:url];

    // 3.利用NSURLConnection对象发送请求
    /*
     第一个参数: 需要请求的对象
     第二个参数: 服务返回给我们的响应头信息
     第三个参数: 错误信息
     返回值: 服务器返回给我们的响应体
     */
    NSHTTPURLResponse *response = nil; // 真实类型
    // 如果是调用NSURLConnection的同步方法, 会阻塞当前线程
    NSData *data = [NSURLConnection sendSynchronousRequest:request returningResponse:&response error:nil];
    NSLog(@"%@", [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding]);
    NSLog(@"response = %@", response.allHeaderFields);
```

- 异步请求(sendAsynchronousRequest)

```objc
    // 1.创建一个URL
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login?username=520it&pwd=520it&type=JSON"];

    // 2.根据URL创建NSURLRequest对象
    NSURLRequest *request = [NSURLRequest requestWithURL:url];

    // 3.利用NSURLConnection对象发送请求
    /*
     第一个参数: 需要请求的对象
     第二个参数: 回调block的队列, 决定了block在哪个线程中执行
     第三个参数: 回调block
     */
    // 注意点: 如果是调用NSURLConnection的同步方法, 会阻塞当前线程
    [NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse *response, NSData *data, NSError *connectionError) {
    // respnse : 响应头
    // data : 响应体
    // connectionError : 错误信息
        NSLog(@"%@", [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding]);
    }];
```

- POST方法(NSMutableURLRequest)

```objc
    // 1.创建一个URL
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login"];

    // 2.根据URL创建NSURLRequest对象
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    // 2.1设置请求方式
    // 注意: POST一定要大写
    request.HTTPMethod = @"POST";
    // 2.2设置请求体
    // 注意: 如果是给POST请求传递参数: 那么不需要写?号
    request.HTTPBody = [@"username=520it&pwd=520it&type=JSON" dataUsingEncoding:NSUTF8StringEncoding];

    // 3.利用NSURLConnection对象发送请求
    [NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse *response, NSData *data, NSError *connectionError) {
        NSLog(@"%@", [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding]);
    }];
```

- 请求服务器响应(alloc/initWithRequest)

```objc
    // 1.创建URL
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/resources/images/minion_02.png"];

    // 2.根据URL创建NSURLRequest
    NSURLRequest *request = [NSURLRequest requestWithURL:url];

    // 3.利用NSURLConnection发送请求
    /*
    // 只要调用alloc/initWithRequest, 系统会自动发送请求
    [[NSURLConnection alloc] initWithRequest:request delegate:self];
    */

    /*
    // startImmediately: 如果传递YES, 系统会自动发送请求; 如果传递NO, 系统不会自动发送请求
    NSURLConnection *conn = [[NSURLConnection alloc] initWithRequest:request delegate:self startImmediately:NO];
    [conn start];
    */
    // 可用此方法代替
    [NSURLConnection connectionWithRequest:request delegate:self];
```
+ 代理方法

```objc
#pragma mark - NSURLConnectionDataDelegate
/*
 只要接收到服务器的响应就会调用
 response:响应头
 */
- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response
{
    // 获取服务器返回给我们的文件的总大小
    self.totalLength = response.expectedContentLength;
    
}

/*
  接收到服务器返回的数据时调用(该方法可能调用一次或多次)
  data: 服务器返回的数据(当前这一次传递给我们的, 并不是总数)
 */
- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data
{
    // 计算进度
    self.currentLength += data.length;
    
    NSLog(@"当前进度为:%f", 1.0 * self.currentLength / self.totalLength);
}
/*
 接收结束时调用
 */
- (void)connectionDidFinishLoading:(NSURLConnection *)connection
{
    NSLog(@"%s", __func__);
}

/*
 请求错误时调用(请求超时)
 */
- (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error
{
    NSLog(@"%s", __func__);
}
```
##中文问题
+ 如果是POST请求,参数中可以有中文,因为POST的数据是放在请求体,请求体接收的是二进制
+ 如果是GET请求,参数中不可以有中文
+ 开发时无论是POST还是GET,最好先用`stringByAddingPercentEscapesUsingEncoding`转化一下

```objc
// 1.创建URL
    NSString *urlStr = @"http://120.25.226.186:32812/login2?username=小码哥&pwd=520it&type=JSON";
    NSLog(@"转换前:%@", urlStr);
    // 2.对URL进行转码
    urlStr = [urlStr stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
```

###登录练习拼接字符串
```objc
// 利用NSURLConnection发送请求
    [NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse *response, NSData *data, NSError *connectionError) {
        [MBProgressHUD hideHUD];
        
        // NSData 转 NSString
        NSString *responseObj = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
        // 截取字符串
        // 注意:在企业开发中千万不要给用户详细错误提示信息
        NSUInteger start = [responseObj rangeOfString:@":\""].location + 2;
        NSUInteger length = [responseObj rangeOfString:@"\"" options:NSBackwardsSearch].location - start;
        NSRange range = NSMakeRange(start, length);
        NSString *res = [responseObj substringWithRange:range];
        
        if ([responseObj containsString:@"error"]) {
            [MBProgressHUD showError:res];
        }else{
            [MBProgressHUD showMessage:res];
        }
//        
//        if ([responseObj containsString:@"error"]) {
//            [MBProgressHUD showError:@"用户名或密码错误"];
//        }else{
//            [MBProgressHUD showMessage:@"登录成功"];
//        }
    }];
```
---

##NSURLConnection和RunLoop
- 默认情况会将NSURLConnection添加当前线程到RunLoop,如果是在子线程中调用NSURLConnection可能会有问题, 因为子线程默认没有RunLoop

- 如何让代理方法在子线程中执行?
    + [conn setDelegateQueue:[[NSOperationQueue alloc] init]];

- 注意:
    + NSURLConnection是异步请求
```objc
        // 1.发送请求
        NSURLRequest *request = [NSURLRequest requestWithURL:[NSURL URLWithString:@"http://120.25.226.186:32812/resources/images/minion_01.png"]] ;
       NSRunLoop *runloop = [NSRunLoop currentRunloop];
       NSURLConnection *conn = [NSURLConnection connectionWithRequest:request delegate:self];
        [conn setDelegateQueue:[[NSOperationQueue alloc] init]];
        // 2.启动runLoop
        // 默认情况下子线程没有RunLoop, 所以需要启动
        [runloop run];

        // 3.如果手动发送请求, 系统内部会自动帮子线程创建一个RunLoop
    //    NSURLConnection *conn = [[NSURLConnection alloc] initWithRequest:request delegate:self startImmediately:YES];
    //    [conn start];
```

---
