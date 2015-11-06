---
layout: post
title: "iOS中KVC、KVO、NSNotification、delegate 总结及区别"
date: 2014-04-20
comments: false
categories: iOS
---                               

###1、KVC，即是指 NSKeyValueCoding，一个非正式的Protocol，提供一种机制来间接访问对象的属性。而不是通过调用Setter、Getter方法访问。KVO 就是基于 KVC 实现的关键技术之一。

Demo：

```objc
@interface myPerson : NSObject
{    
        NSString*_name;    
        int      _age;    
        int      _height;    
        int      _weight;
} @end
 
@interface testViewController :UIViewController 
 
@property (nonatomic, retain) myPerson*testPerson; 
 
@end
 
- (void)testKVC
{    
testPerson = [[myPerson alloc] init];        
 NSLog(@"testPerson‘s init height =%@", [testPerson valueForKey:@"height"]);   
[testPerson setValue:[NSNumber numberWithInt:168]forKey:@"height"];
        NSLog(@"testPerson‘s height = %@", [testPerson valueForKey:@"height"]);
}
```

- 第一段代码是定义了一个myPerson的类，这个类有一个_height的属性，但是没有提供任何getter／setter的访问方法。同时在testViewController这个类里面有一个myPerson的对象指针。
- 当myPerson实例化后，常规来说是无法访问这个对象的_height属性的，不过通过KVC我们做到了，代码就是testKVC这个函数。
- 运行之后打印值就是：
<pre>
2015-3-13 11:16:21.970 test[408:c07] testPerson‘s init height = 0
2015-3-13 11:16:21.971 test[408:c07] testPerson‘s height = 168
</pre>
- 这就说明确实读写了_height属性。
 

- KVC的常用方法：

```
- (id)valueForKey:(NSString *)key; 
-(void)setValue:(id)value forKey:(NSString *)key;
```

- valueForKey的方法根据key的值读取对象的属性，setValue:forKey:是根据key的值来写对象的属性。

<pre>
注意：
（1）. key的值必须正确，如果拼写错误，会出现异常
（2）. 当key的值是没有定义的，valueForUndefinedKey:这个方法会被调用，如果你自己写了这个方法，key的值出错就会调用到这里来
（3）. 因为类key反复嵌套，所以有个keyPath的概念，keyPath就是用.号来把一个一个key链接起来，这样就可以根据这个路径访问下去
（4）. NSArray／NSSet等都支持KVC
</pre>

###2、KVO的是KeyValue Observe的缩写，中文是键值观察。这是一个典型的观察者模式，观察者在键值改变时会得到通知。iOS中有个Notification的机制，也可以获得通知，但这个机制需要有个Center，相比之下KVO更加简洁而直接。

- KVO的使用也很简单，就是简单的3步。
	- 1.注册需要观察的对象的属性addObserver:forKeyPath:options:context:
	- 2.实现observeValueForKeyPath:ofObject:change:context:方法，这个方法当观察的属性变化时会自动调用
	- 3.取消注册观察removeObserver:forKeyPath:context:

Demo：

```objc
@interface myPerson : NSObject  
{  
    NSString *_name;  
    int      _age;  
    int      _height;  
    int      _weight;  
}  
@end  
  
@interface testViewController : UIViewController  
@property (nonatomic, retain) myPerson *testPerson;  
  
- (IBAction)onBtnTest:(id)sender;  
@end  
  
- (void)testKVO  
{  
    testPerson = [[myPerson alloc] init];  
      
    [testPerson addObserver:self forKeyPath:@"height" options:NSKeyValueObservingOptionNew context:nil];  
}  
  
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context  
{  
    if ([keyPath isEqualToString:@"height"]) {  
        NSLog(@"Height is changed! new=%@", [change valueForKey:NSKeyValueChangeNewKey]);  
    } else {  
        [super observeValueForKeyPath:keyPath ofObject:object change:change context:context];  
    }  
}  
  
- (IBAction)onBtnTest:(id)sender {  
    int h = [[testPerson valueForKey:@"height"] intValue];      
    [testPerson setValue:[NSNumber numberWithInt:h+1] forKey:@"height"];  
    NSLog(@"person height=%@", [testPerson valueForKey:@"height"]);  
}  
  
- (void)dealloc  
{  
    [testPerson removeObserver:self forKeyPath:@"height" context:nil];  
    [super dealloc];  
}  
```

- 第一段代码声明了myPerson类，里面有个_height的属性。在testViewController有一个testPerson的对象指针。
- 在testKVO这个方法里面，我们注册了testPerson这个对象height属性的观察，这样当testPerson的height属性变化时， 会得到通知。在这个方法中还通过NSKeyValueObservingOptionNew这个参数要求把新值在dictionary中传递过来。
- 重写了observeValueForKeyPath:ofObject:change:context:方法，这个方法里的change这个NSDictionary对象包含了相应的值。
- 需要强调的是KVO的回调要被调用，属性必须是通过KVC的方法来修改的，如果是调用类的其他方法来修改属性，这个观察者是不会得到通知的。

###3、NSNotification的用法见[http://blog.csdn.net/eduora_meimei/article/details/44198909](http://blog.csdn.net/eduora_meimei/article/details/44198909)

---

#区别：

---

##delegate
###优势 ：
1. 非常严格的语法。所有将听到的事件必须是在delegate协议中有清晰的定义。
2. 如果delegate中的一个方法没有实现那么就会出现编译警告/错误
3. 协议必须在controller的作用域范围内定义
4. 在一个应用中的控制流程是可跟踪的并且是可识别的；
5. 在一个控制器中可以定义定义多个不同的协议，每个协议有不同的delegates
6. 没有第三方对象要求保持/监视通信过程。
7. 能够接收调用的协议方法的返回值。这意味着delegate能够提供反馈信息给controller

###缺点 ：
1. 需要定义很多代码：1.协议定义；2.controller的delegate属性；3.在delegate本身中实现delegate方法定义
2. 在释放代理对象时，需要小心的将delegate改为nil。一旦设定失败，那么调用释放对象的方法将会出现内存crash
3. 在一个controller中有多个delegate对象，并且delegate是遵守同一个协议，但还是很难告诉多个对象同一个事件，不过有可能。

##notification
###优势 ：
1. 不需要编写多少代码，实现比较简单；
2. 对于一个发出的通知，多个对象能够做出反应，即1对多的方式实现简单
3. controller能够传递context对象（dictionary），context对象携带了关于发送通知的自定义的信息

###缺点 ：
1. 在编译期不会检查通知是否能够被观察者正确的处理； 
2. 在释放注册的对象时，需要在通知中心取消注册；
3. 在调试的时候应用的工作以及控制过程难跟踪；
4. 需要第三方对喜爱那个来管理controller与观察者对象之间的联系；
5. controller和观察者需要提前知道通知名称、UserInfodictionary keys。如果这些没有在工作区间定义，那么会出现不同步的情况；
6. 通知发出后，controller不能从观察者获得任何的反馈信息。

##KVO
###优势 ：
1. 能够提供一种简单的方法实现两个对象间的同步。例如：model和view之间同步；
2. 能够对非我们创建的对象，即内部对象的状态改变作出响应，而且不需要改变内部对象（SKD对象）的实现；
3. 能够提供观察的属性的最新值以及先前值；
4. 用key paths来观察属性，因此也可以观察嵌套对象；
5. 完成了对观察对象的抽象，因为不需要额外的代码来允许观察值能够被观察

###缺点 ：
1. 我们观察的属性必须使用strings来定义。因此在编译器不会出现警告以及检查；
2. 对属性重构将导致我们的观察代码不再可用；
3. 复杂的“IF”语句要求对象正在观察多个值。这是因为所有的观察代码通过一个方法来指向；
4. 当释放观察者时不需要移除观察者。

#对比:
###1. 效率肯定是delegate比NSNotification高。
- delegate方法比notification更加直接，最典型的特征是，delegate方法往往需要关注返回值，也就是delegate方法的结果。
- 比如-windowShouldClose:，需要关心返回的是yes还是no。所以delegate方法往往包含 should这个很传神的词。也就是好比你做我的delegate，我会问你我想关闭窗口你愿意吗？你需要给我一个答案，我根据你的答案来决定如何做下一步。
- 相反的，notification最大的特色就是不关心接受者的态度，我只管把通告放出来，你接受不接受就是你的事情，同时我也不关心结果。
- 所以notification往往用did这个词汇，比如NSWindowDidResizeNotification，那么NSWindow对象放出这个notification后就什么都不管了也不会等待接 受者的反应。

###2. KVO和NSNotification的区别：
- 和delegate一样，KVO和NSNotification的作用也是类与类之间的通信，
与delegate不同的是;
	- 1）这两个都是负责发出通知，剩下的事情就不管了，所以没有返回值；
    - 2）delegate只是一对一，而这两个可以一对多。这两者也有各自的特点。
