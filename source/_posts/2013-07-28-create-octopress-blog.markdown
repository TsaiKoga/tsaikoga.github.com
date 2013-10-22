---
layout: post
title: "create octopress blog"
date: 2013-07-28 22:51
comments: true
categories: gem
---
####克隆octopress项目
    git clone git://github.com/imathis/octopress.git blog
    cd blog

查看Gemfile文件：
稍作修改，如：source http://ruby.taobao.org
    vim Gemfile

之后执行:
    bundle install

接着要做的事是查看rakefile文件的：
    vim Rakefile	# 里面有许多rake任务

我们执行其中的:
    rake install # 复制"theme/"中的"sass/"和"source/"文件夹到"blog/"文件夹下

**查看网页效果：**
    rake preview

他会生成public文件夹

--------------------------------------------------------------------

####创建仓库：名字要以用户名为头，区分大小写。\(username.github.io\)
本地执行:
    rake setup_github_pages

在github上新建一个octopress仓库，当要填入url时，不要填入".git"，这样生成的页面才在".com"上,把public生成好的文件变成版本推送到远程仓库github，之后在远程仓库上做开发。
说到在远程仓库做开发，就是将"开发分支source"覆盖原有的"master分支"。
    git checkout source				# 切到开发分支source上
    git push origin source		# 推送到远程仓库


####如果想在source分支下有_deploy文件夹(当你的_desploy文件夹没了)，只要
    git clone https://github.com/TsaiKoga/tsaikoga.github.com.git

重新克隆了项目下来，再改变文件夹tsaikoga.github.com/名字为_deploy/, 
修改_config.yml文件中的内容:一定要把url改成/tsaikoga.github.com，

执行：
    rake generate

再执行以下名令，**部署到远程仓库**：
    rake deploy

最后到网址刷新看看。
******************************** 
####附加功能：
注册一个google analytics帐号：
生成tracking_id:

####新建一个博客：
    rake new_post["Hello TsaiKoga"]

存放到了 source/_post 之中, 编辑_post文件，如果使用disqus，这里的comment必须填true，下面有原因。

####做评论，可以基于第三方插件：disqus
扯开一下话题：
> Disqus的主要目标是通过提供功能强大的第三评论系统，将当前不同网站的相对孤立、隔绝的评论系统，连接成具有社会化特性的大网，用户使用Disqus，在不同网站上评论，无需重复注册账号，只需使用Disqus账号或者*第三方平台账号*，即可方便的进行评论，且所有评论都会存储、保存在Disqus账号后台，方便随时查看、回顾。

可以看到source/_layouts/post.html文件中已经有这个插件的评论方法：
在_config.yml中加入你申请的disqus的short_name
