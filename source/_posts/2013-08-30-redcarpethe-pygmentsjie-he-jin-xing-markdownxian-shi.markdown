---
layout: post
title: "redcarpet和pygments结合进行markdown显示"
date: 2013-08-29 22:23
comments: true
categories: [gem]
---
## redcarpet
----------------------------------------------------
用于将内容转化成markdown形式进行显示，它提供许多配置，具体操作方式如下：

1.首先是在gemfile中加入redcarpet.

2.安装redcarpet

``` sh
	gem install redcarpet
```

3.以前可直接在要显示的view中加入如下代码:

``` html
    <%= Redcarpet::Markdown.new(render, :extension => {})%>
```

现在在辅助方法中写如下代码：

``` ru
    def markdown(text)
			require 'redcarpet'
			render = Redcarpet::Render::HTML
			options = {
				autolink: true,
				filter_html: true,
				fenced_code_blocks: true,
        no_intra_emphasis: true
			}
			markdown = Redcarpet::Markdown.new(render.new(hard_wrap: true), options)
			markdown.render(text).html_safe
		end
```

4.在view中加入显示markdown的代码:

``` html
    <%= raw(markdown.render(@post.content).html_safe) %>
```

## pygments
---------------------------------------------------

可以和redcarpet很好的结合，用于语法高亮设置。
具体代码如下：

``` ru
		def markdown(text)
				require 'redcarpet'
				render = HTMLwithPygments
				options = {
						autolink: true, 
						filter_html: true, 
						prettify: true,
						fenced_code_blocks: true,
						no_intra_emphasis: true
				}
				markdown = Redcarpet::Markdown.new(render.new(hard_wrap: true), options)
				markdown.render(text).html_safe
		end
		 
		class HTMLwithPygments < Redcarpet::Render::HTML
				require 'pygments'
				def block_code(code, language)
						Pygments.highlight(code, :lexer => language, :options => {:linenos => true})
				end
		end
```

> ###注意：
> 1. 辅助方法要引入文件redcarpet和pygments。
> 2. hard_wrap必须Redcarpet::Render::HTML实例化时进行配置。
> 3. 必须有fenced_code_blocks才能将代码块放入代码框中。

{% video http://media.railscasts.com/assets/episodes/videos/272-markdown-with-redcarpet.mp4 800 504 http://railscasts.com/static/episodes/posters/loading.png %}

参考：

1. [github上的redcarpet](https://github.com/vmg/redcarpet)

2. [github上的pygments](https://github.com/tmm1/pygments.rb)

3. [视频参考](http://railscasts.com/episodes/272-markdown-with-redcarpet?view=comments)

