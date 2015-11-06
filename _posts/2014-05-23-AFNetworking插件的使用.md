---
layout: post
title: "AFNetworking插件的使用"
date: 2014-05-23
comments: false
categories: 实用技术
---

##NSURLConnection包装方法
###GET
```objc
    // 1.创建AFN管理者
    // AFHTTPRequestOperationManager内部包装了NSURLConnection
    AFHTTPRequestOperationManager *manager = [AFHTTPRequestOperationManager manager];

    // 2.利用AFN管理者发送请求
    NSDictionary *params = @{
                             @"username" : @"520it",
                             @"pwd" : @"520it"
                             };
    [manager GET:@"http://120.25.226.186:32812/login" parameters:params success:^(AFHTTPRequestOperation *operation, id responseObject) {
    // 注意点: 如果服务器返回的是JSON, AFN会自动转换为OC对象
         NSLog(@"请求成功---%@", responseObject);
    } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
         NSLog(@"请求失败---%@", error);
    }];
```
###POST
```objc
    // 1.创建AFN管理者
    // AFHTTPRequestOperationManager内部包装了NSURLConnection
    AFHTTPRequestOperationManager *manager = [AFHTTPRequestOperationManager manager];

    // 2.利用AFN管理者发送请求
    NSDictionary *params = @{
                             @"username" : @"520it",
                             @"pwd" : @"520it"
                             };
    [manager POST:@"http://120.25.226.186:32812/login" parameters:params success:^(AFHTTPRequestOperation *operation, id responseObject) {
        NSLog(@"请求成功---%@", responseObject);
    } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
        NSLog(@"请求失败---%@", error);
    }];
    ```
##NSURLSession包装方法
###GET
```objc
    // 1.创建AFN管理者
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];

    // 2.利用AFN管理者发送请求
    NSDictionary *params = @{
                             @"username" : @"520it",
                             @"pwd" : @"520it"
                             };

    [manager GET:@"http://120.25.226.186:32812/login" parameters:params success:^(NSURLSessionDataTask *task, id responseObject) {
         NSLog(@"请求成功---%@", responseObject);
    } failure:^(NSURLSessionDataTask *task, NSError *error) {
        NSLog(@"请求失败---%@", error);
    }];
```
### POST
```objc
    // 1.创建AFN管理者
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];

    // 2.利用AFN管理者发送请求
    NSDictionary *params = @{
                             @"username" : @"520it",
                             @"pwd" : @"520it"
                             };

    [manager POST:@"http://120.25.226.186:32812/login" parameters:params success:^(NSURLSessionDataTask *task, id responseObject) {
        NSLog(@"请求成功---%@", responseObject);
    } failure:^(NSURLSessionDataTask *task, NSError *error) {
        NSLog(@"请求失败---%@", error);
    }];
```

##文件下载

```objc
    // 1.创建AFN管理者
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];

    // 2.利用AFN管理者发送请求
     NSURLRequest *reuqest = [NSURLRequest requestWithURL:[NSURL URLWithString:@"http://120.25.226.186:32812/resources/videos/minion_02.mp4"]];

    [[manager downloadTaskWithRequest:reuqest progress:nil  destination:^NSURL *(NSURL *targetPath, NSURLResponse *response) {
        // targetPath: 已经下载好的文件路径
        NSLog(@"targetPath = %@", targetPath);
        NSString *path = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) firstObject];
        NSURL *documentsDirectoryPath = [NSURL fileURLWithPath:[path stringByAppendingPathComponent:response.suggestedFilename]];
        // 返回需要保存文件的目标路径
        return documentsDirectoryPath;
    } completionHandler:^(NSURLResponse *response, NSURL *filePath, NSError *error) {
        NSLog(@"filePath = %@", filePath);
    }] resume];

```

###监听进度

```objc
要跟踪进度，需要使用 NSProgress，是在 iOS 7.0 推出的，专门用来跟踪进度的类！

// 下载时监听进度
- (void)download2{
    // 1. 创建AFNetworking管理者
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
    
    // 2. 利用AFNetworking管理者发送请求
    NSURLRequest *request = [NSURLRequest requestWithURL:[NSURL URLWithString:@"http://120.25.226.186:32812/resources/videos/minion_02.mp4"]];
    
    // 3.创建NSProgress
    NSProgress *progress = nil;
    self.progress = progress;
    
    // 4. 创建下载任务
    NSURLSessionDownloadTask *task = [manager downloadTaskWithRequest:request progress:&progress destination:^NSURL *(NSURL *targetPath, NSURLResponse *response) {
        // 文件移动到caches目录
        NSString *path = [response.suggestedFilename cacheDir];
        // 获取文件的URL地址
        NSURL *destURL = [NSURL fileURLWithPath:path];
        // 返回URL地址
        return destURL;
    } completionHandler:^(NSURLResponse *response, NSURL *filePath, NSError *error) {
        NSLog(@"filePath = %@", filePath);
    }];
    
    // 5. 给NSProgress注册监听, 监听它的completedUnitCount属性的改变
    /*
     NSProsess属性:
        @property int64_t totalUnitCount; 总大小
        @property int64_t completedUnitCount; 完成大小
     */
    [progress addObserver:self forKeyPath:@"completedUnitCount" options:NSKeyValueObservingOptionNew context:nil];
    
    [task resume];
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context{
    if ([object isKindOfClass:[NSProgress class]]) {
        NSProgress *progress = (NSProgress *)object;
        
        // 计算下载进度
        NSLog(@"%f", 1.0 * progress.completedUnitCount / progress.totalUnitCount);
    }
}

- (void)dealloc{
    [self removeObserver:self.progress forKeyPath:@"completedUnitCount"];
}
```

##文件上传

```objc
// 1.创建AFN管理者
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];

    // 2.利用AFN管理者发送请求
    [manager POST:@"http://120.25.226.186:32812/upload" parameters:@{@"username" : @"lnj"} constructingBodyWithBlock:^(id<AFMultipartFormData> formData) {
        NSData *data = [NSData dataWithContentsOfFile:@"/Users/NJ-Lee/Desktop/Snip20150811_1.png"];
    // name : 服务器对应的数据参数    
    [formData appendPartWithFileData:data name:@"file" fileName:@"lnj.png" mimeType:@"image/png"];
    } success:^(NSURLSessionDataTask *task, id responseObject) {
          NSLog(@"请求成功---%@", responseObject);
    } failure:^(NSURLSessionDataTask *task, NSError *error) {
        NSLog(@"请求失败---%@", error);
    }];
```

```objc
[formData appendPartWithFileURL:[NSURL fileURLWithPath:@"/Users/NJ-Lee/Desktop/Snip20150811_1.png"] name:@"file" fileName:@"lnj.png" mimeType:@"image/png" error:nil];
```
最好用

```objc
[formData appendPartWithFileURL:[NSURL fileURLWithPath:@"/Users/NJ-Lee/Desktop/Snip20150811_1.png"] name:@"file" error:nil];
```

- AFN解耦
    + 自定义单利类继承Manager
    + 优点: 替换框架只需要求改单利类即可

- 序列化
    + AFN默认将服务器返回的数据当做JSON处理, 会自动解析
        * manager.responseSerializer = [AFJSONRequestSerializer serializer];
    + 告诉AFN，以XML形式解析服务器返回的数据
        * manager.responseSerializer = [AFXMLParserResponseSerializer serializer];
    + 告诉AFN, 不处理服务器返回的数据, 原样返回
        * manager.responseSerializer = [AFHTTPResponseSerializer serializer];
- 主要用于存储对象状态为另一种通用格式，比如存储为二进制、xml、json等等，把对象转换成这种格式就叫序列化，而反序列化通常是从这种格式转换回来。

- 使用序列化主要是因为跨平台和对象存储的需求，因为网络上只允许字符串或者二进制格式，而文件需要使用二进制流格式，如果想把一个内存中的对象存储下来就必须使用序列化转换为xml（字符串）、json（字符串）或二进制（流） 

##网络状态检测
- AFN

```objc
    // 1.创建网络监听对象
    AFNetworkReachabilityManager *manager = [AFNetworkReachabilityManager sharedManager];
    // 2.设置网络状态改变回调
    [manager setReachabilityStatusChangeBlock:^(AFNetworkReachabilityStatus status) {
        /*
         AFNetworkReachabilityStatusUnknown          = -1,  // 未知
         AFNetworkReachabilityStatusNotReachable     = 0,   // 无连接
         AFNetworkReachabilityStatusReachableViaWWAN = 1,   // 3G 花钱
         AFNetworkReachabilityStatusReachableViaWiFi = 2,   // 局域网络,不花钱
         */
        switch (status) {
            case 0:
                NSLog(@"无连接");
                break;
            case 1:
                NSLog(@"3G 花钱");
                break;
            case 2:
                NSLog(@"局域网络,不花钱");
                break;
            default:
                NSLog(@"未知");
                break;
        }
    }];

    // 3.开始监听
    [manager startMonitoring];
```
##苹果自带

```objc
    - (void)viewDidLoad {
    [super viewDidLoad];

    // 注册通知
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(getNetworkStatus) name:kReachabilityChangedNotification object:nil];

    // 开始监听网络
    self.reachability = [Reachability reachabilityForInternetConnection];
    [self.reachability startNotifier];
}

- (void)dealloc
{
    [[NSNotificationCenter defaultCenter] removeObserver:self];
    [self.reachability stopNotifier];
}
- (void)getNetworkStatus{
    if ([Reachability reachabilityForLocalWiFi].currentReachabilityStatus != NotReachable) {
        NSLog(@"wifi");
    }else if ([Reachability reachabilityForInternetConnection].currentReachabilityStatus != NotReachable)
    {
        NSLog(@"手机自带网络");
    }else
    {
        NSLog(@"没有网络");
    }
}
```

