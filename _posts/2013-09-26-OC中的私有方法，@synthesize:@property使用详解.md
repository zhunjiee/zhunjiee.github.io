---
layout: post
title: "OC中的私有方法"
date: 2013-09-26
comments: false
categories: ObjC
---

##1.OC中的私有变量
- 在类的实现(即.m文件)中也可以声明成员变量,但是因为在其他文件中通常都只是包含头文件而不会包含实现文件,所以在.m文件中声明的成员变量是@private的。在.m中定义的成员变量不能和它的头文件.h中的成员变量同名,在这期间使用@public等关键字也是徒劳的。

```
@implementation Dog
{
    @public
    int _age;
}
@end
```
---

##2.OC中的私有方法
- 私有方法：只有实现没有声明的方法
- 原则上：私有方法只能在本类中的方法的中才能调用。

    + >注意: OC中没有真正的私有方法

```objc
@interface Dog : NSObject

@end
@implementation Dog

- (void)eat
{
    NSLog(@"啃骨头");
}
@end
int main(int argc, const char * argv[]) {

    Dog *d = [Dog new];
    SEL s1 = @selector(eat);
    [d performSelector:s1];

    return 0;
}
```

---

#@synthesize基本概念

##1.什么是@synthesize
- @synthesize是编译器的指令
- 什么是编译器的指令 ？
    + 编译器指令就是用来告诉编译器要做什么！
- @synthesize会让编译器做什么呢？
    + @synthesize 用在实现文件中告诉编译器实现成员变量的的访问器(getter/setter)方法
    + 这样的好处是:免去我们手工书写getterr和setter方法繁琐的代码

##2.@synthesize基本使用
- 在@implementation中, 用来自动生成setter和getter的实现

```
用@synthesize age = _age;就可以代替
- (int)age{
	return _age;
}
- (void)setAge:(int)age{
	_age = age;
}
```

- @synthesize编写步骤

```
1.在@implementation和@end之间写上@synthesize
2.在@synthesize后面写上和@property中一样的属性名称, 这样@synthesize就会将@property生成的什么拷贝到@implementation中
3.由于getter/setter方法实现是要将传入的形参 给属性和获取属性的值,所以在@synthesize的属性后面写上要将传入的值赋值给谁和要返回哪个属性的值, 并用等号连接
```

- 以下写法会赋值给谁?

```
@interface Person : NSObject
{
    @public
    int _age;
    int _number;
}

@property int age;

@end

@implementation Person

@synthesize age = _number;

@end

int main(int argc, const char * argv[]) {

    Person *p = [Person new];
    [p setAge:30];
    NSLog(@"_number = %i, _age = %i", p->_number, p->_age);

    return 0;
}
```
---

##3.@synthesize注意点
- @synthesize age = _age;
    + setter和getter实现中会访问成员变量_age
    + 如果成员变量_age不存在，就会自动生成一个@private的成员变量_age
    
- @synthesize age;
    + setter和getter实现中会访问@synthesize后同名成员变量age
    + 如果成员变量age不存在，就会自动生成一个@private的成员变量age

- 多个属性可以通过一行@synthesize搞定,多个属性之间用逗号连接

```
@synthesize age = _age, number = _number, name  = _name;
```
---




#@property
---

## @property基本概念
- @property在`类`中的功能:会自动生成getter和setter方法的声明和实现,还会生成 `_成员属性`

- @peroperty在`分类`中的功能:只会生成getter和setter方法的声明

---

##1.什么是@property
- @property是编译器的指令
- 什么是编译器的指令 ？
    + 编译器指令就是用来告诉编译器要做什么！

- @property会让编译器做什么呢？
    + @property 用在声明文件中告诉编译器声明成员变量的的访问器(getter/setter)方法
    + 这样的好处是:免去我们手工书写getterr和setter方法繁琐的代码

---

##2.@property基本使用
- 在@inteface中,用来自动生成setter和getter的声明

```
用@property int age;就可以代替下面的两行
- (int)age;   // getter
- (void)setAge:(int)age;  // setter
```

- @property编写步骤

```
1.在@inteface和@end之间写上@property
2.在@property后面写上需要生成getter/setter方法声明的属性名称, 注意因为getter/setter方法名称中得属性不需要_, 所以@property后的属性也不需要_.并且@property和属性名称之间要用空格隔开
3.在@property和属性名字之间告诉需要生成的属性的数据类型, 注意两边都需要加上空格隔开
```
---
# @property增强

---


##1.@property增强
- 自从Xcode 4.x后，@property可以同时生成setter和getter的声明和实现

```
@interface Person : NSObject
{
    int _age;
}
@property int age;
@end
```
---

##2.@property增强注意点
- 默认情况下，setter和getter方法中的实现，会去访问下划线 _ 开头的成员变量。

```
@interface Person : NSObject
{
    @public
    int _age;
    int age;
}
@property int age;

@end

int main(int argc, const char * argv[]) {

    Person *p = [Person new];
    [p setAge:30];
    NSLog(@"age = %i, _age = %i", p->age, p->_age);

    return 0;
}
```

- 如果没有会自动生成一个_开头的成员变量,自动生成的成员变量是私有变量, 声明在.m中,在其它文件中无法查看,但当可以在本类中查看


- @property只会生成最简单的getter/setter方法,而不会进行数据判断

```
Person *p = [Person new];
[p setAge:-10];
NSLog(@"age = %i", [p age]);
```
- 如果需要对数据进行判断需要我们之间重写getter/setter方法
    + 若手动实现了setter方法，编译器就只会自动生成getter方法
    + 若手动实现了getter方法，编译器就只会自动生成setter方法
    + 若同时手动实现了setter和getter方法，编译器就不会自动生成不存在的成员变量

---

# @property修饰符

---

##1.@property修饰符
- 修饰是否生成getter方法的
    + readonly    只生成setter方法，不生成getter方法
    + readwrite  既生成getter 又生成setter方法（默认）

```
@property (readonly) int age;
```

- 指定所生成的方法的方法名称
    + getter=你定制的getter方法名称
    + setter=你定义的setter方法名称(注意setter方法必须要有 :)

```
@property (getter=isMarried)  BOOL  married;
说明，通常BOOL类型的属性的getter方法要以is开头
```

---

## 总结：@property的使用策略
- assign
    - `基本数据类型`、`枚举`、`结构体`等非OC对象类型
- weak
    - OC对象类型（比如NSArray、NSDate、NSNumber、模型类）
- strong
    - OC对象类型（比如NSArray、NSDate、NSNumber、模型类）
    - 一个对象只要有强指针引用着，就不会被销毁
- copy
    - 一般用在`NSString`、`block`类型上


---