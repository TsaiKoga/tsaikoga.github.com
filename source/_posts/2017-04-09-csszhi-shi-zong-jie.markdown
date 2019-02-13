---
layout: post
title: "CSS知识总结"
date: 2017-04-09 10:27
comments: true
categories: [CSS]
---

### XHTML必要规则：
------------------------------
1. 标签都要小写
2. 属性名要小写
3. 严格的嵌套（例如：```<i><b>Hello</b></i> <!-->而不是<b><i>hello</i></b><-->```）
4. 标签必须封闭
5. 空元素标记也要封闭（例如：```<img/><br/>```）
6. 属性要用双引号
7. 属性使用完整形式（例如：```<button disabled=true></button><!-->而不是直接disabled<-->```）
8. 结构标记不能放在内容标记中（如```<table>不能放在<p>中```）

<br/>
<br/>

## CSS 总结
----------------------------
### 引入CSS
------------------------------
- 行内式：通过```<style type="text/css"></style>```直接引入
- 嵌入式：通过```<head>```中引入
- 链接式：通过```<link href=“style/style.css" rel="stylesheeet" type="txet/css"/>```引入
- 导入式：通过```<style type="text/css">@import "style.css";</style>```，注意：导入式是等到加载完后才导入css的，所以会有一瞬间没有css样式

<br/>

### 选择器
------------------------------
- 标记选择器：例如p、h1、 h2等标签去掉尖括号
- 类别选择器：.class
- ID选择器： #id
- 复合选择器：
1. 交集选择器：将两个选择器并列在一起，如h1.spec
2. 并集选择器：通过逗号分隔开，如h1, h2
3. 后代选择器：通过空格分隔，这里的后代可以是儿子，孙子等
4. 子选择器：通过>号分隔，与后代不同，它只适用于儿子。

优先级：行内样式 > ID样式 > 类别样式 > 标记样式

<br/>

### 盒子模型
------------------------------
margin - border -padding - content

<br/>
由于盒子模型的 border 和 padding，实际设置宽度应该减去这两个值，才能保证盒子的宽度不会变大；
还好 CSS3 引入box-sizing: border-box，这个属性可以让 padding 和 border 不对 width 造成影响。

```css
.fancy {
  width: 500px;
  margin: 20px auto;
  padding: 50px;
  border: solid blue 10px;
  -webkit-box-sizing: border-box;
     -moz-box-sizing: border-box;
          box-sizing: border-box;
}
```
<br/>

### 布局Postion
------------------------------
提到 postion，必须知道一件事，就是元素是否会被 postioned，因为 absolute 需要他来调整位置。
目前只有一个属性值 static 不会被positioned，其他都会。
<br/>
- static: 默认的 position 值，不会被positioned。
- relative: 通过 top / right / bottom / left 偏离原位置，**其他元素位置不会受到它偏离后的影响**
- fixed: 固定定位，脱离文档流；手机版本不支持，详情请看[解决方法](http://bradfrost.com/blog/mobile/fixed-position/)
- absolute: 对于最近的 **positioned 祖先** 绝对定位，像fixed一样通过 top / right / bottom  /left 来调整位置

<br/>
### 浮动
------------------------------
浮动主要是float,有left，right。他会对相邻元素造成影响。通过clear属性可以清除影响，如：
```css
.box {
  float: left;
  width: 200px;
  height: 100px;
  margin: 1em;
}
.after-box {
  clear: left;
}
```

如果 div 中元素（例如图片）设置了浮动，div 反而比他里面这个元素（图片）小，该怎么办？
通过设置为 ```overflow: auto``` 解决，IE6还要设置```zoom: 1```

<br/>
<br/>
### 响应式布局：
------------------------------
#### 设置 Meta 标签：
------------------------------
大多数移动浏览器将HTML页面放大为宽的视图（viewport）以符合屏幕分辨率。你可以使用视图的meta标签来进行重置。下面的视图标签告诉浏览器，使用设备的宽度作为视图宽度并禁止初始的缩放。在<head>标签里加入这个meta标签。
``` css
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
```
viewport是网页默认的宽度和高度，上面这行代码的意思是，网页宽度默认等于屏幕宽度（width=device-width），原始缩放比例（initial-scale=1）为1.0，即网页初始大小占屏幕面积的100%。
** 注意：（user-scalable = no 属性能够解决 iPad 切换横屏之后触摸才能回到具体尺寸的问题。 ）**

<br/>

#### 通过媒介查询来设置样式 Media Queries：
------------------------------
Media Queries 是响应式设计的核心。
它根据条件告诉浏览器如何为指定视图宽度渲染页面。假如一个终端的分辨率小于 980px，那么可以这样写：
``` css
@media screen and (max-width: 980px) {
}
```
<br/>

#### 设置多种视图宽度
------------------------------
假如我们要设定兼容 iPad 和 iphone 的视图，那么可以这样设置：
```css
/** iPad **/
@media only screen and (min-width: 768px) and (max-width: 1024px) {}
/** iPhone **/
@media only screen and (min-width: 320px) and (max-width: 767px) {}
```
<br/>

#### 图片文字也要响应式
------------------------------
1.宽度需要使用百分比
```css
#head { width: 100% }
```
<br/>
2.处理图片缩放的方法
简单的解决方法可以使用百分比，但这样不友好，会放大或者缩小图片。那么可以尝试给图片指定的最大宽度为百分比。假如图片超过了，就缩小。假如图片小了，就原尺寸输出。
```css
img { width: auto; max-width: 100%; }
```
<br/>
3.相对大小的文字：
字体也不能使用绝对大小（px），而只能使用相对大小（em）。
先指定全局字体，这里设置为页面默认大小的100%，即16像素：
```css
body {
　font: normal 100% Helvetica, Arial, sans-serif;
}
```
然后
```css
h1 {
　font-size: 1.5em;
}
```
h1的大小是默认大小的1.5倍，即24像素（24/16=1.5）。
<br/>

##### vw 和 vh

vw 和 vh 是相对视区宽度和高度，这里的 视区 指的是网页宽度高度，也就是 window.innerWidth 和 window.innerHeight
然后 vw 如果为 10 ，也就是该元素相对网页宽度是 1/10；


相关响应式资料资料：http://www.ruanyifeng.com/blog/2012/05/responsive_web_design.html
