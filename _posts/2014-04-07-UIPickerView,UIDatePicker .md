---
layout: post
title: "UIPickerView,UIDatePicker"
date: 2014-04-07
comments: false
categories: UI
---

#UIPickerView
---
##1. UIPickerViewDataSource

- 返回pickerView的组数

```objc
- (NSInteger)numberOfComponentsInPickerView:(UIPickerView *)pickerView{
    return self.foodsData.count;
}
```
- 返回每一组的行数

```objc
- (NSInteger)pickerView:(UIPickerView *)pickerView numberOfRowsInComponent:(NSInteger)component{
    NSArray *items = self.foodsData[component];
    return items.count;
}
```
##2. UIPickerViewDelegate
- 每一组每一行显示文字

```objc
- (NSString *)pickerView:(UIPickerView *)pickerView titleForRow:(NSInteger)row forComponent:(NSInteger)component{
    //1. 获取对应的数据
    NSArray *items = self.foodsData[component];
    return items[row];
}
```
- 返回控件长什么样子

```objc
- (UIView *)pickerView:(UIPickerView *)pickerView viewForRow:(NSInteger)row forComponent:(NSInteger)component reusingView:(UIView *)view{

    ZJFlagView *flagView = [ZJFlagView flagView];

    flagView.flag = self.flags[row];

    return flagView;
}
```
- 设置行高

```objc
- (CGFloat)pickerView:(UIPickerView *)pickerView rowHeightForComponent:(NSInteger)component{
    return 70;
}
```
- 设置被选择的行的数据

```objc
- (void)pickerView:(UIPickerView *)pickerView didSelectRow:(NSInteger)row inComponent:(NSInteger)component{
    ZJFlag *flag =  self.flags[row];
    self.text = flag.name;
}
```

- 随机显示数据

```objc
- (IBAction)randomFood {
    NSInteger column = self.foodsData.count;
    for (int i = 0; i < column; i++) {
        // 获取旧的选中行
        NSInteger oldRow = [self.pickerView selectedRowInComponent:i];

        NSArray *items = self.foodsData[i];
        NSInteger rowCount = items.count;

        NSInteger randomRow = oldRow;

        // 获取旧的随机数与新的随机数一样的话,再随机
        while (oldRow == randomRow) {
            randomRow = arc4random() % rowCount;
        }

        //更新被选择行的数据
        [self pickerView:nil didSelectRow:randomRow inComponent:i];
        [self.pickerView selectRow:randomRow inComponent:i animated:YES];
    }
}
@end
```
---

#UIDatePicker

```objc
#import "BirthdayTextField.h"

@implementation BirthdayTextField

- (void)setUp{
    // 1.创建日期选择控件
    UIDatePicker *datePicker = [[UIDatePicker alloc] init];
    // 2.设置日期格式
    datePicker.datePickerMode = UIDatePickerModeDate;
    // 3.设置日期地区
    datePicker.locale = [NSLocale localeWithLocaleIdentifier:@"zh"];

#pragma mark - 格式化输出日期
    NSDateFormatter *fmt = [[NSDateFormatter alloc] init];
    fmt.dateFormat = @"yyyy-MM-dd";
    NSDate *date = [fmt dateFromString:@"1990-1-1"];

    // 4.设置开始日期
    datePicker.date = date;

    // 监听用户输入
    [datePicker addTarget:self action:@selector(dateChange:) forControlEvents:UIControlEventValueChanged];

    // 自定义文本框键盘
    self.inputView = datePicker;
}

// xib
- (void)awakeFromNib{
    [self setUp];
}
// 代码
- (instancetype)initWithFrame:(CGRect)frame{
    if (self = [super initWithFrame:frame]) {
        [self setUp];
    }
    return self;
}

// 时间改变事件
- (void)dateChange:(UIDatePicker *)datePicker{
    NSDateFormatter *fmt = [[NSDateFormatter alloc] init];
    fmt.dateFormat = @"yyyy-MM-dd";
    NSString *dateString = [fmt stringFromDate:datePicker.date];
    self.text = dateString;
}

// 初始化文本框文字
- (void)setUpText{
    [self dateChange:(UIDatePicker *)self.inputView];
}

@end
```