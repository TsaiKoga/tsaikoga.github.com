---
layout: post
title: "前后端分离"
date: 2019-04-14 12:00
comments: true
categories: ['ProjectManagement']
---
### 目录
#### [一、模板引擎](#1)
#### [二、单页面应用 或 API](#2)
#### [三、前端 MVC 模式](#3)
#### [四、前端 MVVM 模式](#4)
#### [五、Nodejs 前后端分离](#5)



<h2 id="1"> 一、模板引擎</h2>

- ERB: 前端渲染引擎，可以将后端传来的数据渲染注入到前端 HTML 中，也可以帮它叫做 ModelAndView（与 MVVM 的View-Model有区别，View-Model 是简化 Controller，有逻辑，负责 View 和 Model 的交互，而这里只是 Model 数据与 View 强绑定而已）；
- JS: 除了实现一些前端效果，还需要用 ajax 发起对后台的请求；

<br>

我还记得大三的时候，接触 Rails 框架，那时候 Rails 已经推出了 Assets 功能，
所有 CSS / JS 放在 assets/ 目录中，
html.erb 放在 views/ 目录；

然后通过编译，前端样式 assets/\*.css 文件最终生成 application.css 放在了 public/ 目录中；
前端 JS assets/\*.coffee 文件最终生成 application.js 也放在 public/ 目录下；
而 views/\*.html.erb 文件编译后变成纯 html 调用；
目前 PHP 的 Laravel 框架也是如此；

<br>

**优点：**

形成的是带有动态数据的html，比较有利于SEO

**缺点：**

前后端内容实际上没有隔离；所以一般做 Rails 的人既要会前端，又要会后端；

前端需要搭建整个 Rails 或者 Laravel 环境；

<br>

<h2 id="2"> 二、单页面应用 或 APP</h2>

- 单页面应用，其实就是所有请求都是 Ajax，这样解决了前面提到的模板引擎问题；
- 开发 APP，前端开发 H5、IOS、Android 等，后端只需要开发 API 返回前端需要的 json 数据；

<br>

记得刚进入公司做后端 API 开发的时候，发现很多 vue 页面还是通过上诉模板方式与 Laravel 糅合在一起；
然后前端文件会存放在后端 Laravel 项目仓库中；
后来我们把他分割开，前端单独起一个项目仓库，然后服务器编译完再复制到后端 Laravel 项目下的 Public/目录中；
虽然还是没有完全分离开，但是前端已经不需要后端语法，直接就是访问后端 API 接口即可；

<br>

**优点：**

前端不需要懂后端的语法，不用学习像 ERB 这样的模板引擎语法，

只需要通过 ajax 请求后端，后端返回后对返回的 json 数据做处理；

**缺点：**

前端逻辑越来越重，越来越复杂，浏览器端 和 客户端 逻辑复杂；

不利于 SEO；

<br>

<h2 id="3"> 三、前端 MVC 模式</h2>
当初在大三的时候，在实验室使用 Extjs，那时候 Extjs 开始走 MVC 架构，
然后毕业答辩时提到前端 MVC，而老师竟然说前端没有 MVC ，后端才有 MVC，
这里就来讲讲什么是前端 MVC，

前端 MVC 和后端 MVC 类似，

- Model 用来存放数据对象；
- View 用来呈现给用户，一般由 HTML/CSS/JS 组成；
- Controller 用来从 View 获取事件和输入，并进行处理，相应更新后更新视图；

<br>

**优点：**

分工明确，可以开发并行；

**缺点：**

Controller 要自己写，大量操作 DOM，页面渲染速度降低；
操作冗余，代码难以维护；

<br>

<h2 id="4"> 四、前端 MVVM 模式：</h2>
ViewModel 代替了 Controller；

- Model 用来存放数据对象；
- View 用来呈现给用户，一般由 HTML/CSS/JS 组成；
- View-Model：简化的 Controller，为 View 提供处理好的数据，view 绑定 view-model，视图与数据模型强耦合。数据的变化实时反映在 view 上，不需要手动处理；

<br>

解决了 MVC 模式的痛点，Controller 大量逻辑被简化，不需要大量操作 DOM 了；
View 和 Model 没有直接联系，是通过 View-Model 进行交互的；

<br>

**优点：**

不需大量 DOM 操作，前端工作减少，逻辑清晰；

**缺点：**

不利于 SEO；

<br>

<h2 id="5"> 五、Nodejs 前后端分离：</h2>

我们公司已经使用了 Vue 作为前端，一个 MVVM 模式的框架，
但是，我们还是会经常遇到各种问题；

<br>

**例如：**

1. 页面设计逻辑例如循环构造呈现的 JSON 数据，要么得前端处理，要么后端处理，一般都是前端处理，前端所需的排序功能、筛选功能，
以及到了视图层的页面展现，也许都需要对接口所提供的数据进行二次处理。数据量一大便会浪费浏览器性能，也造成代码难以维护，很多时候无法通用；

2. 多端产品 APP 会遇到接受格式不一致，像是 H5 可以接受 BOOLEAN 字段，而 IOS 端只能通过 1 或 0 改变；

<br>

我们把原先的 MMVM 模式作为前端的 UI 层；
然后将 Nodejs 起的服务作为前端服务层，这个前端 Server 层需要部署在服务器上；

前端 UI 层：用来处理浏览器层的展现逻辑。通过 CSS 渲染样式，通过 JavaScript 添加交互功能；
前端 Server 层：处理路由、模板、数据获取、cookie 等，路由完全前端控制，所以无论是 单页面 还是 多页面，前端都可以自由调控；

<br>

**优点：**

解决第 1 点问题：后端可以只做自己的事情，数据直接成为 json 发送给前端，前端 Nodejs 处理 json 逻辑，
Node Server 层放在服务器上，不会造成浏览器资源浪费；

解决第 2 点问题：使用 Node Server 层可以支持多端开发需要的内容不一致问题，H5 能处理 BOOLEAN 值，IOS 无法处理的情况；

**缺点：**

前端所需要了解服务端编程，所学内容增加，是基于 Nodejs 的全栈开发；
