---
layout: post
title: "Swift入门-swift中代理的使用"
date: 2014-11-14
comments: false
categories: Swift
---


我们很久以前就讲过代理方法,不过以前讲的都是OC中的代理方法,现在我们就介绍一下Swift中关于代理的写法,其实实现思路是一样一样的,只是具体细节还请大家一起来看一下

 
1.首先定义一份协议。

```
//声明一个协议, 让其继承(我也不知道该不该叫继承, 然而在这里并不重要) NSObjectProtocol, 只有这样才能在设置代理的时候前面添加weak
protocol ToolProrocol: NSObjectProtocol {
   //代理方法
   func didRecieveResults(result:Int)
}
```

2.定义一个代理属性

```
//声明代理属性
//注意这里: changeColor为协议名, delegate前面必须有weak修饰, 如果没有weak修饰就会造成内存泄露, 而可以加weak的前提是, 这个协议必须继承 NSObjectProtocol
weak var delegate : ToolProrocol?
```

3.使用者，首先遵守代理协议

```
class ViewController: UIViewController,ToolProrocol
```

4.并且设置代理和实现

```
  xxx.delegate = self
  
  func didRecieveResults(result: Int) {
    }
```

5.最后直接调用就ok了

```
self.delegate?.didRecieveResults(1)
```