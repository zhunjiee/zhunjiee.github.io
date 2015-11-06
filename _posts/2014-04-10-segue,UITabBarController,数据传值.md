---
layout: post
title: "segue,UITabBarController,数据传值"
date: 2014-04-10
comments: false
categories: UI
---


#segue

###什么是segue
- Storyboard上每一根用来界面跳转的线，都是一个UIStoryboardSegue对象（简称Segue）

###segue的属性
- 每一个Segue对象，都有3个属性
    + 唯一标识
`@property (nonatomic, readonly) NSString *identifier;`

    + 来源控制器
`@property (nonatomic, readonly) id sourceViewController;`

    + 目标控制器
`@property (nonatomic, readonly) id destinationViewController;`

###Segue的类型

根据Segue的执行（跳转）时刻，Segue可以分为2大类型

- `自动型`：点击某个控件后（比如按钮），自动执行Segue，自动完成界面跳转

![](https://dn-zhunjiee.qbox.me/Snip20150813_1.png)


- `手动型`：需要通过写代码手动执行Segue，才能完成界面跳转
- 手动型的segue需要设置一个标识

![](https://dn-zhunjiee.qbox.me/Snip20150813_2.png)


##*手动执行segue进行页面跳转*
**1. 在恰当的时刻，使用`performSegueWithIdentifier`方法执行对应的Segue**

- `performSegueWithIdentifier`的底层实现(了解,不必实现):
	- 1.去storyboard中寻找segue线
	- 2.设置segue的来源控制器
	- 3.创建目的控制器
	- 4.通知来源控制器,线已经准备好了
	- 5.执行segue线,进行跳转[segue perform]

```objc
// Segue必须由来源控制器来执行，也就是说，这个perform方法必须由来源控制器来调用
[self performSegueWithIdentifier:@“login2contacts” sender:nil];
// 1.去storyboard中寻找segue线
// 2.设置segue的来源控制器
// 3.创建目的控制器
// 4.通知来源控制器,告诉来源控制器我的线已经准备好了
// 5.执行segue线,进行跳转[segue perform]
```
**2. 调用sourceViewController的`prepareForSegue`方法，做一些跳转前的准备工作并且传入创建好的Segue对象**

```objc
// 这个sender是当初performSegueWithIdentifier:sender:中传入的sender
// 做一些数据之间的传值
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender{
	 ZJContactViewController *目的控制器 = segue.destinationViewController;
    目的控制器.accountStr = _userName.text;
}

```

`prepareForSegue:`的底层实现(了解,不必实现):

- 调用Segue对象的`- (void)perform;`方法开始执行界面跳转操作

	- (1) **如果segue的style是`push`**
		- 取得sourceViewController所在的UINavigationController
		- 调用UINavigationController的`push`方法将`destinationViewController`压入栈中，完成跳转


	- (2) **如果segue的style是`modal`**
        - 调用sourceViewController的`presentViewController`方法将`destinationViewController`展示出来

#数据传值:
- 数据从(**传递方**)传递给(**接收方**)

注意:
- 1.接收方一定要有属性去接收
- 2.传递方需要拿到接收方,面对面交易

##顺传
- **来源控制器传值给目的控制器**

```objc
// 手动执行的时候才需要写
[self performSegueWithIdentifier:@“login2contacts” sender:nil];

// 准备好segue线之后,跳转之前会调用
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender{
    ZJContactViewController *vc = segue.destinationViewController;
    vc.accountStr = _userName.text;
}
```

##逆传
- **目的控制器(传递方)传递给来源控制器(接收方)**
- 1.在跳转之前,在**传递方**把**接收方**先保存起来
- 2.等需要真的给接收方传值的时候,直接拿到它传值


- 逆传步骤
    1. **接收方**声明属性
    2. **传递方**先定义代理和代理协议
    3. 在跳转到传递方之前,先设置**传递方**的代理为**接收方**
    4. 在**传递方**需要传递数据的时候,直接通知代理做事情,其实就是让代理调用协议方法

- 目的控制器(传递方)
- ZJContanctViewController

```objc
// 无论跳转到添加控制器还是编辑控制器都会调用
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender{
    
    if ([segue.destinationViewController isKindOfClass:[ZJAddViewController class]]) {
        // 添加控制器的时候才需要设置添加控制器的代理
        
        // 拿到添加控制器(传递方)
        // 给传递方设置代理为联系人控制器,传递方就能拿到接收方
        ZJAddViewController *vc = segue.destinationViewController;
        
        3. 在跳转到传递方之前,先设置传递方的代理为接收方
        vc.delegate = self;
        
    }else{// 进入编辑控制器
        ZJEditViewController *editVc = segue.destinationViewController;
        
        // 获取当前选中的角标
        NSIndexPath *indexPath = [self.tableView indexPathForSelectedRow];
        
        // 获取选中cell对应的模型
        ZJContact *contact = self.contacts[indexPath.row];
        
        // 选中的cell对应的模型
        editVc.contact = contact;
        
    }
}

5. 代理方法的实现
#pragma mark - ZJAddViewControllerDelegate
// 添加一个联系人会调用,会把新增的联系人传递过来
- (void)addViewController:(ZJAddViewController *)addVc didAddContact:(ZJContact *)contact{
    // 保存到联系人数组
    [self.contacts addObject:contact];
    
    // 展示到tableView上
    [self.tableView reloadData];
}
```

- 来源控制器(接收方)
- ZJAddViewController

```objc
2. 传递方先定义代理和代理协议
@protocol ZJAddViewControllerDelegate <NSObject>

@optional
// 在添加界面点击添加按钮的时候,通知代理接受联系人模型
- (void)addViewController:(ZJAddViewController *)addVc didAddContact:(ZJContact *)contact;
@end

@interface ZJAddViewController : UIViewController
@property (nonatomic, weak) id<ZJAddViewControllerDelegate> delegate;
@end






// 点击添加按钮的时候调用
- (IBAction)addContact:(id)sender {
    
    // 把姓名和电话文本框的内容包装成模型
    ZJContact *contact = [ZJContact contactWithName:self.nameField.text phone:self.phoneField.text];
    
    4. 通知代理接收联系人模型
    if ([_delegate respondsToSelector:@selector(addViewController:didAddContact:)]) {
        [_delegate addViewController:self didAddContact:contact];
    }
    
    // 回到上一个控制器
    [self.navigationController popViewControllerAnimated:YES];
}

```

---

#UITabBarController

---

- UITabBarController(49)的使用步骤
    - 初始化UITabBarController
    - 设置UIWindow的rootViewController为UITabBarController
    - 根据具体情况，通过addChildViewController方法添加对应个数的子控制器

```objc
// 启动完成的时候调用
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    // 1.创建窗口
    self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];

    // 2.设置窗口的根控制器
    // 2.1创建UITabBarController的根控制器
    UITabBarController *tab = [[UITabBarController alloc] init];
    // 2.2.设置根控制器
    self.window.rootViewController = tab;

    // 3.创建子控制器
    UIViewController *vc = [[UIViewController alloc] init];

    vc.view.backgroundColor = [UIColor redColor];
    vc.tabBarItem.title = @"消息";
    vc.tabBarItem.image = [UIImage imageNamed:@"tab_recent_nor"];

    // 3.1添加子控制器
    [tab addChildViewController:vc];



    // 4.显示窗口
    [self.window makeKeyAndVisible];


    return YES;
}
```

###UITabBarController的子控制器
UITabBarController添加控制器的方式有2种
- 添加单个子控制器

```objc
- (void)addChildViewController:(UIViewController *)childController;
// UITabBarController默认显示第0个控制器的view
```
- 设置子控制器数组

```objc
@property(nonatomic,copy) NSArray *viewControllers;
```

###UITabBarButton
- UITabBarButton里面显示什么内容，由对应子控制器的tabBarItem属性决定

- UITabBarItem有以下属性影响着UITabBarButton的内容

```objc
- 标题文字
@property(nonatomic,copy) NSString *title;

- 图标
@property(nonatomic,retain) UIImage *image;

- 选中时的图标
@property(nonatomic,retain) UIImage *selectedImage;

- 提醒数字
@property(nonatomic,copy) NSString *badgeValue;

```


#主流框架搭建
- 先搭建UITabBarController,再添加导航控制器为UITabBarController的子控制器
- Hide Bottom Bar On Push : 跳转时隐藏底部控制器,且只能隐藏系统的TabBar