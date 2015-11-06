---
layout: post
title: "iOS之 atomic nonatomic区别"
date: 2014-06-23
comments: false
categories: 知识囊
---

atomic 和 nonatomic 用来决定编译器生成的getter和setter是否为原子操作。

- 脑补
	- 原子操作："原子操作(atomic operation)是不需要synchronized"，这是Java多线程编程的老生常谈了。所谓原子操作是指不会被线程调度机制打断的操作；这种操作一旦开始，就一直运行到结束，中间不会有任何 context switch （切换到另一个线程）。
	- synchronized['sɪŋkrənaɪzd] ：Java语言的关键字（iOS中我们在[互斥锁](http://www.zhunjiee.com/%E5%AE%9E%E7%94%A8%E6%8A%80%E6%9C%AF/2014/05/08/%E5%A4%9A%E7%BA%BF%E7%A8%8B%E4%B9%8B-NSThread.html)处讲到过)，可用来给对象和方法或者代码块加锁，当它锁定一个方法或者一个代码块的时候，同一时刻最多只有一个线程执行这段代码。
	
##atomic
- 设置成员变量的@property属性时，默认为atomic，提供多线程安全。

- 在多线程环境下，原子操作是必要的，否则有可能引起错误的结果。加了atomic，setter函数会变成下面这样：

```
{lock}
	if (property != newValue) { 
		[property release]; 
		property = [newValue retain]; 
	}
{unlock}
```

##nonatomiac        
###禁止多线程，变量保护，提高性能。

- **atomic是Objc使用的一种线程保护技术，基本上来讲，是防止在写未完成的时候被另外一个线程读取，造成数据错误。而这种机制是耗费系统资源的，所以在iPhone这种小型设备上，如果没有使用多线程间的通讯编程，那么nonatomic是一个非常好的选择。**

- 指出访问器不是原子操作，而默认地，访问器是原子操作。这也就是说，在多线程环境下，解析的访问器提供一个对属性的安全访问，从获取器得到的返回值或者通过设置器设置的值可以一次完成，即便是别的线程也正在对其进行访问。如果你不指定 nonatomic ，在自己管理内存的环境中，解析的访问器保留并自动释放返回的值，如果指定了 nonatomic ，那么访问器只是简单地返回这个值。（这个看不懂）

- 不保证setter/getter的原子性，多线程情况下数据可能会有问题。
- nonatomic，非原子性访问，不加同步，多线程并发访问会提高性能。先释放原先变量，再将新变量retain，然后赋值。