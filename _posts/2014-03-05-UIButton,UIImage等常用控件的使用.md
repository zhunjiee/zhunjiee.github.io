---
layout: post
title: "UIButton,UIImage等常用控件的使用"
date: 2014-03-05
comments: false
categories: UI
---

#UILabel
- text 设置label显示的文本
- font 字体大小  系统自带样式 / 粗体样式 / 斜体样式

```objc
label.font = [UIFont systemFontOfSize:25]; // 系统样式
label.font = [UIFont boldSystemFontOfSize:25]; // 粗体样式
label.font =[UIFont italicSystemFontOfSize:25]; // 斜体样式
```
- textColor 字体颜色
- backgroundColor 背景颜色
- lineBreakMode 内容缩进模式
- numOfLines 0 :自动换行
- textAligment : 设置文字位置(NSTextAlignmentCenter[居中])

---

#UIButton
- (void)**setTitle**:(NSString \*)title **forState**:(UIControlState)state;
    + 根据状态设置按钮内容

- (void)**setTitleColor**:(UIColor \*)color **forState**:(UIControlState)state;
    + 根据状态设置按钮颜色

- (void)**setImage**:(UIImage *)image forState:(UIControlState)state;
- (void)**setBackgroundImage**:(UIImage *)image forState:(UIControlState)state;

---
##监听按钮点击用到的UIControl方法

- \- (void)**addTarget**:(nullable id)target **action**:(SEL)action **forControlEvents**:(UIControlEvents)controlEvents;

```objc
[butttn addTarget:self action:@selector(btnClick) forControlEvents:UIControlEventTouchUpInside];

```


---
# UIImageView

### contentMode属性

```objc
imgView.contentMode = UIViewContentModeScaleToFill;
```
**用于设置图片在图片框中的状态是拉伸,平铺...等等**

- 带有scale单词的：图片有可能会拉伸
    - **UIViewContentModeScaleToFill**
        - 将图片拉伸至填充整个imageView
        - 图片显示的尺寸跟imageView的尺寸是一样的
    - 带有aspect单词的：保持图片原来的宽高比
        - **UIViewContentModeScaleAspectFit**
            - 保证刚好能看到图片的全部
        - **UIViewContentModeScaleAspectFill**
            - 拉伸至图片的宽度或者高度跟imageView一样

- 没有scale单词的：图片绝对不会被拉伸，保持图片的原尺寸
    - UIViewContentModeCenter
    - UIViewContentModeTop
    - UIViewContentModeBottom
    - UIViewContentModeLeft
    - UIViewContentModeRight
    - UIViewContentModeTopLeft
    - UIViewContentModeTopRight
    - UIViewContentModeBottomLeft
    - UIViewContentModeBottomRight


## initWithImage:方法
- 利用这个方法创建出来的imageView的尺寸和传入的图片尺寸一样

```objc
UIImageView *imgView = [[UIImageView alloc] initWithImage:[UIImage imageNameed:@"1.jpg"]];
```
## 修改frame的3种方式
- 直接使用CGRectMake函数

```objc
imageView.frame = CGRectMake(100, 100, 200, 200);
```
- 利用临时结构体变量

```objc
CGRect tempFrame = imageView.frame;
tempFrame.origin.x = 100;
tempFrame.origin.y = 100;
tempFrame.size.width = 200;
tempFrame.size.height = 200;
imageView.frame = tempFrame;
```

- 使用大括号{}形式

```objc
imageView.frame = (CGRect){
			{100, 100}, 
			{200, 200}
		};
```

- 抽取重复代码
    - 将相同代码放到一个新的方法中
    - 不用的东西就变成方法的参数

**图片的加载方式**

- 缓存
    
    ```objc
    UIImage *image = [UIImage imageNamed:@"图片名"];
    ```
       - 使用场合：图片比较小、使用频率较高
       - 建议把需要缓存的图片直接放到Images.xcassets
- 无缓存
    
    ```objc
    NSString *file = [[NSBundle mainBundle] pathForResource:@"图片名" ofType:@"图片的扩展名"];
    UIImage *image = [UIImage imageWithContentsOfFile:@"图片文件的全路径"];
    ```
	- 使用场合：图片比较大、使用频率较小
    - 不需要缓存的图片不能放在Images.xcassets
    - 放在Images.xcassets里面的图片，只能通过图片名去加载图片

- 延迟做一些事情

```objc
[abc performSelector:@selector(stand:) withObject:@"123" afterDelay:10];
// 10s后自动调用abc的stand:方法，并且传递@"123"参数
```

- 音频文件的简单播放

```objc
// 创建一个音频文件的URL(URL就是文件路径对象)
NSURL *url = [[NSBundle mainBundle] URLForResource:@"音频文件名" withExtension:@"音频文件的扩展名"];
// 创建播放器
self.player = [AVPlayer playerWithURL:url];
// 播放
[self.player play];
```

#动态创建UI控件

---

```objc
@interface ViewController ()
//1. 声明
@property (weak, nonatomic) UILabel *label;
@end


@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    //2. 创建lebel并设置相关属性
    UILabel *label1 = [[UILabel alloc] initWithFrame:CGRectMake(100, 100, 100, 100)];
    label1.text = @"123";
    label1.textAlignment = NSTextAlignmentCenter;
    label1.backgroundColor = [UIColor orangeColor];
    //3. 加入到父控件中
    [self.view addSubview:label1];
    //4. 等于
    self.label1 = label1;
}
@end
```

#UITextField
```objc
#pragma mark - UITextFieldDelegate

// 是否允许开始编辑
- (BOOL)textFieldShouldBeginEditing:(UITextField *)textField{
    return NO;
}

// 是否允许结束编辑
- (bool)textFieldShouldEndEditing:(UITextField *)textField{
    return NO;
}

//  是否允许用户输入文字
// 作用:拦截用户的输入,可以用在微博140个字!!!
- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string{
    return NO;
}
```

#UISwitch
- @property(nonatomic,getter=isOn) BOOL on;
    - 开关的状态

- \- (void)**setOn**:(BOOL)on **animated**:(BOOL)animated;
    - 设置开关的状态

```objc
if (self.switch.on == no){
    [self.switch setOn:YES animated:YES];
}

// 自动登录状态改变时调用
- (IBAction)autoLogChange:(UISwitch *)sender {
    if (sender.on == YES) {
        [self.remPwdSwitch setOn:YES animated:YES];
    }
}

// 记住密码状态改变时调用
- (IBAction)remPwdChange:(UISwitch *)sender {
    if (sender.on == NO) {
        self.autoLogSwitch.on = NO;
    }
}
```


###可以用文本框内文字的长度判断真假
```objc
- (void)textChange{
    // 文本框内有文字,长度不为0,代表"真"
    self.login.enabled = self.userName.text.length && self.passWord.text.length;
}
```