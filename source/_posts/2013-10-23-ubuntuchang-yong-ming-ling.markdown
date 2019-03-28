---
layout: post
title: "Ubuntu常用命令"
date: 2013-10-23 16:20
comments: true
categories: [Linux]
---

## 目录：

#### [1.先介绍关于文件和目录的命令](#1)

#### [2.再介绍用户的命令](#2)

#### [3.查看系统信息的命令](#3)

#### [4.关于软件包操作的命令](#4)

#### [5.常用命令](#5)
#####   [5.1. find命令](#5.1)
#####   [5.2. locat命令](#5.2)
#####   [5.3. grep命令](#5.3)
#####   [5.4. ack-grep命令](#5.4)
#####   [5.5. cat head tail命令](#5.5)
#####   [5.6. awk命令](#5.6)
#####   [5.7. sed命令](#5.7)
#####   [5.8. netstat命令](#5.8)
#####   [5.9. telnet命令](#5.9)
#####   [5.10. crontab命令](#5.10)



![无法显示图片](/images/posts/2013-10-23/ubuntu.jpg "ubuntu")

<h2 id="1"> 1.先介绍关于文件和目录的命令：</h2>

> ls 列出当前目录文件（不包括隐含文件）

> ls -a 列出当前目录文件（包括隐含文件）

> ls -l 列出当前目录下文件的详细信息

> cd .. 回当前目录的上一级目录

> cd - 回上一次所在的目录

> cd ~ 或 cd 回当前用户的宿主目录

> mkdir 目录名 创建一个目录

> rmdir 空目录名 删除一个空目录

> rm 文件名 文件名 删除一个文件或多个文件

> rm -rf 非空目录名 删除一个非空目录下的一切

> mv 路经/文件 /经/文件移动相对路经下的文件到绝对路经下

> mv 文件名 新名称 在当前目录下改名

<h2 id="2"> 2.再介绍用户的命令:</h2>

> useradd 创建一个新的用户

>	groupadd 组名 创建一个新的组

>	passwd 用户名 为用户创建密码

>	passwd -d用户名 删除用户密码也能登陆

>	passwd -S用户名 查询账号密码

>	usermod -l 新用户名 老用户名 为用户改名

>	userdel–r 用户名 删除用户一切

<h2 id="3"> 3.查看系统信息的命令：</h2>

> uname -a 查看内核版本

> cat /etc/issue 查看ubuntu版本

> lsusb 查看usb设备

> sudo ethtool eth0 查看网卡状态

> cat /proc/cpuinfo 查看cpu信息

> lshw 查看当前硬件信息

> sudo fdisk -l 查看磁盘信息

> df -h 查看硬盘剩余空间

> free -m 查看当前的内存使用情况

> ps -A 查看当前有哪些进程

<h2 id="4"> 4.关于软件包操作的命令：</h2>

> apt-cache search package 搜索包

>	apt-cache show package 获取包的相关信息，如说明、大小、版本等

>	sudo apt-get install package 安装包

>	sudo apt-get install package - - reinstall 重新安装包

>	sudo apt-get -f install 修复安装”-f = –fix-missing”

>	sudo apt-get remove package 删除包

>	sudo apt-get remove package - - purge 删除包，包括删除配置文件等

>	sudo apt-get update 更新源

>	sudo apt-get upgrade 更新已安装的包

>	sudo apt-get dist-upgrade 升级系统

>	sudo apt-get dselect-upgrade 使用 dselect 升级

>	apt-cache depends package 了解使用依赖

>	apt-cache rdepends package 是查看该包被哪些包依赖

>	sudo apt-get build-dep package 安装相关的编译环境

>	apt-get source package 下载该包的源代码

>	sudo apt-get clean && sudo apt-get autoclean 清理无用的包

>	sudo apt-get check 检查是否有损坏的依赖

>	清理所有软件缓存（即缓存在/var/cache/apt/archives目录里的deb包）

>	sudo apt-get clean

<h2 id="5"> 5.最后讲几个比较重要，在实际操作中提供方便的命令：</h2>

<h3 id="5.1"> find命令</h3>

  find是最常见和最强大的查找命令，你可以用它找到任何你想找的文件。

**使用格式如下：**

``` bash
	 $ find <指定目录> <指定条件> <指定动作>
```

- <指定目录>： 所要搜索的目录及其所有子目录。默认为当前目录。

- <指定条件>： 所要搜索的文件的特征。

- <指定动作>： 对搜索结果进行特定的处理。

**举例：**

``` bash
    $ find . -name 'my*' -ls		# 搜索当前目录中，所有文件名以my开头的文件，并显示它们的详细信息。
	$ find . -type f -mmin -10    # 搜索当前目录中，所有过去10分钟中更新过的普通文件。如果不加-type f参数，则搜索普通文件+特殊文件+目录。
```

<br>
-------------------------------------------------------------------------

<h3 id="5.2"> locate命令<h3>

locate命令其实是"find -name"的另一种写法，但是要比后者快得多，原因在于它不搜索具体目录，而是搜索一个数据库（/var/lib/locatedb），这个数据库中含有本地所有文件信息。Linux系统自动创建这个数据库，并且每天自动更新一次，所以使用locate命令查不到最新变动过的文件。为了避免这种情况，可以在使用locate之前，先使用updatedb命令，手动更新数据库。

例如，Docker 下载的镜像没有启动cron，启动后执行这个命令：

``` bash
sudo updatedb
```

<br>

-------------------------------------------------------------------------

<h3 id="5.3"> 5.3 grep命令</h3>

grep命令用于查找文件中的字符串。

**命令格式如下：**

``` bash
	grep [选项] 查找模式 [文件名1, 文件名2, ...]

	grep [选项] [-e 查找模式|-f 文件][文件名1, 文件名2, ...]
```

> 当然：最好用的参数

>	-E 将查找模式解释成正则表达式

>	-F 将查找模式解释成单纯的字符串

>	-i --ignore-case 匹配比较时不区分字母的大小

>	-R 以递归方式查询目录下所有子目录中的文件

<br>

-------------------------------------------------------------------------

<h3 id="5.4"> 5.4 ack-grep命令</h3>

grep命令加强。首先，我们先了解什么是ack？

>	http://betterthangrep.com

>	ack is a tool **like grep**, optimized for programmers

>	is written purely in Perl 5,takes advantage of the power of Perl's regular expressions.

ack诞生的目的就是要取代grep。

注意：ubuntu下要安装**ack-grep**，因为在debian系中，ack这个名字被其他的软件占用了。

``` bash
    sudo apt-get install ack-grep
```
<br>

-------------------------------------------------------------------------

<h3 id="5.5"> 5.5 cat head tail 输出命令</h3>

- cat 查看所有
- head -n 查看前几行
- tail -n 查看后几行
查看第500行，则是：

``` bash
head -n 500 xx.log | tail -n 1
```

<br>
-------------------------------------------------------------------------

<h3 id="5.6"> 5.6 awk 命令</h3>

AWK 命令可以对日志或其他文件内容进行自定义输出。
例如：查看 apache 日志中时间在 某个区间内的所有行；

``` bash
cat access.log | awk '{if($4>"[19/Mar/2019:09:45:00 +0800]" && $4<"[19/Mar/2019:09:55:00 +0800]")print $0'}
```

其中
$0 是一整行；
$1 是第一列，按着推，$2...$9是第二列到第九列；

表达式写在 '{}' 中，输出内容用 print;

<br>

-------------------------------------------------------------------------

<h3 id="5.7"> 5.7 sed 命令</h3>

SED 对文件中的内容进行添加删除操作；

``` bash
# 将文本插入文件中匹配的行：
sed '/<\/Host>/i\<Connector executor="tomcatThreadPool" po" URIEncoding="UTF-8" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />' /opt/appstack/apache-tomcat/conf/server.xml
```
第一个 // 中为匹配内容；
第二个 i 为插入；
第三个 i 后面为插入内容；
最后带上文件路径；

<br>
-------------------------------------------------------------------------

<h3 id="5.8"> 5.8 netstat命令</h3>

 Netstat是在内核中访问网络及相关信息的程序，它能提供TCP连接，TCP和UDP监听，进程内存管理的相关报告。

**命令格式如下：**

``` bash
    netstat -tulpn	      # 显示服务器进程
    sudo netstat -antup   # 查看进程
    ack-grep function     # 查找函数位置（必须在文件目录下）
```

<br>
-------------------------------------------------------------------------

<h3 id="5.9"> 5.9 telnet命令</h3>

因为 ping 只能用于 ip 或单个域名；
所以我们用 telnet 可以用来本地查看 远程服务器端口是否可连接；

``` bash
telnet www.baidu.com 8080
```

<br>

-------------------------------------------------------------------------

<h3 id="5.10"> 5.10 crontab命令</h3>

这个命令我非常喜欢，他是**设置定时执行脚本或任务**,可以用它来定时做一些事情。如定时周期性的备份网站数据库，并email发送到指定邮箱。

参数就几个很简单：

``` bash
    crontab -l # 显示现有任务条目
    crontab -r # 删除当前的任务
    crontab -e # 编辑任务单，一般使用 nano 编辑，如DH。
```

在 Docker 的 ubuntu 镜像中可能没有起动，

输入

``` bash
cron
```
就启动了进程 cron

**时间参数：**

> 15 \* \* \* \* : 每小时第15分执行1次

> 15,18 \* \* \* \* : 每小时第15和18分各执行1次

> \*/15 \*/2 \* \* \*: 每隔15分钟执行1次

> 15 20 \* \* 6: 每周星期6的20：15执行1次

<br>
-------------------------------------------------------------------------
