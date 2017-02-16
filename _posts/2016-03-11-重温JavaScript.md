---
layout: post
title: "重温JavaScript"
date: 2016-03-11
comments: false
categories: web前端
---

### 引入外部js文件

```
// 一般放在文件最后
<script type="text/javascript" src="/js/myScript.js"></script>
</body>
```

### 控制台打印
`console.log("");`

### 弹出框(alert)
`alert("提示的内容");`

### 输入框(prompt)
`prompt("text", "defaultText" );`

### 确认框(confirm)
```
var r=confirm("Press a button!");
if (r==true)
  {
  alert("You pressed OK!");
  }
else
  {
  alert("You pressed Cancel!");
  }
}
```

### 输出 - write

直接把 `<p>` 元素写到 HTML 文档输出中：

```
<script>
document.write("<p>My First JavaScript</p>");
</script>
```

### 变量
```
var x=2;
var y=3;
var z=x+y;
```

### 数据类型
**字符串**

```
var carname="Bill Gates";
var carname='Bill Gates';
```

**数字**

```
var x1=34.00;      //使用小数点来写
var x2=34;         //不使用小数点来写
```

**布尔**

布尔（逻辑）只能有两个值：true 或 false。

```
var x=true
var y=false
```
**数组**

```
var cars=new Array();
cars[0]="Audi";
cars[1]="BMW";
cars[2]="Volvo";
```
或者 (condensed array):

```
var cars=new Array("Audi","BMW","Volvo");
```
或者 (literal array):

```
var cars=["Audi","BMW","Volvo"];
```

**对象**

```
var person={firstname:"Bill", lastname:"Gates", id:5566};
```

```
var person={
firstname : "Bill",
lastname  : "Gates",
id        :  5566
};
```

对象属性有两种寻址方式：

- 实例

```
name=person.lastname;
name=person["lastname"];
```

**Undefined 和 Null**

Undefined 这个值表示变量不含有值。

可以通过将变量的值设置为 null 来清空变量。

- 实例

```
cars=null;
person=null;
```

**声明变量类型**

当您声明新变量时，可以使用关键词 "new" 来声明其类型：

```
var carname=new String;
var x=      new Number;
var y=      new Boolean;
var cars=   new Array;
var person= new Object;
```
