---
layout: post
title: "Ubuntu下安装redis"
date: 2013-10-15 10:02
comments: true
categories: [nosql]
---

#### 安装步骤：

-------------------------------------------------------------------

1. **安装并编译redis:**


		wget http://redis.googlecode.com/files/redis-2.2.13.tar.gz    # 下载压缩包
		tar -zxf redis-2.2.13.tar.gz    # 解压该压缩包
		cd redis-2.2.13
		make                              # 编译
		sudo make install               # 安装redis

> _由下图自动生成的代码可知：这时Redis 的可执行文件被放到了/usr/local/bin_

![无法显示图片](/images/posts/2013-10-15/redis1.png "生成代码")


2. **拷贝指定配置文件到指定目录：**

        cp redis.conf /etc/

3. **启动服务并验证：**

		redis-server /etc/redis.conf
4. **查看是否成功启动：**

		ps -ef | grep redis
5. **测试set方法和get方法：**

		$redis-cli
		redis> set foo bar
		OK
		redis> get foo
		"bar"
-------------------------------------------------------------------

### 其他设置：

#### 设置开机自动启动，关机自动关闭：
		sudo update-rc.d redis-server defaults

#### 初始化用户和日志路径：

第一次启动Redis前，建议为Redis单独建立一个用户，并新建data和日志文件夹

    sudo useradd redis
    sudo mkdir -p /var/lib/redis
    sudo mkdir -p /var/log/redis
    sudo chown redis.redis /var/lib/redis
    sudo chown redis.redis /var/log/redis
