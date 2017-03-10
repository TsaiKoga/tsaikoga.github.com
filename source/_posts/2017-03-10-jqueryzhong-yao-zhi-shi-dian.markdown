---
layout: post
title: "JQuery重要知识点"
date: 2017-03-10 15:14
comments: true
categories: [Javascript]
---

对于Jquery，用起来很像很简单；但是有些知识点经常会忘记。这里特别写一篇来巩固。

### 加载：

#### 关于 ready 和 onLoad 的区别
两个不同点：

1)
只有文档完全下载到本地，才会触发```window.onload```事件，这意味着所有事件对于Javascript都是可以操作的。

通过```$(document).ready()```注册的事件处理程序，则会在 **DOM** 完全就绪并可以使用时调用。这意味着
所有元素对于脚本都是可以访问的。但不意味所有关联文件（例如DOM的图片）都已下载完毕。

换句话说：当HTML都下载完成别解析成为DOM树后，代码才可以运行。

<br/>

2)
对于```onload```属性一次只能保存一个函数引用，所以不能再现有行为上添加新的行为。

对于```$(document).ready()```调用这方法每次都会向内部的行为队列添加新函数。当页面加载完成后，所有函数会被执行。按照注册他们的顺序依次执行。

<br/>

关于```$(document).ready()```结构，实际上是在基于```document```这个DOM元素构建成的JQuery对象上调用```ready()```方法。

实际上可写成：
```javascript
$(document).ready({
  ... ...
});
```
或写成：
```javascript
$(function(){
  ...
});
```

<br/>

------------------------------------------

### 事件：

#### 事件捕获

允许多个元素响应单击事件的策略

#### 事件冒泡

当事件发生时，会首先发送给最具体的元素，在这个元素获得响应机会后，事件会 **向上冒泡到更一般** 的元素。

<br/>

**如何解决事件冒泡副作用**

访问事件对象e
```javascript
function(e) {
  if (e.target == this) {
    ... ...
  }
}
```


关于```event.stopPropagation()```阻止所有DOM响应这个事件，但是不能阻止默认操作;
必须要用```event.preventDefault()```来在操作前终止事件。

<br/>

```javascript
$('switch').on('click', 'button', function(){ })
```
相当于
```javascript
$('switch').on('click', function(e){
  if(e.target == 'button'){ }
})
```

模仿用户操作行为的方法 ```trigger()``` 它提供了和```on()```方法一样的简写方法。
像```click()```等on动作不带参数会自动触发。

<br/>

--------------------------------------

### 属性值：

#### 值回调
把回调函数作为参数传递


#### HTML 属性和 DOM 属性的区别

**HTML属性值:** 页面标记中放在引号的值

**DOM属性值:** Javascript能存取的值，可以通过chrome的elements检查器查看properties

**不同点：** 有些DOM名称和HTML不同，例如：className;
有些DOM属性HTML是没有的，DOM的```attr()```就没发操作他们。

测试DOM属性而非HTML属性才可确保跨浏览器的一致性。如：```prop()```来获取DOM属性值，而非```attr()```

最大不同在操作表单值：DOM和HTML操作表单的名称很多不同，最好不用```prop()```和```attr()```，而用```val()```



----------------------------------------

### Ajax

#### 单向数据传输：
AHAH(async http and html)用load()

服务器返回的JSON是字符串，需要变成JS字面量；
```javascript
$.getJSON('.json', function(data){
  ...
});
```

**向页面注入脚本：**
```javascript
$.getScript()
```

**获取XML：**
```javascript
$.get('.xml', function(data) {
    ...  
})
```

#### 双向数据传递：

```javascript
get('a.php', args, function(){})
```

```javascript
post('a.php', args, function(){})
```

```javascript
load('a.php', args, function(){})
```
load在包含数据对象参数时，会使用post方法

ajaxStart()回调，ajax请求开始且尚未进行任何其他传输时 和 read() 一样只能由$(document)调用

ajaxStop()回调，ajax最后一次请求结束时调用，和 read() 一样只能由$(document)调用

ajaxError()回调，参数是一个XMLHttpRequest的引用。
