---
layout: post
title: "iOS中的蓝牙-GameKit用法"
date: 2014-07-01
comments: false
categories: 实用技术
---

### 概述(更新)
#### iOS中提供了4个框架用于实现蓝牙连接
- 1.GameKit.framework(用法简单)
    * `只能用于iOS设备之间的同个应用内连接`,多用于游戏(eg.拳皇,棋牌类),从`iOS7开始过期`

- 2.MultipeerConnectivity.framework(代替1)
    * `只能用于iOS设备之间的连接,从iOS7开始引入`,主要用于`非联网状态`下,通过wifi或者蓝牙进行文件共享(仅限于沙盒的文件),多用于附近无网聊天

- 3.ExternalAccessory.framework(MFi)
    * `可用于第三方蓝牙设备交互`,但是蓝牙设备必须经过`苹果MFi认证`(国内很少)

- 4.CoreBluetooth.framework（时下热门)
    * `可用于第三方蓝牙设备交互`,必须要支持蓝牙4.0
    * 硬件至少是4s,系统至少是iOS6
    * 蓝牙4.0以低功耗著称,一般也叫BLE（Bluetooth Low Energy）
    * 目前应用比较多的案例:运动手环,嵌入式设备,智能家居

#### 设计到的系统/框架
- HealthKit/物联网HomeKit/wathOS1,2/iBeacon

# GameKit用法
## 一.准备工作
- 1.搭建UI
![图1](https://dn-zhunjiee.qbox.me/Snip20150928_1.png)

- 2.拖线

```objc
// 图片
@property (weak, nonatomic) IBOutlet UIImageView *imageView;

// 建立连接
- (IBAction)buildConnect:(id)sender{}

// 发送数据
- (IBAction)sendData:(id)sender{}
```
## 二.连接蓝牙
- 显示可以连接的蓝牙设备列表

```objc
- (IBAction)buildConnect:(id)sender {
    // 创建弹窗
    GKPeerPickerController *ppc = [[GKPeerPickerController alloc] init];
    // 设置代理  @interface ViewController () <GKPeerPickerControllerDelegate>
    ppc.delegate = self;
    // 展示
    [ppc show];
}
```
- 监听蓝牙的连接

```objc
#pragma mark -GKPeerPickerControllerDelegate
// 连接成功就会调用
- (void)peerPickerController:(GKPeerPickerController *)picker // 弹窗
              didConnectPeer:(NSString *)peerID // 连接到的蓝牙设备号
                   toSession:(GKSession *)session // 连接会话(通过它进行数据交互)
{
    NSLog(@"%s, line = %d", __FUNCTION__, __LINE__);
    // 弹窗消失
    [picker dismiss];
}
```

##利用蓝牙传输数据

- 点击图片从相册中选择一张显示本机
    * 可以修改imaV为Btn,也可以为imaV添加手势
        * 1.修改imageView的用户交互
    ![图2](https://dn-zhunjiee.qbox.me/Snip20150928_2.png)
        * 2.添加手势到图片上
    ![图3](https://dn-zhunjiee.qbox.me/Snip20150928_3.png)
        * 3.拖出手势的响应事件
    ![图4](https://dn-zhunjiee.qbox.me/Snip20150928_4.png)
        * 4.完善相册选择图片代码

```objc
    // 手势-点击从相册中选一张照片
- (IBAction)tapImage:(UITapGestureRecognizer *)sender {
    NSLog(@"%s, line = %d", __FUNCTION__, __LINE__);
    // 先判断是否有相册
    if (![UIImagePickerController isSourceTypeAvailable:UIImagePickerControllerSourceTypePhotoLibrary]) {
        return;
    }
    // 创建弹出的控制器
    UIImagePickerController *ipc = [[UIImagePickerController alloc] init];
    // 设置图片来源为相册
    ipc.sourceType = UIImagePickerControllerSourceTypePhotoLibrary;
    // 设置代理 @interface ViewController () <UINavigationControllerDelegate, UIImagePickerControllerDelegate>
    ipc.delegate = self;
    // modal出来
    [self presentViewController:ipc animated:YES completion:nil];
}
#pragma mark - UINavigationControllerDelegate, UIImagePickerControllerDelegate
// 选中某图片后调用
- (void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary<NSString *,id> *)info
{
    // 控制器返回
    [picker dismissViewControllerAnimated:YES completion:nil];
    // 设置图片
    self.imageView.image = info[UIImagePickerControllerOriginalImage];
}
```

- 点击发送数据完成图片显示到另一个蓝牙机器上
    * 1.分析需要通过GKSession对象来传递数据,所以在`peerPickerController:didConnectPeer:didConnectPeer:`的方法中保存session会话

```objc
@property (nonatomic, strong) GKSession *session; /**< 蓝牙连接会话 */

// 连接成功就会调用
- (void)peerPickerController:(GKPeerPickerController *)picker // 弹窗
              didConnectPeer:(NSString *)peerID // 连接到的蓝牙设备号
                   toSession:(GKSession *)session // 连接会话(通过它进行数据交互)
{
    NSLog(@"%s, line = %d", __FUNCTION__, __LINE__);
    // 弹窗消失
    [picker dismiss];
    // 保存会话
    self.session = session;
}
```
- 发送

```objc
// 发送数据
- (IBAction)sendData:(id)sender {
    if (self.imageView.image == nil) return; // 有图片才继续执行
    // 通过蓝牙链接会话发送数据到所有设备
    [self.session sendDataToAllPeers:UIImagePNGRepresentation(self.imageView.image) // 数据
                        withDataMode:GKSendDataReliable // 枚举:发完为止
                               error:nil];

}
```

- 接收

```objc
// 连接成功就会调用
- (void)peerPickerController:(GKPeerPickerController *)picker // 弹窗
              didConnectPeer:(NSString *)peerID // 连接到的蓝牙设备号
                   toSession:(GKSession *)session // 连接会话(通过它进行数据交互)
{
    NSLog(@"%s, line = %d", __FUNCTION__, __LINE__);
    // 弹窗消失
    [picker dismiss];
    // 保存会话
    self.session = session;
    // 处理接收到的数据[蓝牙设备接收到数据时,就会调用 [self receiveData:fromPeer:inSession:context:]]
    // 设置数据接受者为:self
    [self.session setDataReceiveHandler:self
                       withContext:nil];
}
#pragma mark - 蓝牙设备接收到数据时,就会调用
- (void)receiveData:(NSData *)data // 数据
           fromPeer:(NSString *)peer // 来自哪个设备
          inSession:(GKSession *)session // 连接会话
            context:(void *)context
{
    NSLog(@"%s, line = %d", __FUNCTION__, __LINE__);
    // 显示
    self.imageView.image = [UIImage imageWithData:data];
    // 写入相册
    UIImageWriteToSavedPhotosAlbum(self.imageView.image, nil, nil, nil);
}


```

## 四.注意

- 只能用于iOS设备之间的链接
- 只能用于同一个应用程序之间的连接
- 最好别利用蓝牙发送比较大的数据
