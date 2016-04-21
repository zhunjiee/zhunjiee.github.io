---
layout: post
title: "什么是Socket,XMPP？"
date: 2014-07-29
comments: false
categories: 知识囊
---
##什么是scoket?
- Socket又称"套接字”- **网络上的两端通过建立一个双向的通信连接实现数据的交换，这个端就称为一个socket**。- 应用程序通常通过"套接字"向网络发出请求或者应答网络请求
![](https://dn-zhunjiee.qbox.me/Snip20151220_1.png)

##网络通信的要素
网络上的请求就是通过Socket来建立连接然后互相通信http:192.168.0.1:80/login- **IP地址**（网络上主机设备的唯一标识）- **端口号**(定位程序)	- 用于标示进程的逻辑地址，不同进程的标示	- 有效端口：0~65535，其中0~1024由系统使用或者保留端口，开发中建议使用1024以上的端口- **传输协议**（用什么样的方式进行交互）	- 通讯的规则	- 常见协议：TCP、UDP##TCP&UDP
###TCP（传输控制协议）- **建立连接**，形成传输数据的通道- 在连接中进行**大数据传输**（数据大小不收限制）- 通过**三次握手(请求,回应，确认)**完成连接，是**可靠协议**，安全送达- 必须建立连接，**效率会稍低**>关于三次握手

>第一次握手：**客户端尝试连接服务器**，向服务器发送syn包（同步序列编号Synchronize Sequence Numbers），syn=j，客户端进入SYN_SEND状态等待服务器确认

>第二次握手：**服务器接收客户端syn包并确认**（ack=j+1），同时向客户端发送一个SYN包（syn=k），即SYN+ACK包，此时服务器进入SYN_RECV状态

>第三次握手：**客户端收到服务器的SYN+ACK包**，向服务器发送确认包ACK(ack=k+1），此包发送完毕，客户端和服务器进入ESTABLISHED状态，完成三次握手
###UDP（用户数据报协议）- 将数据及源和目的封装成数据包中，**不需要建立连接**- 因为无需连接，因此是**不可靠协议**- 每个数据报的大小限制在**64K之内**- 不需要建立连接，**速度快**
- 常用于**视频广播**等场景>UDP是一种无连接的传输层协议，它主要用于不要求分组顺序到达的传输中，分组传输顺序的检查与排序由应用层完成，提供面向事务的简单不可靠信息传送服务。UDP 协议基本上是IP协议与上层协议的接口。UDP协议适用端口分别运行在同一台设备上的多个应用程序。

>UDP提供了无连接通信，且不对传送数据包进行可靠性保证，适合于一次传输少量数据，UDP传输的可靠性由应用层负责。

>UDP报文没有可靠性保证、顺序保证和流量控制字段等，可靠性较差。但是正因为UDP协议的控制选项较少，在数据传输过程中延迟小、数据传输效率高，适合对可靠性要求不高的应用程序，或者可以保障可靠性的应用程序，如DNS、TFTP、SNMP等。##实现Socket服务端监听
- 实现Socket的监听方法有：	- 1.使用C语言实现	- 2.使用**CocoaAsyncSocket**第三方框，内部是对C的封装- Telnet命令 ***telnet 192.168.10.10 5288***	- telnet命令是连接 *服务器上的某个端口对应的服务*##Socket编程案例
###通过Socket编程编写 "群聊服务器" 与 "群聊客户端"
- <群聊服务器>
- 新建命令行项目
- 使用**CocoaAsyncSocket**第三方框,将**GCDAsyncSocket**的.h和.m文件复制到项目中


```
#import "ServerSocket.h"
#import "GCDAsyncSocket.h"

@interface ServerSocket ()<GCDAsyncSocketDelegate>
/** 服务端Socket */
@property (nonatomic, strong) GCDAsyncSocket *ServerSocket;
/** 存放客户端Socket的数组 */
@property (nonatomic, strong) NSMutableArray *clientSocketArray;
@end

@implementation ServerSocket
/** clientSocketArray的懒加载 */
- (NSMutableArray *)clientSocketArray{
    if (!_clientSocketArray) {
        _clientSocketArray = [NSMutableArray array];
    }
    return _clientSocketArray;
}

- (instancetype)init{
    if (self = [super init]) {
        // 1. 创建服务端Socket对象
        self.ServerSocket = [[GCDAsyncSocket alloc] initWithDelegate:self delegateQueue:dispatch_get_global_queue(0, 0)];
    }
    return self;
}

- (void)start{
    // 2. 绑定端口,马上监听端口
    NSError *error = nil;
    [self.ServerSocket acceptOnPort:5288 error:&error];
    
    if (!error) {
        NSLog(@"服务端开启成功");
    }else{
        NSLog(@"服务端开启失败");
    }
}

#pragma mark - GCDAsyncSocketDelegate
// 接收到客户端 连接请求
- (void)socket:(GCDAsyncSocket *)sock didAcceptNewSocket:(GCDAsyncSocket *)newSocket{
#warning 将新连接的客户端保存起来,避免断开
    [self.clientSocketArray addObject:newSocket];
    
    NSLog(@"连接上的客户端 %ld", self.clientSocketArray.count);
    
    
#warning 如果想要接收数据,必须要监听数据的读取
    [newSocket readDataWithTimeout:-1 tag:0];
}


// 接收到客户端的数据
- (void)socket:(GCDAsyncSocket *)sock didReadData:(NSData *)data withTag:(long)tag{
    // 将二进制数据转换为字符串
    NSString *str = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
    NSLog(@"%@", str);
    
#warning 进行数据读取之前一定要先写这句
    [sock readDataWithTimeout:-1 tag:0];
    
    // 把数据发送给其他客户端
    for (GCDAsyncSocket *client in self.clientSocketArray) {
        if (client != sock) {
            [client writeData:data withTimeout:-1 tag:0];
        }
    }
}
@end
```
>注：iOS项目 会自动开启主运行循环，而 命令行项目 必须手动开启

- 运行程序，服务端就开始运行了

- 终端充当客户端，输入以下命令来连接10086服务器：
	- **telnet 192.168.10.10 5288**- 其中 `192.168.10.10`为服务器的IP地址，`5288`为服务器端口号---- <群聊客户端>

```
#import "ViewController.h"
#import "GCDAsyncSocket.h"

@interface ViewController ()<GCDAsyncSocketDelegate, UITableViewDataSource>
@property (weak, nonatomic) IBOutlet NSLayoutConstraint *inputViewBottom;
@property (weak, nonatomic) IBOutlet UIView *inputView;
@property (weak, nonatomic) IBOutlet UIButton *sendButton;
@property (weak, nonatomic) IBOutlet UITableView *tableView;
@property (weak, nonatomic) IBOutlet UITextField *textField;

/** 客户端Socket */
@property (nonatomic, strong) GCDAsyncSocket *clientSocket;
/** 存放数据的数组 */
@property (nonatomic, strong) NSMutableArray *dataSource;
@end

@implementation ViewController

static NSString * const cellID = @"cell";

/** dataSource的懒加载 */
- (NSMutableArray *)dataSource{
    if (!_dataSource) {
        _dataSource = [NSMutableArray array];
    }
    return _dataSource;
}

- (void)viewDidLoad {
    [super viewDidLoad];
    
    // 1. 连接服务器
    // 1.1 创建客户端Socket对象
    GCDAsyncSocket *clientSocket = [[GCDAsyncSocket alloc] initWithDelegate:self delegateQueue:dispatch_get_global_queue(0, 0)];
    // 1.2 连接
    NSError *error = nil;
    [clientSocket connectToHost:@"192.168.1.102" onPort:5288 error:&error];
    if (error) {
        NSLog(@"%@", error);
    }
    self.clientSocket = clientSocket;
    
    // 注册cell
    [self.tableView registerClass:[UITableViewCell class] forCellReuseIdentifier:cellID];
}


#pragma mark - GCDAsyncSocketDelegate
// 2. 监听数据读取
/**
 *  连接到服务器后调用
 */
- (void)socket:(GCDAsyncSocket *)sock didConnectToHost:(NSString *)host port:(uint16_t)port{
    NSLog(@"与服务器连接成功,建立了通信连接");
    
#warning 客户端连接成功后,要监听数据读取
    [sock readDataWithTimeout:-1 tag:0];
}

/**
 *  与服务器断开连接时调用
 */
- (void)socketDidDisconnect:(GCDAsyncSocket *)sock withError:(NSError *)err{
    NSLog(@"与服务器断开连接 %@", err);
}

/**
 *  接收数据
 */
- (void)socket:(GCDAsyncSocket *)sock didReadData:(NSData *)data withTag:(long)tag{
    NSString *str = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
    NSLog(@"%@ %@", str, [NSThread currentThread]);
    
    // 准备读取下次的数据
    [sock readDataWithTimeout:-1 tag:0];
    
    // -----刷新表格
    [self.dataSource addObject:str];
#warning 当前所有的代理都是在子线程调用
    // 回到主线程刷新表格
    [[NSOperationQueue mainQueue] addOperationWithBlock:^{
        [self.tableView reloadData];
    }];
    
}


#pragma mark - Table View Data Source
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section{
    return self.dataSource.count;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
    UITableViewCell *cell = [self.tableView dequeueReusableCellWithIdentifier:cellID];
    cell.textLabel.text = self.dataSource[indexPath.row];
    
    return cell;
}


- (IBAction)sendBtnClick{
    // 获取用户输入的文字
    NSString *text = self.textField.text;
    if (text.length == 0) {
        return;
    }
    
    // 发送
    [self.clientSocket writeData:[text dataUsingEncoding:NSUTF8StringEncoding] withTimeout:-1 tag:0];
    
    // 刷新表格
    [self.dataSource addObject:text];
    [self.tableView reloadData];
}

- (void)socket:(GCDAsyncSocket *)sock didWriteDataWithTag:(long)tag{
    NSLog(@"数据发送成功");
}

@end
```

##短连接与长连接

###短连接
**连接**->**传输数据**->**关闭连接**

HTTP是无状态的，浏览器和服务器每进行一次HTTP操作，就建立一次连接，但任务结束就**中断连接**。

也可以这样说：短连接是指SOCKET连接后发送后接收完数据后马上断开连接。
 
###长连接
**连接**->**传输数据**->**保持连接** ->**传输数据**-> 。。。 ->**关闭连接**。

长连接指建立Socket连接后不管是否使用都**保持连接**，但安全性较差。

关于长连接我们在介绍**推送通知**的时候接触过

##Socket层上的协议
- Socket层上的协议指的**数据传输的格式**- HTTP协议的传输格式   
   http1.1  - **content-type**:multipart/form-data,  - **content-length**:188,  - **body**:username=zhangsan&password=123456- **XMPP协议，是一款即时通讯协议**  - 可扩展消息处理现场协议）是基于可扩展标记语言（XML）的协议，它用于即时消息（IM）以及在线现场探测。这个协议可能最终允许因特网用户向因特网上的其他任何人发送即时消息  
  传输格式：

```  <from>zhangsan<from>   <to>lisi<to>   <body>一起吃晚上</body>```

- HTTP/XMPP 等协议 定义了数据传输的**格式**
- TCP/UDP 等定义了数据传输的**方式**
![](https://dn-zhunjiee.qbox.me/Snip20151229_8.png)

---

##什么是XMPP?
- XMPP：The Extensible Messaging and Presence Protocol（可扩展通讯和表示协议）
- XMPP是一种基于**XML的即时通讯协议**，XMPP的官方文档是RFC 3920	- 这个文档定义了登录，退出，获取好友，发送消息等等的**XML数据传输协议**XMPP是一个典型的**C/S**架构，基本的网络形式是客户端通过**TCP/IP**连接到服务器，通过**Socket**建立连接，然后在之上传输XML流XMPP是一种类似于HTTP协议的一种数据传输协议，其过程就如同“解包装--〉包装”的过程。只需要理解其接收的类型及返回的类型，便可以很好的利用XMPP来进行数据通讯XMPP官方网站——http://xmpp.org###环信简介
环信是在XMPP的基础上进行二次开发，将基于移动互联网的即时通讯能力，如单聊、群聊、发语音、发图片、发位置、实时音频、实时视频等，通过云端开放的 Rest API 和客户端 SDK 包的方式提供给开发者和企业。让App内置聊天功能和以前网页中嵌入分享功能一样简单。