---
layout: post
title: "Rails Assets Pipeline"
date: 2013-10-26 10:10
comments: true
categories: [Rails]
---

本章我将从三个方面介绍Assets Pipeline:

1. Assets Pipeline的产生背景
2. Assets Pipeline的定义
3. Assets Pipeline的主要功能
4. Assets Pipeline的使用方法
5. 生产环境下的Assets Pipeline

--------------------------------------------

###背景：

在Rails项目中，只用**public/文件夹是公开读取**的，所以我们通常会把**javascript,css等静态资源放入**其中，好让浏览器直接读取。但是随着这些静态文件越来越多，如何管理他们成为一个问题，为了加快浏览器下载速度，我们要合并javascript和stylesheet文件，来减少浏览器请求次数，更进一步还会压缩文件来加速下载时间。

  后来,Rails3引进了Assets Pipeline的新功能，它可以让静态资源存放在不同目录下，rails帮你组合压缩，特别是一些javascript插件。没有Assets Pipeline之前，我们必须将javascript复制到public/中浏览器才可以读取。

> Yahoo!和google都有自己开源的压缩工具[YUI Compressor][1]和[Closure Compile][2]。

[1]:http://yui.github.io/yuicompressor/
[2]:https://developers.google.com/closure/compiler
--------------------------------------------

###什么是Assets Pipeline：

Assets Pipeline是能**使Javascript,CSS的assets（包括自己在assets文件夹中写的erb,sass和coffescript文件）连接、缩小和压缩的一个框架**。这个框架通过它的**中心库Sprokets(将资源预处理，压缩和缩小)**使所有rails开发者从中受益。

assets pipeline是默认启动的，如果想关闭也可以到config/application.rb文件中将如下代码放入：
		config.assets.enable = false

当然，我们也可以在创建项目时，就不产生assets:
		rails new appname --skip-sprockets

----------------------------------------------

###主要功能:

- **连接：(减少浏览器请求数)**


	它可以避免浏览器为了渲染页面而不得不发送过多的请求。网页浏览器限制了并行请求的数量， 所以更少的请求能让你的应用程序加载更快。

	Rails 默认将所有的 JavaScript 文件连结成一个主要的 .js 文件，和将所有的 CSS 文件连结成一个主要的 .css 文件。在生产环境中， Rails 给**每个文件名插入一个 MD5 指纹**,以便**文件被网页浏览器缓存**。你可以通过修改指纹使缓存无效，这在你修改文件后会自动发生。


- **缩小/压缩：(加快js,css及图片等静态资源下载速度)**

	对于 CSS 文件，是通过去除空格和注释来实现的。
	
	对于 JavsScript, 会有更多的复杂过程。你可以从选项中选择一套构件或者指定你自己的。

- **使用索引文件：(建立索引文件提高访问速度)**

	Sprockets 对一些特殊的用途会使用名为 index (使用相关扩展) 的文件，作为一系列相关文件的索引。

- **高级语言预编译(coffeescript -> javascript / scss -> css等)：**

	它能你使用更高级的语言来编写资源，然后预编译成实质的资源。默认支持的语言包括 CSS 的 Sass，JavaScript 的 CoffeeScript 和可用于所有资源的 ERB.

-------------------------------------------

###使用方法：

在 Rails 之前的版本里，所有的资源都放置在 public 的子目录下比如 images, javascript 和 stylesheets. 对于 asset pipeline, 这些资源现在被指定到 **app/assets 目录**。这个**目录下的所有文件都通过 Sprockets 中间件供应**，这个中间件通过引入 sprockets gem 使用。

####资源组织：

Pipeline assets 可以被放置到一个应用程序中这三个位置中的一个:

- app/assets 放置属于**应用程序的资源**，比如自选图像，JavaScript 文件和样式文件。
- lib/assets 用于不在应用程序**范围内的自有函式库**，或者那些**跨应用程序共通的函式**。
- vendor/assets 用于属于**外部实体的资源**，比如 JavaScript 插件和 CSS 框架的代码。

----------------------------------------

 a. 搜索路径：

	介绍搜索路径时，先来介绍**资源清单**：
	
	> - 资源清单一般是app/assets/stylesheets/application.css和app/assets/javascripts/application.js
	>
	> - assets pipeline**默认搜索路径将会在app/assets/images和三个资源路径app/assets/、lib/assets/、 vendor/assets下的所有javascripts, stylesheets子目录**，而索引的**文件将由文件清单给出**,如：//= require home 将是 app/assets/javascripts/home.js。
	>
	> - 下图就是一份JS资源清单：

	![无法显示图片](/images/posts/2013-10-26/resource_list.png "JS资源清单")
	
	刚才已经提到，那是默认的搜索路径，你也可以附加路径：在config/application.rb里添加路径到pipeline。例如：
	
		config.assets.paths << Rails.root.join("app", "assets", "flash")

	_注意：想在**资源清单外引用**的文件必须加载到**预编译列表**里，否则它们在**生产环境**将不可以用。_

 b. 使用索引文件:

	如果你了解数据库，就应该对索引很熟悉。如果一个字段**经常被查找**，你可以将它**建立索引**从而提高访问速度。

	Sprockets也采取类似方法，对一些**特殊的用途**会使用名为 index (使用相关扩展) 的文件。

	> 例如，如果你的**许多模块都要使用某个 jQuery 函式库**，这个函式库存放在 lib/assets/library_name。 lib/assets/library_name/index.js 会作为**这个库的所有文件的 manifest**. 这个文件可以按顺序包含一组需要使用的文件，或者一个简单的 require_tree 指令。

 c. 连接资源代码：

    **使用标签：**

    pipeline(在当前的环境上下文中没有被关闭)提供标签访问你的资源,这些文件通过Sprockets获得。

    Sprockets 也会搜寻在 config.assets.paths 指定的所有路径，这些路径包括常规的应用程序路径和任何被 Rails engines 添加进来的路径。

		<%= stylesheet_link_tag "application" %>
		<%= javascript_include_tag "application" %>
		<%= image_tag "rails.png" %>

    **CSS:**

    assets pipeline会**自动解析erb**，你可以在CSS增加扩展名.erb，这样资源中就可以**使用helpers**方法(如：asset\_path)。

    使用assets pipeline，**资源路径必须重写**并且sass-rails提供两个helpers方法：-url和-path。

    **JS:**

    使用erb扩展名可以使用asset\_path的helpers方法。你也可以在coffeescript增加扩展名erb。


 d. 资源清单和指令：

    **作用：**Sprockets 使用资源清单文件去**确认哪些资源要引入并供应**的。

    这些资源清单文件包含一些 **指令** — 告诉 Sprockets 哪些文件要被**按顺序引入**，然后将它们**连结成单个 CSS 或者 JavaScript 文件的指示**。根据这些指令， Sprockets 加载这些被指定的文件，如果有必要就对它们**进行加工**，接着将它们**连结成单个文件然后压缩**它们 ( 如果 **Rails.application.config.assets.compress** 为 true ). 

    由于只处理单个文件而不是多个， 浏览器可以发起更少的请求所以页面的加载时间将会大大的缩减。

    还记得清单内容吗？之前提到的资源清单。

    **构成：**

	  - JS资源清单:

    那些**以//=开头的注释就是指令**，而 **require 指令**是用于告诉 Sprockets **需要加入**的文件,也不用加扩展名，当资源清单是js文件，Sprockets会假设你加入的是一个.js文件。
 
	  **require_tree** 指令告诉 Sprockets **递归**地去包含在**指定目录下_所有_** 的 JavaScript 文件到**输出里**。 这些**路径必须在资源清单文件中有相关的指定**。 有也可以使用 require\_directory 指令，它会将在某个特定目录下所有的 JavaScript 文件包含进去，但不递归。

		//= require jquery
		//= require jquery_ujs
		//= require_tree .

	  - CSS资源清单：
	
	  用css的注释开头，同样用require引入文件，指令**require_tree**和JS中的一样（**加入当前目录所有stylesheets文件**），require_self将文件中的 CSS (如果有) 放置到 require_self 调用的准确位置。

		/* ...
		*= require_self
		*= require_tree .
		*/

    _注意：用到多个sass文件时，用**@import rule**替代Sprockets_指令，Sprockets 指令在 Sass 文件中定义的**变量和 mixins** 都只能在其**被定义的文档**中可用。_

 e. 预处理：

    文件的**扩展名**被用于判断某个资源文件要进入**哪个预处理过程**。

    **预处理**可以通过**添加其它的扩展**被加入，扩展的**处理顺序**按是**从右到左**。这应该用于需要按顺序处理的过程。编译好的 JavaScript 和 CSS 文件送回给浏览。

    > 例如：

    > app/assets/javascripts/projects.js.erb.coffee，那么现由 CoffeeScript 拦截器先处理。它不能解析 ERB 所以你会碰到问题。

-------------------------------------------------------------

####生产环境：

 - 预编译资源

   Rails 本身绑定了一个 **rake** 任何去编译资源资源清单和 pipeline 中的其它文件到**磁盘**里。

   编译后的资源都被写入到了在 config.assets.prefix 指定的位置里。默认情况下，是 pulibc/assets 目录。

   为了更**快速的预编译资源**，你可以在 config/application.rb 里将 **config.assets.initialize\_on\_precompile 设置为 false** 去**部分加载**你的应用程序。

   > 如果你设置 config.assets.initialize_on_precomile 为 false,在**开发模式**范围内会**忽略这个标记的值**. 改变这个标记也会影响到 engines. Engines 也可以**指定预编译资源.** 因为完整的环境还没有加载完, **engines (或者其他的 gems) 将不会被加载**, 这会引起**资源丢失**.

   所以，你可以通过以下命令生成已编译版本的资源文件:
		
       bundle exec rake assets:precompile

----------------------------------------------------------------





