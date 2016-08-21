---
layout: post
title: "Scroll to top"
date: 2013-08-30 21:57
comments: true
categories: [Javascript]
---

许多网页都有提供一个便捷功能：当你浏览网页时，由于网页内容比较多，需要向下滑动；但是，当你要向上时，自己滑动太慢了。这里将的是一个自动向上滑的按钮。
具体操作如下：

1. 在页面header下写一个\<a\>标签里面放张图片指着向上。对\<a\>标签的css设置有几个重要的：position: fixed（表明标签会随着页面处在同一位置）,overflow: hidden

2. 接着设置js

> 1. 设置它在下拉后显示，具体是看window的scrollTop的位置,如果大于0，则已经开始下拉，所以让\<a\>标签显示。否则，隐藏。
> 2. 单击事件，滚到最上层。具体操作：最好用Jquery，设置时间让其向上滑动。如：	$('html, body').animate({scrollTop: 0}, 300)

**example:** (标签\<a class="go_top"\>)

``` js
    $(document).ready ->
        $("a.go_top").click () ->
            $('html, body').animate({scrollTop: 0}, 300)
            return false

        $(window).bind 'scroll resize', ->
            scroll_from_top = $(window).scrollTop()
            if scroll_from_top >= 1
                $('a.go_top').show()
            else
                $('a.go_top').hide()
```
