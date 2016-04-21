---
layout: post
title: "关于[self class]和[super class]的解释"
date: 2014-04-12
comments: false
categories: 知识囊
---

经常会看到这样的面试题:
<pre>
有两个类:Person类 和 Student类,其中Student类继承自Person类,Person类继承自NSObject,
在Student类中有如下代码:
	NSLog("%@", [self class]);
	NSLog("%@", [super class]);
	NSLog("%@", [self superclass]);
	NSLog("%@", [super superclass]);
	
请给出打印结果.
</pre>

结果是:

```
Student
Student 
Person 
Person
```

这里疑问主要集中在为什么self和super打印出来的类是一样的?

- 先说一下class和superclass的作用:
	- class的作用: 是 获取方法调用者 的 真实类型
	- superclass的作用: 获取方法调用者 的 父类类型


代码	`NSLog("%@", [self class]);`打印结果是Student这个应该没有疑问,因为就是自己调用的class方法,所以打印出来的自己的真实类型就是Student;

而我们纠结的就是`NSLog("%@", [super class]);`的打印结果也是Student,这是为什么呢?

这里我再重复一遍class的作用:获取**方法调用者** 的 真实类型,对,就是方法调用者,`[super class]`是我们在Student类中通过super关键字调用class方法打印出**方法调用者**的真实类型,其实际调用者还是Student.

这么说可能有点绕,但你只要记住super是是预编译指令,并不是代表父类,而是通过super关键字能找到父类.

对,其实就是这么简单,千万不要被super的表面意思蒙蔽了双眼,要理解其本质,还是很简单的.

---
下面给出self和super的底层实现,也可以不用看.


###self和super底层实现原理：

当使用 self 调用方法时，会从当前类的方法列表中开始找，如果没有，就从父类中再找；而当使用 super 时，则从父类的方法列表中开始找，然后调用父类的这个方法。
    
当使用 self 调用时，会使用 objc_msgSend 函数： id objc_msgSend(id theReceiver, SEL theSelector, ...)。第 一个参数是消息接收者，第二个参数是调用的具体类方法的 selector，后面是 selector 方法的可变参数。以 [self setName:] 为例，编译器会替换成调用 objc_msgSend 的函数调用，其中 theReceiver 是 self，theSelector 是 @selector(setName:)，这个 selector 是从当前 self 的 class 的方法列表开始找的 setName，当找到后把对应的 selector 传递过去。
    
当使用 super 调用时，会使用 objc_msgSendSuper 函数：id objc_msgSendSuper(struct objc_super *super, SEL op, ...)第一个参数是个objc_super的结构体，第二个参数还是类似上面的类方法的selector，

```
struct objc_super {
      id receiver;
      Class superClass;
};
```

当编译器遇到  [super setName:] 时，开始做这几个事：

1）构 建 objc_super 的结构体，此时这个结构体的第一个成员变量 receiver 就是 子类，和 self 相同。而第二个成员变量 superClass 就是指父类
调用 objc_msgSendSuper 的方法，将这个结构体和 setName 的 sel 传递过去。

2）函数里面在做的事情类似这样：从 objc_super 结构体指向的 superClass 的方法列表开始找 setName 的 selector，找到后再以 objc_super->receiver 去调用这个 selector