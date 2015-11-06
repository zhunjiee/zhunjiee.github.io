---
layout: post
title: "https,UIWebView,OC和JS相互调用"
date: 2014-05-24
comments: false
categories: 实用技术
---

#https
- **概念**:
    ﻿+ HTTPS（全称：Hyper Text Transfer Protocol over Secure Socket Layer），是以安全为目标的HTTP通道，简单讲是HTTP的安全版。即HTTP下加入SSL层，HTTPS的安全基础是SSL，因此加密的详细内容就需要SSL。

- **主要作用**：
    ﻿+ 一种是建立一个信息安全通道，来保证数据传输的安全；
    ﻿+ 另一种就是确认网站的真实性，凡是使用了 https 的网站，都可以通过点击浏览器地址栏的锁头标志来查看网站认证之后的真实信息，也可以通过 CA 机构颁发的安全签章来查询

- **实现原理**:
    ﻿+ 1. 客户端向服务器发送一个开始信息“Hello”以便开始一个新的会话连接；
    ﻿+ 2. 服务器根据客户的信息确定是否需要生成新的主密钥，如需要则服务器在响应客户的“Hello”信息时将包含生成主密钥所需的信息；
    ﻿+ 3. 客户根据收到的服务器响应信息，产生一个主密钥，并用服务器的公开密钥加密后传给服务器；
    ﻿+ 4. 服务器恢复该主密钥，并返回给客户一个用主密钥认证的信息，以此让客户认证服务器。

实现代码(了解)

```objc
#pragma mark - NSURLSessionDataDelegate
// 只要访问的是HTTPS的路径就会调用
// 该方法的作用是处理服务器返回的证书,需要在方法中高速系统是否需要安装服务器返回的证书
- (void)URLSession:(NSURLSession *)session didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition, NSURLCredential *))completionHandler{
    // NSURLAuthenticationChallenge : 授权质问 -(里面有)-> 受保护空间 -(里面有)-> 服务器返回的证书类型
    // NSURLSessionAuthChallengeDisposition : 如何处理证书
    // NSURLCredential : 证书是谁
    
    /*证书安装步骤:
     1. 从服务器返回的受保护控件中拿到证书的类型 challenge.protectionSpace.authenticationMethod
     2. 判断服务器返回的证书是否是服务器信任的 NSURLAuthenticationMethodServerTrust
     3. 根据服务器返回的受保护控件创建一个证书
     4. 安装证书
     */
    
    NSURLSessionAuthChallengeDisposition disposition = NSURLSessionAuthChallengePerformDefaultHandling;
    __block NSURLCredential *credential = nil;
    
    
    if ([challenge.protectionSpace.authenticationMethod isEqualToString:NSURLAuthenticationMethodServerTrust]) {
        
        credential = [NSURLCredential credentialForTrust:challenge.protectionSpace.serverTrust];
        if (credential) {
            disposition = NSURLSessionAuthChallengeUseCredential;
        } else {
            disposition = NSURLSessionAuthChallengePerformDefaultHandling;
        }
    } else {
        disposition = NSURLSessionAuthChallengeCancelAuthenticationChallenge;
    }
    
    if (completionHandler) {
        completionHandler(disposition, credential);
    }
}
```

AFNetworking实现

```objc
NSURL * url = [NSURL URLWithString:@"https://www.google.com"];
AFHTTPRequestOperationManager * requestOperationManager = [[AFHTTPRequestOperationManager alloc] initWithBaseURL:url];
dispatch_queue_t requestQueue = dispatch_create_serial_queue_for_name("kRequestCompletionQueue");
requestOperationManager.completionQueue = requestQueue;
AFSecurityPolicy * securityPolicy = [AFSecurityPolicy policyWithPinningMode:AFSSLPinningModeCertificate];
//allowInvalidCertificates 是否允许无效证书（也就是自建的证书），默认为NO
//如果是需要验证自建证书，需要设置为YES
securityPolicy.allowInvalidCertificates = YES;
//validatesDomainName 是否需要验证域名，默认为YES；
//假如证书的域名与你请求的域名不一致，需把该项设置为NO
//主要用于这种情况：客户端请求的是子域名，而证书上的是另外一个域名。因为SSL证书上的域名是独立的，假如证书上注册的域名是www.google.com，那么mail.google.com是无法验证通过的；当然，有钱可以注册通配符的域名*.google.com，但这个还是比较贵的。
securityPolicy.validatesDomainName = NO;
//validatesCertificateChain 是否验证整个证书链，默认为YES
//设置为YES，会将服务器返回的Trust Object上的证书链与本地导入的证书进行对比，这就意味着，假如你的证书链是这样的：
//GeoTrust Global CA 
//    Google Internet Authority G2
//        *.google.com
//那么，除了导入*.google.com之外，还需要导入证书链上所有的CA证书（GeoTrust Global CA, Google Internet Authority G2）；
//如是自建证书的时候，可以设置为YES，增强安全性；假如是信任的CA所签发的证书，则建议关闭该验证；
securityPolicy.validatesCertificateChain = NO;
requestOperationManager.securityPolicy = securityPolicy;
```

##UIWebView

##常用属性和方法
```objc
重新加载（刷新）
- (void)reload;
停止加载
- (void)stopLoading;
回退
- (void)goBack;
前进
- (void)goForward;
需要进行检测的数据类型
@property(nonatomic) UIDataDetectorTypes dataDetectorTypes
```
##常用属性和方法
```objc
是否能回退
@property(nonatomic,readonly,getter=canGoBack) BOOL canGoBack;
是否能前进
@property(nonatomic,readonly,getter=canGoForward) BOOL canGoForward;
是否正在加载中
@property(nonatomic,readonly,getter=isLoading) BOOL loading;
是否伸缩内容至适应屏幕当前尺寸
@property(nonatomic) BOOL scalesPageToFit;
```
##监听UIWebView的加载过程
```objc
成为UIWebView的代理，遵守UIWebViewDelegate协议，就能监听UIWebView的加载过程
开始发送请求（加载数据）时调用这个方法
- (void)webViewDidStartLoad:(UIWebView *)webView;
请求完毕（加载数据完毕）时调用这个方法
- (void)webViewDidFinishLoad:(UIWebView *)webView;
请求错误时调用这个方法
- (void)webView:(UIWebView *)webView didFailLoadWithError:(NSError *)error;
```
##监听UIWebView的加载过程
```objc
UIWebView在发送请求之前，都会调用这个方法，如果返回NO，代表停止加载请求，返回YES，代表允许加载请求
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType;
```


##OC中调用JS
```objc
[self.webView stringByEvaluatingJavaScriptFromString:@"test()"];
```
##JS中调用OC
- 原理:
    ﻿- 利用该方法作为JS和OC之间的桥梁
    ﻿- 在JS跳转页面
    ﻿- 在OC代理方法中通过判断自定义协议头,决定是否是JS调用OC方法
    ﻿- 在OC代理方法中通过截取字符串,获取JS想调用的OC方法名称


###JS中调用OC(无参)
html

```
    ﻿       // ------- JS 调用 OC --------
            function test2 () {
                // 通过重定向
                location.href = "hbw://call";
            }
```
oc

```objc
// 每次发送请求前都会调用
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType{
    NSString *schem = @"hbw://";
    NSString *urlStr = request.URL.absoluteString;
    if ([urlStr hasPrefix:schem]) {
        NSLog(@"需要调用OC方法");
        // 1. 从URL中获取方法名称
        NSString *methodName = [urlStr substringFromIndex:schem.length];
        
        // 2. 调用方法
        SEL sel = NSSelectorFromString(methodName);
        [self performSelector:sel withObject:nil];
        
       
        return NO; // 执行OC方法,不去加载网页了

    }
    return YES;
}


- (void)call
{
    NSLog(@"%s", __func__);
}


```
###JS中调用OC(有参)
html
```
// ------- JS 调用 OC --------
            function test2 () {
                // 通过重定向
                location.href = "hbw://callWithNumber_?10086";
            }
```
OC
```objc
// 每次发送请求前都会调用
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType{
    NSString *schem = @"hbw://";
    NSString *urlStr = request.URL.absoluteString;
    if ([urlStr hasPrefix:schem]) {
        NSLog(@"需要调用OC方法");
        // 1. 从URL中获取方法名称
        // hbw://sendMessageWithnumber_andContent_?10086&hahaha
        NSString *subPath = [urlStr substringFromIndex:schem.length];
        
        NSArray *subPaths = [subPath componentsSeparatedByString:@"?"];
        
        // 2. 拿到方法名字符串
        NSString *methodName = [subPaths firstObject];
        methodName = [methodName stringByReplacingOccurrencesOfString:@"_" withString:@":"];
        
        // 3. 调用方法
        SEL sel = NSSelectorFromString(methodName);
        NSString *prama = nil;
        if (subPaths.count == 2) {
            prama = [subPaths lastObject];
            // 截取参数
            NSArray *pramas = [prama componentsSeparatedByString:@"&"];
            // 两个参数的方法
            [self performSelector:sel withObject:[pramas firstObject] withObject:[pramas lastObject]];
            return NO;
        }
        // 一个参数的方法
        [self performSelector:sel withObject:prama];
        
        return NO; // 执行OC方法,不去加载网页了
    }
    return YES;
}

- (void)callWithNumber:(NSString *)number
{
    NSLog(@"打电话给%@", number);
}


- (void)sendMessageWithNumber:(NSString *)number andContent:(NSString *)content{
    NSLog(@"发送信息给%@, 内容是:%@", number, content);
}

```
