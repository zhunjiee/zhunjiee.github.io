---
layout: post
title: "UITableView,状态栏,索引条,cell的循环利用"
date: 2014-03-25
comments: false
categories: UI
---


#UITableView

## 如何让tableView展示数据
- 设置数据源对象

```objc
self.tableView.dataSource = self;
```

- 数据源对象要遵守协议

```objc
@interface ViewController () <UITableViewDataSource>

@end
```

- 实现数据源方法

```objc
// 多少组数据
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView;

// 每一组有多少行数据
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section;

// 每一行显示什么内容
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath;

// 每一组的头部
- (NSString *)tableView:(UITableView *)tableView titleForHeaderInSection:(NSInteger)section;

// 每一组的尾部
- (NSString *)tableView:(UITableView *)tableView titleForFooterInSection:(NSInteger)section
```
---
#UITableView代理方法
```objc

// 当选中一行的时候调用（点击）
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    NSLog(@"选中了:%zd", indexPath.row);
}


// 当取消选中一行的时候调用
- (void)tableView:(UITableView *)tableView didDeselectRowAtIndexPath:(NSIndexPath *)indexPath
{
    NSLog(@"取消选中了:%zd", indexPath.row);
}

// 在头部添加控件
- (UIView *)tableView:(UITableView *)tableView viewForHeaderInSection:(NSInteger)section
{
    return [UIButton buttonWithType:UIButtonTypeInfoDark];
}

// 在尾部添加控件
- (UIView *)tableView:(UITableView *)tableView viewForFooterInSection:(NSInteger)section
{
    return [[UISwitch alloc] init];
}

// 设置header的高度
- (CGFloat)tableView:(UITableView *)tableView heightForHeaderInSection:(NSInteger)section
{
    if (section == 0) return 20;
    if (section == 1) return 50;
}


//  返回每个cell的高度
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath
{
    if (indexPath.row % 2 == 0) {
        return 50;
    } else {
        return 100;
    }
}




```
## tableView的常见设置及方法
```objc
// 设置每一行cell的高度
self.tableView.rowHeight = 100;

// 设置每一组头部的高度
self.tableView.sectionHeaderHeight = 50;

// 设置每一组尾部的高度
self.tableView.sectionFooterHeight = 50;

// 设置分割线颜色
self.tableView.separatorColor = [UIColor redColor];
// 设置分割线样式
self.tableView.separatorStyle = UITableViewCellSeparatorStyleNone;

// 是否显示滚动条
self.leftTable.showsVerticalScrollIndicator = NO; // 垂直滚动条
﻿self.leftTable.showsHorizontalScrollIndicator = NO; // 水平滚动条

// 设置表头控件
self.tableView.tableHeaderView = [[UISwitch alloc] init];
// 设置表尾控件
self.tableView.tableFooterView = [UIButton buttonWithType:UIButtonTypeContactAdd];

// 设置右边索引文字的颜色
self.tableView.sectionIndexColor = [UIColor redColor];
// 设置右边索引文字的背景色
self.tableView.sectionIndexBackgroundColor = [UIColor blackColor];

// 返回每一个cell的高度
- (UITableViewCell *)cellForRowAtIndexPath:(NSIndexPath *)indexPath;

- (NSInteger)numberOfSections;
- (NSInteger)numberOfRowsInSection:(NSInteger)section;

- (CGRect)rectForSection:(NSInteger)section;                                    // includes header, footer and all rows
- (CGRect)rectForHeaderInSection:(NSInteger)section;
- (CGRect)rectForFooterInSection:(NSInteger)section;
- (CGRect)rectForRowAtIndexPath:(NSIndexPath *)indexPath;

- (NSIndexPath *)indexPathForRowAtPoint:(CGPoint)point;                         // returns nil if point is outside of any row in the table
- (NSIndexPath *)indexPathForCell:(UITableViewCell *)cell;                      // returns nil if cell is not visible
- (NSArray *)indexPathsForRowsInRect:(CGRect)rect;                              // returns nil if rect not valid


```
---
##状态栏的样式

```objc
- (UIStatusBarStyle)preferredStatusBarStyle{
    return UIStatusBarStyleLightContent;
}
```

##隐藏状态栏
```objc
// 隐藏状态栏
- (BOOL)prefersStatusBarHidden
{
    return YES;
}
```

##索引条
```objc
- (NSArray *)sectionIndexTitlesForTableView:(UITableView *)tableView
{
    return [self.carGroups valueForKeyPath:@"title"];
}
```


#tableViewCell的常见设置

---

```objc
// 设置右边的指示样式
cell.accessoryType = UITableViewCellAccessoryDisclosureIndicator;
//UITableViewCellAccessoryCheckmark 对号
//UITableViewCellAccessoryDetailButton 详情
//UITableViewCellAccessoryDetailDisclosureButton 详情 >
//UITableViewCellAccessoryDisclosureIndicator >

// 设置右边的指示控件
cell.accessoryView = [[UISwitch alloc] init];
// accessoryView优先级大于accessoryType

// 设置cell的选中样式
cell.selectionStyle = UITableViewCellSelectionStyleNone;
// backgroundView优先级 > backgroundColor

// 设置背景色
cell.backgroundColor = [UIColor redColor];

// 设置背景view
UIView *bg = [[UIView alloc] init];
bg.backgroundColor = [UIColor blueColor];
cell.backgroundView = bg;

// 设置选中的背景view
UIView *selectedBg = [[UIView alloc] init];
selectedBg.backgroundColor = [UIColor purpleColor];
cell.selectedBackgroundView = selectedBg;
```
###UITableViewCell的contentView
- contentView下默认有3个子视图
    + 其中2个是UILabel(通过UITableViewCell的textLabel和detailTextLabel属性访问)
    + 第3个是UIImageView(通过UITableViewCell的imageView属性访问)

- UITableViewCell还有一个UITableViewCellStyle属性，用于决定使用contentView的哪些子视图，以及这些子视图在contentView中的位置

```objc
UITableViewCell *cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleSubtitle reuseIdentifier:nil];
// 标题的下面会有副标题
```
![](https://dn-zhunjiee.qbox.me/Snip20150724_1.png)
#cell的循环利用
---

###传统的写法

```objc
/**
 *  每当有一个cell要进入视野范围内，就会调用一次
 */
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    static NSString *ID = @"wine";

    // 1.先去缓存池中查找可循环利用的cell
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:ID];

    // 2.如果缓存池中没有可循环利用的cell
    if (!cell) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:ID];
    }

    // 3.设置数据
    cell.textLabel.text = [NSString stringWithFormat:@"%zd行的数据", indexPath.row];

    return cell;
}
```
- xib的传统写法

```objc
if (!cell) {
    cell = [[[NSBundle mainBundle] loadNibNamed:NSStringFromClass([ZJWineCell class]) owner:nil options:nil]lastObject];
    }
```


###新的写法（注册cell）

```objc
NSString *ID = @"wine";

- (void)viewDidLoad {
    [super viewDidLoad];

    // 注册某个重用标识 对应的 Cell类型(UITableViewCell class类型)
    [self.tableView registerClass:[UITableViewCell class] forCellReuseIdentifier:ID];
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    // 1.先去缓存池中查找可循环利用的cell
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:ID];

    // 2.设置数据
    cell.textLabel.text = [NSString stringWithFormat:@"%zd行的数据", indexPath.row];

    return cell;
}
```
注:通过xib创建cell时的注册

```objc
 [self.tableView registerNib:[UINib nibWithNibName:NSStringFromClass([ZJTgCell class]) bundle:nil] forCellReuseIdentifier:ID];
```


##tableview有数据的时候才会显示分割线
```objc
    self.tableView.tableFooterView = [[UIView alloc] init];
```