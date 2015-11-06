---
layout: post
title: "iOS开发之MapKit"
date: 2014-10-09
comments: false
categories: 实用技术
---
##MapKit框架的使用
1. 导入框架(MapKit.framework)
2. 导入主头文件
`#import <MapKit/MapKit.h>`

###MapKit框架使用须知
- MapKit框架中所有数据类型的前缀都是MK
- MapKit有一个比较重要的UI控件 ：MKMapView，专门用于地图显示

##跟踪显示用户的位置
设置MKMapView的userTrackingMode属性可以跟踪显示用户的当前位置
- MKUserTrackingModeNone ：不跟踪用户的位置
- MKUserTrackingModeFollow ：跟踪并在地图上显示用户的当前位置
- MKUserTrackingModeFollowWithHeading ：跟踪并在地图上显示用户的当前位置，地图会跟随用户的前进方向进行旋转

##地图的类型
可以通过设置MKMapView的mapViewType设置地图类型
- MKMapTypeStandard ：普通地图（左图）
- MKMapTypeSatellite ：卫星云图 （中图）
- MKMapTypeHybrid ：普通地图覆盖于卫星云图之上（右图） 
- MKMapTypeSatelliteFlyover NS_ENUM_AVAILABLE(10_11, 9_0), // 立体卫星
- MKMapTypeHybridFlyover NS_ENUM_AVAILABLE(10_11, 9_0), // 立体混合

##MapKit常见属性
```objc
// 设置地图类型
    /*
     MKMapTypeStandard = 0, // 标准地图
     MKMapTypeSatellite, // 卫星地图
     MKMapTypeHybrid, // 混合
     MKMapTypeSatelliteFlyover NS_ENUM_AVAILABLE(10_11, 9_0), // 立体卫星
     MKMapTypeHybridFlyover NS_ENUM_AVAILABLE(10_11, 9_0), // 立体混合
     */
    self.mapView.mapType = MKMapTypeHybrid;
    
// 设置地图的控制项
    // 控制滚动
    self.mapView.scrollEnabled = NO;
    // 控制缩放
    self.mapView.zoomEnabled = NO;
    // 控制旋转
    self.mapView.rotateEnabled = NO;
    // 3D视图
    self.mapView.pitchEnabled = NO;
    
// 设置地图的显示项
    // 显示建筑物
    self.mapView.showsBuildings = YES;
    // 指南针
    self.mapView.showsCompass = YES;
    // 兴趣点
    self.mapView.showsPointsOfInterest = YES;
    // 比例尺
    self.mapView.showsScale = YES;
    // 显示交通
    self.mapView.showsTraffic = YES;
    
// 显示用户位置
    // 请求用户定位授权
    [self locManager];
    // 在地图上显示用户位置,地图不会跟着用户的变化而变化(不会自动放大地图)
    // self.mapView.showsUserLocation = YES;
    
    /**
     MKUserTrackingModeNone = 0, // 不追踪
     MKUserTrackingModeFollow, // 追踪
     MKUserTrackingModeFollowWithHeading, // 带方向的追踪
     */
    // 请求授权
    // 地图会跟着用户位置的变化而变化(会自动放大地图)
    self.mapView.userTrackingMode = MKUserTrackingModeFollowWithHeading;
```

##MKMapView的代理
MKMapView可以设置一个代理对象，用来监听地图的相关行为

- 常见的代理方法有

```objc
一个位置更改默认只会调用一次，不断监测用户的当前位置
每次调用，都会把用户的最新位置（userLocation参数）传进来
- (void)mapView:(MKMapView *)mapView didUpdateUserLocation:(MKUserLocation *)userLocation{
    // 显示地图中心且缩放到合适的比例
    // 创建一个区域跨度
    MKCoordinateSpan span = MKCoordinateSpanMake(0.040493, 0.029611);
    // 根据一个中心和跨度,创建一个区域
    MKCoordinateRegion region = MKCoordinateRegionMake(userLocation.location.coordinate, span);
    // 设置地图区域
    [mapView setRegion:region animated:YES];   
}


- (void)mapView:(MKMapView *)mapView regionWillChangeAnimated:(BOOL)animated;
地图的显示区域即将发生改变的时候调用

- (void)mapView:(MKMapView *)mapView regionDidChangeAnimated:(BOOL)animated;
地图的显示区域已经发生改变的时候调用
```
```objc
mapView.region.span.latitudeDelta
    
mapView.region.span.longitudeDelta
```


##MKUserLocation
MKUserLocation其实是个大头针模型，包括以下属性

```objc
@property (nonatomic, copy) NSString *title;
显示在大头针上的标题

@property (nonatomic, copy) NSString *subtitle;
显示在大头针上的子标题

@property (readonly, nonatomic) CLLocation *location;
地理位置信息(大头针钉在什么地方?)
```

##设置地图的显示

通过MKMapView的下列方法，可以设置地图显示的位置和区域

```objc
设置地图的中心点位置
@property (nonatomic) CLLocationCoordinate2D centerCoordinate;
- (void)setCenterCoordinate:(CLLocationCoordinate2D)coordinate animated:(BOOL)animated;

设置地图的显示区域
@property (nonatomic) MKCoordinateRegion region;
- (void)setRegion:(MKCoordinateRegion)region animated:(BOOL)animated;
```

##MKCoordinateRegion
MKCoordinateRegion是一个用来表示区域的结构体，定义如下

```objc
typedef struct {
      CLLocationCoordinate2D center; // 区域的中心点位置
      MKCoordinateSpan span; // 区域的跨度
} MKCoordinateRegion;

MKCoordinateSpan的定义
typedef struct {
    CLLocationDegrees latitudeDelta; // 纬度跨度
    CLLocationDegrees longitudeDelta; // 经度跨度
} MKCoordinateSpan;

```

##大头针的基本操作

```objc
添加一个大头针
- (void)addAnnotation:(id <MKAnnotation>)annotation;

添加多个大头针
- (void)addAnnotations:(NSArray *)annotations;

移除一个大头针
- (void)removeAnnotation:(id <MKAnnotation>)annotation;

移除多个大头针
- (void)removeAnnotations:(NSArray *)annotations;

(id <MKAnnotation>)annotation参数是什么东西？
大头针模型对象：用来封装大头针的数据，比如大头针的位置、标题、子标题等数据

```
##大头针模型
```objc
新建一个大头针模型类
#import <MapKit/MapKit.h>

@interface MyAnnotation : NSObject <MKAnnotation>
/** 坐标位置 */
@property (nonatomic, assign) CLLocationCoordinate2D coordinate;
/** 标题 */
@property (nonatomic, copy) NSString *title; 
/** 子标题 */
@property (nonatomic, copy) NSString *subtitle; 
@end
```
##添加大头针
原理:添加大头针实际上就是添加大头针模型

```objc
MyAnnotation *anno = [[MyAnnotation alloc] init];
anno.title = @"侯大大帝国";
anno.subtitle = @"中共威武,党国万岁";
anno.coordinate = CLLocationCoordinate2DMake(40, 116);
[self.mapView addAnnotation:anno];
```
##自定义大头针(也有重复利用机制)
如何自定义大头针?

- 设置MKMapView的代理
- 实现下面的代理方法，返回大头针控件
    ` - (MKAnnotationView *)mapView:(MKMapView *)mapView viewForAnnotation:(id <MKAnnotation>)annotation;`
- 根据传进来的(id <MKAnnotation>)annotation参数创建并返回对应的大头针控件
- 代理方法的使用注意
- 如果返回nil，显示出来的大头针就采取系统的默认样式
- 标识用户位置的蓝色发光圆点，它也是一个大头针，当显示这个大头针时，也会调用代理方法
- 因此，需要在代理方法中分清楚(id <MKAnnotation>)annotation参数代表自定义的大头针还是蓝色发光圆点

```objc
- (MKAnnotationView *)mapView:(MKMapView *)mapView viewForAnnotation:(id<MKAnnotation>)annotation
{
    // 判断annotation的类型
    if (![annotation isKindOfClass:[MJTuangouAnnotation class]]) return nil;
    
    // 创建MKAnnotationView
    static NSString *ID = @"tuangou";
    MKAnnotationView *annoView = [mapView dequeueReusableAnnotationViewWithIdentifier:ID];
    if (annoView == nil) {
        annoView = [[MKAnnotationView alloc] initWithAnnotation:annotation reuseIdentifier:ID];
        // 设置弹框
        annoView.canShowCallout = YES;
    }
       
    "一定不要忘了写下面的内容"
    // 传递模型数据
    annoView.annotation = annotation;
    
    // 设置图片
        MJTuangouAnnotation *tuangouAnnotation = annotation;
    annoView.image = [UIImage imageNamed:tuangouAnnotation.icon];
    
    return annoView;
}
```
##MKAnnotationView
地图上的大头针控件是MKAnnotationView

- MKAnnotationView的属性

```objc
@property (nonatomic, strong) id <MKAnnotation> annotation;
大头针模型

@property (nonatomic, strong) UIImage *image;
显示的图片

@property (nonatomic) BOOL canShowCallout;
是否显示标注(弹框)

@property (nonatomic) CGPoint calloutOffset;
标注的偏移量

@property (strong, nonatomic) UIView *rightCalloutAccessoryView;
标注右边显示什么控件

@property (strong, nonatomic) UIView *leftCalloutAccessoryView;
标注左边显示什么控件
```

MKPinAnnotationView是MKAnnotationView的子类

```objc
MKPinAnnotationView比MKAnnotationView多了2个属性
@property (nonatomic) MKPinAnnotationColor pinColor;
大头针颜色

@property (nonatomic) BOOL animatesDrop;
大头针第一次显示时是否从天而降
```

##坐标点转经纬度
```objc
- (CLLocationCoordinate2D)convertPoint:(CGPoint)point toCoordinateFromView:(UIView *)view;
```
##系统导航(MKMapItem)
```objc
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event{
    
    [self.geoCoder geocodeAddressString:@"广州" completionHandler:^(NSArray *placemarks, NSError *error) {
        // 广州地标对象
        CLPlacemark *gzPL = [placemarks firstObject];
        
        [self.geoCoder geocodeAddressString:@"上海" completionHandler:^(NSArray *placemarks, NSError *error) {
            // 上海地标对象
            CLPlacemark *shPL = [placemarks firstObject];
            [self systemNavigationWithBeginPlacemark:gzPL endPlacemark:shPL];
        }];
    }];

}


- (void)systemNavigationWithBeginPlacemark:(CLPlacemark *)beginPlacemark endPlacemark:(CLPlacemark *)endPlacemark{
    // --------创建起点和终点
    // 起点
    // 创建一个地标对象
    CLPlacemark *startCLPlacemark = beginPlacemark;
    MKPlacemark *startMKPlacemark= [[MKPlacemark alloc] initWithPlacemark:startCLPlacemark];
    MKMapItem *startItem = [[MKMapItem alloc] initWithPlacemark:startMKPlacemark];
    
    // --------创建一个终点
    CLPlacemark *endCLPlacemark = endPlacemark;
    MKPlacemark *endMKPlacemark = [[MKPlacemark alloc] initWithPlacemark:endCLPlacemark];
    MKMapItem *endItem = [[MKMapItem alloc] initWithPlacemark:endMKPlacemark];
    
    NSArray *items = @[startItem, endItem];
    
    // --------系统地图启动选项
    NSDictionary *dict = @{
                           // 导航模式
                           MKLaunchOptionsDirectionsModeKey:MKLaunchOptionsDirectionsModeDriving,
                           // 地图类型
                           MKLaunchOptionsMapTypeKey:@(MKMapTypeStandard),
                           // 是否显示交通
                           MKLaunchOptionsShowsTrafficKey:@(YES)
                               };
    
    // ---------利用系统的app,给我们导航
    [MKMapItem openMapsWithItems:items launchOptions:dict];
}
```
##获取导航路线
```objc
- (void)requestRoutesWithSource:(CLPlacemark *)sourcePlacemark andDestination:(CLPlacemark *)destPlacemark{
    
    // 1. 创建导航请求
    MKDirectionsRequest *request = [[MKDirectionsRequest alloc] init];
    // 1.1设置起点
    MKPlacemark *sourcePL = [[MKPlacemark alloc] initWithPlacemark:sourcePlacemark];
    MKMapItem *sourceItem = [[MKMapItem alloc] initWithPlacemark:sourcePL];
    request.source = sourceItem;
    // 1.2设置终点
    MKPlacemark *destPL = [[MKPlacemark alloc] initWithPlacemark:destPlacemark];
    MKMapItem *destItem = [[MKMapItem alloc] initWithPlacemark:destPL];
    request.destination = destItem;
    
    // 2. 创建导航对象
    MKDirections *directions = [[MKDirections alloc] initWithRequest:request];
    
    // 3. 请求导航路线信息
    [directions calculateDirectionsWithCompletionHandler:^(MKDirectionsResponse *response, NSError *error) {
        NSLog(@"已经请求到导航路线信息");

        /**        
         ----MKDirectionsResponse : 路线响应对象----
         routes : 路线数组
         
         ----MKRoute : 路线对象------
         name : 路线名称
         advisoryNotices : 警告信息数组
         distance : 路线距离
         expectedTravelTime : 期望时间
         transportType : 交通方式
         polyline : 几何路线的数据模型
         steps : 行走的步骤
         
         
         -----MKRouteStep-----
         instructions : 行走提醒信息
         notice : 警告
         polyline : 一小节几何路线数据模型
         distance : 距离
         transportType : 通过方式
         */

        [response.routes enumerateObjectsUsingBlock:^(MKRoute *obj, NSUInteger idx, BOOL *stop) {
            NSLog(@"%@----%f", obj.name, obj.distance);
            
            [obj.steps enumerateObjectsUsingBlock:^(MKRouteStep *obj, NSUInteger idx, BOOL *stop) {
                NSLog(@"%@", obj.instructions);
            }];
    }];
}
```

