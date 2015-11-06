---
layout: post
title: "结构体成员变量,setter/getter方法详解"
date: 2013-09-17
comments: false
categories: ObjC
---


##1.结构体成员变量
```
设计一个”学生“类 1> 属性
* 姓名
* 生日
用结构体作为类的实例变量(生日)
```

```
#import <Foundation/Foundation.h> //定义生日的结构体
typedef struct{
    int year;
    int month;
    int day;
}MyDate;

@interface Person : NSObject
{
    @public
    NSString *_name;//定义姓名 MyDate _birthday;//定义生日
}
@end

@implementation Person
@end


int main(int argc, const char * argv[]) {
    Person *p = [Person new];
    p->_name = @"sb";

    //因为结构体已经初始化为0了,在次初始化就报错了,但是可以逐个赋值。
    //p->_birthday = {1989,12,3};
    p->_birthday.year = 2014;
    p->_birthday.month = 05;
    p->_birthday.day = 12;
    NSLog(@"%@的生日是:%d年%d月%d 日",p->_name,p->_birthday.year,p->_birthday.month,p->_birthday.day);

    //也可以整体赋值
    MyDate de={1993,11,11};
    p->_birthday = de;
    NSLog(@"%@的生日是:%d年%d月%d 日",p->_name,p->_birthday.year,p->_birthday.month,p->_birthday.day);
    return 0;
}

```

---


#getter/setter方法

##1.setter方法
- 作用:用来设置成员变量,可以在方法里面过滤掉一些不合理的值
- 命名规范:
	+ 必须是对象
	+ 返回值类型为void
    + 方法名必须以set开头，而且后面跟上成员变量名去掉”_” 首字母必须大写
    + 必须提供一个参数，参数类型必须与所对应的成员变量的类型一致
    + 形参名称和成员变量去掉下划线相同

- 举例：

    如：如果成员变量为int _age 那么与之对象seter方法为
    -(void) setAge: (int) age;

- setter方法的好处
        不让数据暴露在外,保证了数据的安全性
        对设置的数据进行过滤

##2.getter方法

- 作用：为调用者返回对象内部的成员变量的值

- 命名规范：
        必须是对象方法
        必须有返回值,返回值的类型和成员变量的类型一致
        方法名必须是成员变量去掉下划线
        一定是没有参数的

- 举例

```
如：如果成员变量为int _age 那么与之对象geter方法为

    (int) age;
```

##3.getter/setter示例:

```objc
.h文件
@interface Person : NSObject{
    int _age;
}
- (void)setAge:(int)age;
- (int)age;
@end


.m文件
@implementation Person
- (void)setAge:(int)age
{
    if (age < 0) {
        age = 10;
    }
    _age = age;
}

-(int)age
{
    return _age;
}
@end
```

- getter方法的优点：
    + 可以让我们在使用数据的拿到数据之前对数据进行加工；
    + 比如双十一活动，我们希望对全线商品的价格在原来的价格基础上打五折，那么我们只要去改成品类的价格的getter方法就可以了，让他返回的值为价格 * 0.5

##3.getter/setter方法注意

- 在实际的开发中,不一定set和get方法都会􏰀供,如果内部的成员变量比如学生的学号或计算出来的数据这样的数据只允许外界读取,但是不允许修改的情况,则通常只􏰀供get方法而不􏰀供set方法。

- 成员变量名的命名以下划线开头,get方法名不需要带下划线,

- 成员变量名使用下划线开头有两个好处
    + 与get方法的方法名区分开来
    + 可以和一些其他的局部变量区分开来,下划线开头的变量,通常都是类的成员变量。当我看到以下划线开头变量，那么他一定是成员变量

