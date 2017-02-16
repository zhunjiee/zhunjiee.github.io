---
layout: post
title: "jQuery复习笔记-HTML操作"
date: 2016-03-19
comments: false
categories: web前端
---

## jQuery HTML

### 获取/改变

- 三个简单实用的用于 DOM 操作的 jQuery 方法：
	+ **text()** - 设置或返回所选元素的文本内容
	+ **html()** - 设置或返回所选元素的内容（包括 HTML 标记）
	+ **val()** - 设置或返回表单字段的值


#### 获取/改变属性 - attr()
- jQuery **attr()** 方法用于获取属性值。

实例

```
$("button").click(function(){
  alert($("#w3s").attr("href"));
});
```


### 添加

- **append()** - 在被选元素的结尾插入内容
- **appendTo()**
- **prepend()** - 在被选元素的开头插入内容
- **prependTo()**
- **after()** - 在被选元素之后插入内容
- **insertAfter()**
- **before()** - 在被选元素之前插入内容
- **insertBefore()**




### 删除

- **remove()** - 删除被选元素（及其子元素）
- **empty()** - 从被选元素中删除子元素


### CSS 类
- **css()** - 设置或返回样式属性

- **addClass()** - 向被选元素添加一个或多个类
- **removeClass()** - 从被选元素删除一个或多个类
- **toggleClass()** - 对被选元素进行添加/删除类的切换操作


- width（）
- innerWidth()
- outerWidth() ： 获取第一个匹配元素外部宽度（默认包括补白和边框）。
此方法对可见和隐藏元素均有效。
    + outerWidth( true ) : 将边距margin计算在内


### offset() - 获取到屏幕视口的距离
- 获取匹配元素在当前视口的相对偏移。
- 返回的对象包含两个整型属性：top 和 left，以像素计。此方法只对可见元素有效。

```
var left = $('div').offset().left;
```

### position（）- 获取到父元素的距离

- 获取匹配元素相对父元素的偏移。
- 返回的对象包含两个整型属性：top 和 left。为精确计算结果，请在补白、边框和填充属性上使用像素单位。此方法只对可见元素有效。

### offsetParent()

- 返回第一个匹配元素用于定位的父节点。
- 这返回父元素中第一个其position设为relative或者absolute的元素。此方法仅对可见元素有效。

### e.pageX
- 鼠标相对于文档的左边缘的位置。

---

### 数组遍历 - each()

```
$("imgs").each(function(i, element){ // 参数一：下标    参数二：元素
   this.src = "test" + i + ".jpg";
 });
```

---