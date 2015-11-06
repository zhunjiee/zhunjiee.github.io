---
layout: post
title: "iOS开发之CoreLocation定位"
date: 2014-10-08
comments: false
categories: 实用技术
---

##地图定位
- 首先需要了解2个热门专业术语（装X专用） :
    - LBS ：Location Based Service 位置基础服务
    - SoLoMo ：Social Local Mobile（索罗门） 

##CoreLocation使用步骤
- 导入框架(Xcode5以后可以省略)
- 导入主头文件
`#import <CoreLocation/CoreLocation.h>`

- 创建位置管理者 @property(nonatomic, strong) CLLocationManager *manager;



##iOS8.0以前
- **iOS8以前,获取用户的位置相对是简单一点的**
- 开发者可以在Info.plist中设置NSLocationUsageDescription说明定位的目的(Privacy - Location Usage Description)
![](https://dn-zhunjiee.qbox.me/Snip20150912_2.png)

- 一旦用户选择了“Don’t Allow”，意味着你的应用以后就无法使用定位功能
- 为了严谨起见，最好在使用定位功能之前判断当前应用的定位功能是否可用
- CLLocationManager有个类方法可以判断当前应用的定位功能是否可用
`+ (BOOL)locationServicesEnabled;`

---
##iOS8.0
- **从iOS8开始,用户定位分两种情况**:
    - 总是使用用户位置:**NSLocationAlwaysUsageDescription**
    - 使用应用时定位:**NSLocationWhenInUseDescription**


- 开发者必须在Info.plist中设置NSLocationUsageDescription说明定位的目的(只需配置其中一种)
![](https://dn-zhunjiee.qbox.me/Snip20150912_1.png)

- 然后需要在代码中声明使用的是那种方法

```objc
#import "ViewController.h"
#import <CoreLocation/CoreLocation.h>

@interface ViewController () <CLLocationManagerDelegate>

/*用户位置管理者对象*/
@property (nonatomic, strong) CLLocationManager *locM;

@end

@implementation ViewController

#pragma mark - lazy
- (CLLocationManager *)locM{
    if (!_locM) {
        _locM = [[CLLocationManager alloc] init];
        
        // 精确度
        // _locM.desiredAccuracy = kCLLocationAccuracyHundredMeters;
        
        _locM.delegate = self;
        
        // 获取用户的授权状态
        // ios7只要使用到定位,就会直接请求授权,以下方法是不用写的
        CLAuthorizationStatus status = [CLLocationManager authorizationStatus];
        
        if (status == kCLAuthorizationStatusNotDetermined) {
            /*
             // 请求前台定位授权
             // 在前台授权下, 默认只能在前台获取用户位置信息, 如果想要在后台获取用户位置, 那么勾选Background Modes(后台模式)的 Location updates (会出现蓝条)
             // 如果当前的授权状态 != 用户未选择状态, 那么这个方法不会有效
             if ([[UIDevice currentDevice].systemVersion floatValue] >= 8.0) {
             [self.manager requestWhenInUseAuthorization];
             }
             */
            
            // 请求前后台定位授权
            // 无论是在前台还是后台, 都可以获取用户位置,而且不会出现蓝条
            // 无论是否勾选后台模式
            // 如果当前的授权状态 != 用户未选择状态, 那么这个方法不会有效
            // 如果当前授权状态 == 前台定位授权时, 这个方法也会有效
            if ([_locM respondsToSelector:@selector(requestAlwaysAuthorization)]) {
                [_locM requestAlwaysAuthorization];
            }
        }
        
        // 显著位置变化
        [_locM startMonitoringSignificantLocationChanges];

    }
    return _locM;
}
```

---

所以,我们到目前为止info.plist文件中是这样配置的
![](https://dn-zhunjiee.qbox.me/Snip20150912_3.png)



##获取设备版本号
```objc
[[UIDevice currentDevice].systemVersion floatValue]
```

---
（更新）
##iOS9.0
- 除了8.0的基本配置之外
- 在前台授权下, 默认只能在前台获取用户位置信息, 如果想要在后台获取用户位置, 那么去capabilities里勾选Background Modes(后台模式)的 Location updates 选项 (会出现蓝条), 需要额外的设置一下属性为YES

```objc
if ([[UIDevice currentDevice].systemVersion floatValue] >= 9.0) {
    _locM.allowsBackgroundLocationUpdates = YES;
}

```

##CLLocationManager的常用操作
```objc
@property(assign, nonatomic) CLLocationDistance distanceFilter;
每隔多少米定位一次

@property(assign, nonatomic) CLLocationAccuracy desiredAccuracy;
定位精确度（越精确就越耗电）

开始用户定位
- (void)startUpdatingLocation;

停止用户定位
- (void) stopUpdatingLocation;

// 开始获取设备方向
- (void)startUpdatingHeading;
// 停止获取设备方向
- (void)stopUpdatingHeading;



当调用了startUpdatingLocation方法后，就开始不断地定位用户的位置，中途会频繁地调用代理的下面方法
- (void)locationManager:(CLLocationManager *)manager didUpdateLocations:(NSArray *)locations;
locations参数里面装着CLLocation对象

显著位置变化
// 显著位置变化(基站)(要求设备必须由电话模块)
// 当app被杀死时也可以接收到位置通知(app -- 后台)
- (void)startMonitoringSignificantLocationChanges;



```

##CLLocation
```objc
CLLocation用来表示某个位置的地理信息，比如经纬度、海拔等等
@property(readonly, nonatomic) CLLocationCoordinate2D coordinate;
经纬度

@property(readonly, nonatomic) CLLocationDistance altitude;
海拔

@property(readonly, nonatomic) CLLocationDirection course;
路线，航向（取值范围是0.0° ~ 359.9°，0.0°代表真北方向）

@property(readonly, nonatomic) CLLocationSpeed speed;
行走速度（单位是m/s）

用- (CLLocationDistance)distanceFromLocation:(const CLLocation *)location方法可以计算2个位置之间的距离
```
##区域监听(了解)

1. 创建区域
1.1 确定区域中心
1.2 区域半径
2. 开始区域监听
3. 调用相关代理方法

```objc
#import "ViewController.h"
#import <CoreLocation/CoreLocation.h>

@interface ViewController () <CLLocationManagerDelegate>

@property (weak, nonatomic) IBOutlet UILabel *notice;

@property (nonatomic, strong) CLLocationManager *locM;

@end

@implementation ViewController
#pragma mark - lazy
- (CLLocationManager *)locM{
    if (!_locM) {
        _locM = [[CLLocationManager alloc] init];
        
        if ([[UIDevice currentDevice].systemVersion floatValue] >= 8.0) {
            [_locM requestAlwaysAuthorization];
        }
        
        _locM.delegate = self;
    }
    return _locM;
}

- (void)viewDidLoad {
    [super viewDidLoad];
    
    // 区域监听
    if ([CLLocationManager isMonitoringAvailableForClass:[CLCircularRegion class]]) {
        
        // 1. 创建区域
        // 1.1 确定区域中心
        CLLocationCoordinate2D center = {21.123, 121.345};
        // 1.2 区域半径
        CLLocationDistance distance = 1000;
        if (distance > self.locM.maximumRegionMonitoringDistance) {
            distance = self.locM.maximumRegionMonitoringDistance;
        }
        // 创建区域
        CLCircularRegion *region = [[CLCircularRegion alloc] initWithCenter:center radius:distance identifier:@"haha"];
        
//        // 2. 监听区域
//        [self.locM startMonitoringForRegion:region];
        
        // 请求某个区域当前状态(在后台也可以监听到某个区域)
        [self.locM requestStateForRegion:region];
    }
}


#pragma mark - CLLocationManagerDelegate

// *  进入区域
- (void)locationManager:(CLLocationManager *)manager didEnterRegion:(CLRegion *)region{
    NSLog(@"进入区域%@", region.identifier);
    self.notice.text = @"欢迎来到哈哈哈";
}

// *  离开区域
- (void)locationManager:(CLLocationManager *)manager didExitRegion:(CLRegion *)region{
    NSLog(@"离开区域%@", region.identifier);
    self.notice.text = @"拜拜再见";
}

// *  请求区域状态时调用
- (void)locationManager:(CLLocationManager *)manager didDetermineState:(CLRegionState)state forRegion:(CLRegion *)region{
    if (state == CLRegionStateInside) {
        self.notice.text = @"欢迎来到哈哈哈";
    }else if (state == CLRegionStateOutside){
        self.notice.text = @"拜拜再见";
    }
}

//  注册区域失败调用
- (void)locationManager:(CLLocationManager *)manager monitoringDidFailForRegion:(CLRegion *)region withError:(NSError *)error{
    
}
@end
```


##CLGeocoder
- 使用CLGeocoder可以完成“地理编码”和“反地理编码”
- **地理编码**：根据给定的地名，获得具体的位置信息（比如经纬度、地址的全称等）
- **反地理编码**：根据给定的经纬度，获得具体的位置信息

```objc
地理编码方法
- (void)geocodeAddressString:(NSString *)addressString completionHandler:(CLGeocodeCompletionHandler)completionHandler;

反地理编码方法
- (void)reverseGeocodeLocation:(CLLocation *)location completionHandler:(CLGeocodeCompletionHandler)completionHandler;
```

##地理编码与反地理编码
```objc
// 地理编码
- (IBAction)geoCoder {
    NSString *addStr = self.address.text;
    
    // 容错
    if (addStr.length == 0) {
        return;
    }
    
    // 根据地址关键字,进行地理编码
    [self.geoC geocodeAddressString:addStr completionHandler:^(NSArray *placemarks, NSError *error) {
        /**
         *  CLPlacemark
         *
         * 地标
         * location : 编码对应的位置对象
         * addressDictionary : 地址字典
         
         * name :全称
         thoroughfare : 街道
         locality : 城市
         */
        CLPlacemark *placeM = [placemarks firstObject];
        if (error == nil) {
            self.address.text = placeM.name;
            self.latitude.text = @(placeM.location.coordinate.latitude).stringValue;
            self.longitude.text = @(placeM.location.coordinate.longitude).stringValue;
        }
    }];
}

// 反地理编码
// 根据经纬度 -> 地址
- (IBAction)resvrse {
    // 纬度
    double latitude = [self.latitude.text doubleValue];
    
    // 纬度
    double longitude = [self.longitude.text doubleValue];
    
    CLLocation *location = [[CLLocation alloc] initWithLatitude:latitude longitude:longitude];
    [self.geoC reverseGeocodeLocation:location completionHandler:^(NSArray *placemarks, NSError *error) {
        CLPlacemark *placeM = [placemarks firstObject];
        if (error == nil) {
            self.address.text = placeM.name;
            self.latitude.text = @(placeM.location.coordinate.latitude).stringValue;
            self.longitude.text = @(placeM.location.coordinate.longitude).stringValue;
        }
    }];
}
```


##CLGeocodeCompletionHandler

- 当地理\反地理编码完成时，就会调用CLGeocodeCompletionHandler

```objc
typedef void (^CLGeocodeCompletionHandler)(NSArray *placemarks, NSError *error);
这个block传递2个参数
error ：当编码出错时（比如编码不出具体的信息）有值
placemarks ：里面装着CLPlacemark对象
```
##CLPlacemark
```objc
CLPlacemark的字面意思是地标，封装详细的地址位置信息
@property (nonatomic, readonly) CLLocation *location;
地理位置

@property (nonatomic, readonly) CLRegion *region;
区域

@property (nonatomic, readonly) NSDictionary *addressDictionary;
详细的地址信息

@property (nonatomic, readonly) NSString *name;
地址名称

@property (nonatomic, readonly) NSString *locality;
城市
```

