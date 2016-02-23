---
layout: post
title: "Ubuntu下安装redis 及安全配置"
date: 2013-10-15 10:02
comments: true
categories: [Nosql, "Security"]
---

#### 安装步骤：

-------------------------------------------------------------------

1.**安装并编译redis:**

想安装redis，必须要有先决条件，使你安装更容易：

```sh
    sudo apt-get update
    sudo apt-get install build-essential
    sudo apt-get install tcl8.5
```

选择最新稳定版本：

``` sh
    wget http://download.redis.io/releases/redis-stable.tar.gz
    tar xzf redis-stable.tar.gz
    cd redis-stable
    make
    make test # 运行测试
    sudo make install

```


> _由下图自动生成的代码可知：这时Redis 的可执行文件被放到了/usr/local/bin_

![无法显示图片](/images/posts/2013-10-15/redis1.png "生成代码")


2.** 让redis成为后台驻留程序： **

``` sh
    cd utils
    sudo ./install_server.sh
```

3.** 启动服务并验证： **

6379是redis的默认端口

``` sh
    sudo service redis_6379 start
    sudo service redis_6379 stop
```

4.** 查看是否成功启动： **

``` sh
		ps -ef | grep redis
```
5.** 测试set方法和get方法： **

``` sh
    $redis-cli
    redis> set foo bar
    OK
    redis> get foo
    "bar"
```

<br/>
<br/>

### 其他设置：

-------------------------------------------------------------------

#### 设置开机自动启动，关机自动关闭：

``` sh
    sudo update-rc.d redis_6379 defaults
```

#### 初始化用户和日志路径：

第一次启动Redis前，建议为Redis单独建立一个用户，并新建data和日志文件夹

``` sh
    sudo useradd redis
    sudo mkdir -p /var/lib/redis
    sudo mkdir -p /var/log/redis
    sudo chown redis.redis /var/lib/redis
    sudo chown redis.redis /var/log/redis
```

<br/>
<br/>


### Redis的安全配置：

------------------------------------------------------------------

这里不得不说一下，如果你在服务器上搭建redis，一定要对redis进行安全设置，因为我部署的网站，就是因为没有安全配置而被攻击。

1.** 绑定本机 IP **

默认redis服务器只会让localhost访问，如果你更新了配置文件让其他地方也可以连入redis，那就非常危险，你需要：

``` sh
    sudo vim /etc/redis/redis.conf
    bind 127.0.0.1 # 将此行解除注释，如果没有此行，那就加上这一行
```

<br/>

2.** 配置Redis密码 **
编辑redis的配置文件： redis.conf
将下面这一行取消注释，开启密码验证功能

``` sh
    # requirepass foobared
```

foobared太简单了，我们需要复杂的密码；下面提供一个命令可以生成密码：

``` sh
    echo "foo" | sha256sum
```

他会输出如下密码：

``` sh
    b5bb9d8014a0f9b1d61e21e796d78dccdf1352f23cd32812f4850b878ae4944c
```

黏贴到requirepass后面即可。如果这个命令没有用，你可能需要安装** apg ** 或者 ** pwgen **工具来帮助你生成密码。

更新完密码，请重启redis：
``` sh
    sudo service redis-server restart
```

进入redis，然后输入密码验证操作：

``` sh
    redis-cli    
    auth your_redis_password 
```

<br/>

3.** 重新命名命令**

redis允许你重新对命令进行命名或禁用；
其中有些命令比较危险，例如：**  FLUSHDB, FLUSHALL, KEYS, PEXPIRE, DEL, CONFIG, SHUTDOWN, BGREWRITEAOF, BGSAVE, SAVE, SPOP, SREM, RENAME 和 DEBUG**

你可以自己选择要禁用还是重命名：

禁用：

``` sh
    # It is also possible to completely kill a command by renaming it into
    # an empty string:
    #
    rename-command FLUSHDB ""
    rename-command FLUSHALL ""
    rename-command DEBUG ""
```

重命名：

``` sh
    rename-command CONFIG ""
    rename-command SHUTDOWN SHUTDOWN_MENOT
    rename-command CONFIG ASC12_CONFIG
```

保存后，重启redis

``` sh
    sudo service redis-server restart
    redis-cli
```
如果你进行了第二步设置密码，这里你需要输入密码，
然后输入config命令看看是否有效：

``` sh
    > config get requirepass
```

将会输出：

``` sh
    (error) ERR unknown command 'config'
```

输入重命名后的命令：

``` sh
    asc12_config get requirepass
```

输出：

``` sh
    1) "requirepass"
    2) "your_redis_password"
```

<br/>

4.** 设置文件目录的访问权限 **

``` sh
    ls -l /var/lib | grep redis
```

输出：

``` sh
drwxr-xr-x 2 redis   redis   4096 Aug  6 09:32 redis
```

看到redis数据目录的权限属于redis所有，这非常好，但是那部分不是文件夹的权限，文件夹的是755，我们需要把它设置成770

``` sh
    sudo chmod 700 /var/lib/redis
```

接下来我们来看看配置文件redis.conf

``` sh
    ls -l /etc/redis/redis.conf
    -rw-r--r-- 1 root root 30176 Jan 14  2014 /etc/redis/redis.conf
```
他是644，我们需要做如下修改：

``` sh
    sudo chown redis:root /etc/redis/redis.conf
    sudo chmod 600 /etc/redis/redis.conf
```

最后重启redis服务：

``` sh
    sudo service redis-server restart
```
