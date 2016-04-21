---
layout: post
title: "Swift入门-类型转换与is、as操作"
date: 2014-11-13
comments: false
categories: Swift
---



##三种操作：is、as？和as！



Swift是强类型语言，但也允许开发者通过is、as？和as！这三种操作来对类型进行判断和强制转换。



其中is用作**类型判断**，而as？和as！则分别是**类型转换**的**可选形式**和**强制形式**。在这里强调一下，swift中比较常用的向下转换（downcast）是没有as操作符的。



为了方便后面的解释，这里假设定义了三个类，Fruit、Apple和Orange，其中Apple和Orange都继承自Fruit。



##is操作符



is操作用来判断某一个对象是否是某一个特定的类，它会返回一个bool类型的值。is操作的逻辑很简单，某一个类的对象肯定是自己这个类，也一定是自己的超类，但超类的对象不是子类。如果两个类没有继承关系，那is操作一定返回false.



下面用代码举几个例子看一下：



```

if apple is Apple{

    println("这是个苹果")  //某个类的对象一定是自己这个类

}



if apple is Fruit{

    println("这是个苹果")  //某个类的对象也一定是它的超类

}



if fruit is Apple{

    println("这是个水果")  //但超类的对象不是它的子类

}



if apple is Orange{

    println("这是个橘子")  //没有继承关系的类肯定返回false

}

```



需要注意的是，这里所说的一个对象的类，是指它真实的类而不是表现出来的类。比如把apple1和orange1放进数组，再取出来，由于数组的类型是[Fruit]，所以通过下面这行代码：



`var fruit = array[0]`



取出的fruit，虽然在swift中的表现形式为Fruit，但它实际上是Apple类的对象，在对它使用is操作的时候，要把它当做Apple类的对象来考虑。因此，虽然超类的对象不是子类，但fruit is Apple的返回值却应该是true。



##as！操作符



as！操作符是类型转换的强制格式，优点在于代码简单，如果可以转换，则会返回转换了格式的对象，如果无法转换就会抛出运行时错误。因此除非百分之百确定可以转换，否则不应该使用as！来进行强制类型转换。



和is操作符非常类似，类型转换的规则是，某个类的对象可以转换为自己这个类（这个其实是废话），子类可以向上转换为超类，但超类不能向下（downcast）转换为子类。除非某个子类的对象表现形式为超类，但实际是子类，这时可以使用as！进行向下转换（downcast）。



用代码来说明一下：



```

// 这里假设fakeFruit是一个表现为Fruit类的Apple类对象

var fruit1 = fakeFruit as! Apple  //这是as！的最常见用法，可以成功转换

var fruit2 = fakeFruit as! Fruit  //成功，子类可以向上转换为超类

var fruit3 = fakeFruit as! Orange //失败，没有继承关系的类不可以转换

var orange = realFruit as! Orange //失败，超类不可以强制转换为子类 

```



##as？操作符



as？和as！操作符的转换规则完全一样，但是as？会返回一个被转换类型的可选类型，需要我们解封。因此写法会略有不同，以刚刚的第一个例子来说：



```

if var fruit = fakeFruit as? Apple{

    //Do something with fruit

}

```



另外，由于是可选类型，即使转换失败也不会报错，所以比较推荐使用这种方式进行类型转换。