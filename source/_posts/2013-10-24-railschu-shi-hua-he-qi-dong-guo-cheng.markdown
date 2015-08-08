---
layout: post
title: "Rails服务启动过程"
date: 2013-10-24 14:51
comments: true
categories: [Rails]
---

我们都知道，每当我们console中输入rails s时，系统就启动了你的rails应用程序。

-----------------------------------------------

###app程序中的bin/rails脚本

它是由你的app程序的**bin/rails脚本**执行的(我们的rails命令在通过bin/rails脚本引用rails/commands.rb文件中)。

_如：example\__app/bin/rails_
 
``` ru
    #!/usr/bin/env ruby
    APP_PATH = File.expand_path('../../config/application',  __FILE__)
    require_relative '../config/boot'
    require 'rails/commands'
```

APP\_PATH常量将会在rails/commands中用到，而config/boot指向**config/boot.rb**文件，这个文件**会加载和设置Bundler**。

-----------------------------------------------

###config/boot.rb加载gem

前面bin/rails引入两个文件,现在我们先说第一个文件
config/boot.rb:

``` ru
    # Set up gems listed in the Gemfile.
    ENV['BUNDLE_GEMFILE'] ||= File.expand_path('../../Gemfile', __FILE__)
     
    require 'bundler/setup' if File.exists?(ENV['BUNDLE_GEMFILE'])
```

config/boot.rb用ENV['BUNDLE\_GEMFILE']定位Gemfile文件，当Gemfile文件存在时，就将它引入。这样一来，gem就被引入了。

-------------------------------------------------

###rails/commands.rb执行rails命令

前面config/boot.rb引入的第二个文件,但这个文件必须在第一个文件config/boot.rb文件引入后才被引入。
rails/commands.rb:

``` ru
    ARGV << '--help' if ARGV.empty?

    aliases = {
        "g"  => "generate",
        "d"  => "destroy",
        "c"  => "console",
        "s"  => "server",
        "db" => "dbconsole",
        "r"  => "runner"
    }
     
    command = ARGV.shift
    command = aliases[command] || command
```

当你输入的是rails server的话，将会执行下列代码匹配你的命令：

``` ru
when 'server'
    # Change to the application's path if there is no config.ru file in current dir.
    # This allows us to run `rails server` from other directories, but still get
    # the main config.ru and properly set the tmp directory.
    Dir.chdir(File.expand_path('../../', APP_PATH)) unless File.exists?(File.expand_path("config.ru"))
 
    require 'rails/commands/server'
    Rails::Server.new.tap do |server|
        # We need to require application after the server sets environment,
        # otherwise the --environment option given to the server won't propagate.
        require APP_PATH
        Dir.chdir(Rails.application.root)
        server.start
    end
``` ru

刚才我们的bin/rails脚本中已经定义了常量APP\_PATH为"config/application.rb",而

``` ru
    Dir.chdir(File.expand\_path('../../', APP\_PATH)) unless File.exists?(File.expand\_path("config.ru"))
```
		
表示如果**没有config.ru文件时，就加载rails/commands/server文件，否则加载config/application.rb**。

也就是，你**启动了服务前才帮你加载程序的应用文件application.rb**。
application.rb这个文件将会加载Rails。

-----------------------------------------------------------------------

###rails/commands/server.rb

来看一下rails/commands/server中定义的类Rails::Server吧。

``` ru
    require 'fileutils'
    require 'optparse'
    require 'action_dispatch'
     
    module Rails
        class Server < ::Rack::Server
```

这里引入的**fileutils和optparse**是**标准的ruby库**，分别提供文件的辅助方法和转化选择。

Rails::Server继承了Rack::Server,当**Rack::Server被初始化(设置环境)**时，Rack::Server也被初始化。

``` ru
    def initialize(*)
        super
        set_environment
    end
```

-----------------------------------------------------------------------------

Rack::Server是为**以Rack为基础的应用提供公用的服务接口**，rails就是其中之一。

- 初始化设置环境：

	刚才提到的初始化代码如下：

``` ru
    def set_environment
        ENV["RAILS_ENV"] ||= options[:environment]
    end
```

	不要以为set_environment很少，options干了很多事情：

``` ru
    def options
        @options ||= parse_options(ARGV)
    end
```

	parse_options定义如下：

``` ru
    def parse_options(args)
        options = default_options
     
        # Don't evaluate CGI ISINDEX parameters.
        # http://hoohoo.ncsa.uiuc.edu/cgi/cl.html
        args.clear if ENV.include?("REQUEST_METHOD")
     
        options.merge! opt_parser.parse! args
        options[:config] = ::File.expand_path(options[:config])
        ENV["RACK_ENV"] = options[:environment]
        options
    end
```

	我们将一行一行解释，

	(1)选择default_options如下设置：

``` ru
    def default_options
        {
            :environment => ENV['RACK_ENV'] || "development",
            :pid         => nil,
            :Port        => 9292,
            :Host        => "0.0.0.0",
            :AccessLog   => [],
            :config      => "config.ru"
        }
    end
```

	它**(Rack::Server)提供了默认环境和端口号等信息**，还记得当rails/commands.rb中，如果存在config.ru时，就加载config/applicaton.rb文件吗？

	(2)当ENV哈希环境中没有REQUEST_METHOD这个键时，跳过而去执行合并选项。

``` ru
	Rack::Server中的opt_server

		def opt_parser
			Options.new
		end
```

	(3)Rails::Server重写parse!

``` ru
    def parse!(args)
        args, options = args.dup, {}
     
        opt_parser = OptionParser.new do |opts|
            opts.banner = "Usage: rails server [mongrel, thin, etc] [options]"
            opts.on("-p", "--port=port", Integer,
                            "Runs Rails on the specified port.", "Default: 3000") { |v| options[:Port] = v }
        ...
```

刚才说default\_options已经有了config.ru，这是回到之前rails.commands.rb文件，它加载了config/application.rb文件

------------------------------------------------------------------------

###config/application.rb

当require APP\_PATH被执行，**config/applicaton.rb被加载，server.start就被调用**了。
方法如下：

``` ru
    def start
        url = "#{options[:SSLEnable] ? 'https' : 'http'}://#{options[:Host]}:#{options[:Port]}"
        puts "=> Booting #{ActiveSupport::Inflector.demodulize(server)}"
        puts "=> Rails #{Rails.version} application starting in #{Rails.env} on #{url}"
        puts "=> Run `rails server -h` for more startup options"
        trap(:INT) { exit }
        puts "=> Ctrl-C to shutdown server" unless options[:daemonize]
     
        #Create required tmp directories if not found
        %w(cache pids sessions sockets).each do |dir_to_make|
            FileUtils.mkdir_p(Rails.root.join('tmp', dir_to_make))
        end
     
        unless options[:daemonize]
            wrapped_app # touch the app so the logger is set up
     
            console = ActiveSupport::Logger.new($stdout)
            console.formatter = Rails.logger.formatter
     
            Rails.logger.extend(ActiveSupport::Logger.broadcast(console))
        end
     
        super
    ensure
        # The '-h' option calls exit before @options is set.
        # If we call 'options' with it unset, we get double help banners.
        puts 'Exiting' unless @options && options[:daemonize]
    end
```
	
> - 这个方法非常重要，我们可以看到前面有一堆提示，而后面**创建了tmp/cache,tmp/pids,tmp/sessions和tmp/sockets文件夹**,

> - 在调用**wrapped\_app方法创建Rack应用**。

> - 并且**定了ActiveSupport::Logger**。

还记得**options[:config]默认指向config.ru**吗？这个文件中包含了：

``` ru
    # This file is used by Rack-based servers to start the application.

    require ::File.expand_path('../config/environment',  __FILE__)
    run <%= app_const %>
```

它**引进了config/environment.rb**文件，而**environment.rb这个文件引入了config/application.rb**文件

------------------------------------------------------------------------

###actionpack/lib/action\_dispatch.rb

Action Dispatch是rails框架的一个路由组件，提供routing,session和middlewares

也就是，你启动了服务前才帮你加载程序的应用文件application.rb。

------------------------------------------------------------------------

####总结：(自己所画的rails初始化过程图片)

![图片无法显示](/images/posts/2013-10-23/rails_initialize.png "rails初始化过程")

