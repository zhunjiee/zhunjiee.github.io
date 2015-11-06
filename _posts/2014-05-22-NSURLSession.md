---
layout: post
title: "NSURLSession"
date: 2014-05-22
comments: false
categories: 实用技术
---

- 使用步骤
    + 创建NSURLSession
    + 创建Task
    + 执行Task

##NSURLSessionDataTask
- GET

```objc
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login?username=520it&pwd=520it"];
    NSURLRequest *request = [NSURLRequest requestWithURL:url];

    // 1.获得NSURLSession对象
    NSURLSession *session = [NSURLSession sharedSession];

    // 2.创建任务
    NSURLSessionDataTask *task = [session dataTaskWithRequest:request completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
        NSLog(@"%@", [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding]);
    }];

    // 3.启动任务
    [task resume];
```
如果是发送GET请求, 或者不需要设置请求头的信息,那么建议用下面的方法

```objc
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login?username=520it&pwd=520it"];
    // 1.获得NSURLSession对象
    NSURLSession *session = [NSURLSession sharedSession];

    // 2.创建任务
    NSURLSessionDataTask *task = [session dataTaskWithURL:url completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
         NSLog(@"%@", [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding]);
    }];
    // 3.启动任务
    [task resume];
```
- POST

```objc
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login"];
    // 设置请求头
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    request.HTTPMethod = @"POST";
    request.HTTPBody = [@"username=520it&pwd=520it" dataUsingEncoding:NSUTF8StringEncoding];

    // 1.获得NSURLSession对象
    NSURLSession *session = [NSURLSession sharedSession];

    // 2.创建任务
    NSURLSessionDataTask *task = [session dataTaskWithRequest:request completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
        NSLog(@"%@", [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding]);
    }];
    // 3.启动任务
    [task resume];
```

- NSURLSessionDataDelegate代理注意事项
    + Task都是通过Session监听
```ojbc
 // 1.获得NSURLSession对象
    NSURLSession *session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:self delegateQueue:[[NSOperationQueue alloc] init]];
```
    + NSURLSessionDataDelegate`默认不会接受数据`

```objc
/**
 * 1.接收到服务器的响应
 */
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveResponse:(NSURLResponse *)response completionHandler:(void (^)(NSURLSessionResponseDisposition))completionHandler
{
    NSLog(@"%s", __func__);
    NSLog(@"%@", [NSThread currentThread]);
    /*
     NSURLSessionResponseAllow: 去向处理服务器响应, 内部会调用[task cancle]
     NSURLSessionResponseCancel: 允许处理服务器的响应，才会继续接收服务器返回的数据
     NSURLSessionResponseBecomeDownload: 讲解当前请求变为下载
     */
    completionHandler(NSURLSessionResponseAllow);
}
/**
 * 2.接收到服务器的数据（可能会被调用多次）
 */
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveData:(NSData *)data
{
    NSLog(@"%s", __func__);
}
/**
 * 3.请求成功或者失败（如果失败，error有值）
 */
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error
{
    NSLog(@"%s", __func__);
}
```
---

##NSURLSessionDownloadTask
- DownloadTask默认已经实现边下载边写入
```objc
    // 1.创建request
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/resources/videos/minion_01.mp4"];
    NSURLRequest *request = [NSURLRequest requestWithURL:url];

    // 2.创建session
    NSURLSession *session = [NSURLSession sharedSession];

    // 3.创建下载任务
    NSURLSessionDownloadTask *task = [session downloadTaskWithRequest:request completionHandler:^(NSURL *location, NSURLResponse *response, NSError *error) {
        NSLog(@"%@", location.absoluteString);

        NSFileManager *manager = [NSFileManager defaultManager];
        NSString *toPath = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
        toPath = [toPath stringByAppendingPathComponent:response.suggestedFilename];
        // 将下载好的文件从location移动到cache
        [manager moveItemAtURL:location toURL:[NSURL fileURLWithPath:toPath] error:nil];
    }];

    // 4.开启下载任务
    [task resume];
```

- DownloadTask断点下载
    + 暂停: suspend
    + 继续: resume
    + 取消: cancel
        * 任务一旦被取消就无法恢复
    + 区别并返回下载信息
```objc
    //取消并获取当前下载信息
[self.task cancelByProducingResumeData:^(NSData *resumeData) {
        self.resumeData = resumeData;
    }];

    // 根据用户上次的下载信息重新创建任务
    self.task = [self.session downloadTaskWithResumeData:self.resumeData];
    [self.task resume];
```

- DownloadTask监听下载进度

```objc
/**
 * 每当写入数据到临时文件时，就会调用一次这个方法
 * totalBytesExpectedToWrite:总大小
 * totalBytesWritten: 已经写入的大小
 * bytesWritten: 这次写入多少
 */
- (void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didWriteData:(int64_t)bytesWritten totalBytesWritten:(int64_t)totalBytesWritten totalBytesExpectedToWrite:(int64_t)totalBytesExpectedToWrite
{
    NSLog(@"正在下载: %.2f", 1.0 * totalBytesWritten / totalBytesExpectedToWrite);
}

/*
 * 根据resumeData恢复任务时调用
 * expectedTotalBytes:总大小
 * fileOffset: 已经写入的大小
 */
- (void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didResumeAtOffset:(int64_t)fileOffset expectedTotalBytes:(int64_t)expectedTotalBytes
{
    NSLog(@"didResumeAtOffset fileOffset = %lld , expectedTotalBytes = %lld", fileOffset, expectedTotalBytes);
}
/**
 * 下载完毕就会调用一次这个方法
 */
- (void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didFinishDownloadingToURL:(NSURL *)location
{
    NSLog(@"下载完毕");
    // 文件将来存放的真实路径
    NSString *file = [[NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject] stringByAppendingPathComponent:downloadTask.response.suggestedFilename];

    // 剪切location的临时文件到真实路径
    NSFileManager *mgr = [NSFileManager defaultManager];
    [mgr moveItemAtURL:location toURL:[NSURL fileURLWithPath:file] error:nil];
}
/**
 * 任务完成
 */
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error
{
    NSLog(@"didCompleteWithError");
}

```
---

##离线断点下载
- 使用NSURLSessionDataTask

```objc
NSMutableURLRequest *reuqest = [NSMutableURLRequest requestWithURL:[NSURL URLWithString:@"http://120.25.226.186:32812/resources/videos/minion_01.mp4"]];

- 核心方法, 实现从指定位置开始下载
NSInteger currentBytes = KCurrentBytes;
NSString *range = [NSString stringWithFormat:@"bytes=%zd-", currentBytes];
[reuqest setValue:range forHTTPHeaderField:@"Range"];

- 核心方法, 自己实现边下载边写入
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveData:(NSData *)data;
```
---
###断点下载示例:
- 获取当前文件的下载进度

```objc
- (NSUInteger)getFileSizeWithPath:(NSString *)path
{
    NSUInteger currentSize = [[[NSFileManager defaultManager] attributesOfItemAtPath:path error:nil][NSFileSize] integerValue];
    return currentSize;
}
```
- 懒加载创建session,task

```objc
#pragma mark - lazy
- (NSURLSession *)session{
    if (!_session) {
        // 创建session
        _session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:self delegateQueue:[NSOperationQueue mainQueue]];
        
    }
    return _session;
}
```
- 从本地获得文件已经下载的大小
- 设置请求头,@"Range"为固定
- 创建任务

```objc
- (NSURLSessionDataTask *)task{
    if (!_task) {
        NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/resources/videos/minion_02.mp4"];
        NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
        
        // 从本地文件中获取已经下载的大小
        NSString *range = [NSString stringWithFormat:@"bytes:%zd-", [self getFileSizeWithPath:self.path]];
        // 设置请求头,@"Range"为固定
        [request setValue:range forHTTPHeaderField:@"Range"];
        
        // 创建任务
        _task = [self.session dataTaskWithRequest:request];
    }
    return _task;
}
```
- 初始化文件存放路径及下载进度

```objc
- (void)viewDidLoad{
    // 初始化操作
    // 1. 初始化文件路径
    self.path = [@"minion_02.mp4" cacheDir];
    // 2. 初始化当前下载进度
    self.currentLength = [self getFileSizeWithPath:self.path];
    
}

```

- 内部控制方法

```objc
#pragma mark - 内部控制方法
// 开始
- (IBAction)clickStart:(UIButton *)sender {

    // 开始任务
    [self.task resume];
}
// 暂停
- (IBAction)clickPause:(UIButton *)sender {
    NSLog(@"暂停任务");
    [self.task suspend];
}
// 继续
- (IBAction)clickGoOn:(UIButton *)sender {
    NSLog(@"任务继续");
    [self.task resume];
}
// 取消任务
- (IBAction)clickCancle:(UIButton *)sender {
    // 任务一旦取消,一般就无法再继续了
    [self.task cancel];
}

```

- didReceiveResponse代理方法中:
    - 手动设置接收数据
    - 初始化文件总大小
    - 打开输出流
  
- didReceiveData代理方法中:
    - 累加已经下载的大小
    - 计算进度
    - 写入数据
    
- didCompleteWithError代理方法中
    - 关闭输出流

```objc
#pragma mark - NSURLSessionDataDelegate
// 接收到服务器的响应时调用
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveResponse:(NSURLResponse *)response completionHandler:(void (^)(NSURLSessionResponseDisposition))completionHandler{
    NSLog(@"didReceiveResponse");
    // 注意: 在NSURLSessionDataTask的代理方法中, 默认情况下是不接受服务器返回的数据的, 所以didReceiveData方法和didCompleteWithError默认不会被调用
    
    // 如果想接收服务器返回的数据, 必须手动的告诉系统, 我们需要接收数据
    completionHandler(NSURLSessionResponseAllow);

    // 初始化文件总大小
    self.totalLength = response.expectedContentLength + [self getFileSizeWithPath:self.path];
    
    // 打开输出流
    self.outPutStream = [NSOutputStream outputStreamToFileAtPath:self.path append:YES];
    [self.outPutStream open];
}

// 接收到服务器返回的数据时调用
// data 此次接收到的数据
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveData:(NSData *)data{
    NSLog(@"didReceiveData");
    
    // 累加已经下载的大小
    self.currentLength += data.length;
    
    // 计算进度
    self.progressView.progress = 1.0 * self.currentLength / self.totalLength;
    
    // 写入数据
    [self.outPutStream write:data.bytes maxLength:data.length];
    
}

// 请求完成时调用
// 如果调用该方法时error有值, 代表着下载出现错误
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error{
    NSLog(@"didCompleteWithError");
    
    // 关闭输出流
    [self.outPutStream close];
}
```


---

##NSURLSessionUploadTask
- 设置设置请求头, 告诉服务器是文件上传
    + multipart/form-data; boundary=lnj
- 必须自己`严格按照格式`拼接请求体
- 请求体不能设置给request, 只能设置给fromData
    + The body stream and body data in this request object are ignored.

```objc
[[self.session uploadTaskWithRequest:request fromData:body completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
        NSLog(@"%@", [NSJSONSerialization JSONObjectWithData:data options:kNilOptions error:nil]);
    }] resume];
```

- 注意:以下方法用于PUT请求

```objc
[self.session uploadTaskWithRequest:<#(nonnull NSURLRequest *)#> fromFile:<#(nonnull NSURL *)#>]
```

- 监听上传进度

```objc
/*
 bytesSent: 当前发送的文件大小
 totalBytesSent: 已经发送的文件总大小
 totalBytesExpectedToSend: 需要发送的文件大小
 */
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didSendBodyData:(int64_t)bytesSent totalBytesSent:(int64_t)totalBytesSent totalBytesExpectedToSend:(int64_t)totalBytesExpectedToSend
{
    NSLog(@"%zd, %zd, %zd", bytesSent, totalBytesSent, totalBytesExpectedToSend);
}

- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error
{
    NSLog(@"%s", __func__);
}
```

---
