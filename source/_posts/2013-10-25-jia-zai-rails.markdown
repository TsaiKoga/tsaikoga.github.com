---
layout: post
title: "加载Rails"
date: 2013-10-25 22:58
comments: true
categories: [Rails]
---
[1]: http://tsaikoga.github.io/blog/2013/10/24/railschu-shi-hua-he-qi-dong-guo-cheng/

在上一篇文章[Rails服务启动过程][1],最后加载的config/application.rb文件中显示：

``` ru
	require 'rails/all'
```

这个文件在railties/lib/rails/all.rb。

---------------------------------------

###railties/lib/rails/all.rb

这个文件非常重要，它加载了rails的所有个体框架，从中可以知道这些**框架的加载顺序**：

``` ru
    require "rails"

    %{
        active_record
        action_controller
        action_mail
        rails/test_unit
        sprockets
    }.each do |framework|
        begin
            require "#{framework}/railtie"			# 文件active_record/railtie等
      rescue LoadError
        end
    end
```

> **railtie 是Rails的核心框架**，并且提供hooks来修改启动时的载入流程，
> 如果需要在启动过程或之后与rails框架进行交互，就需要加载Railtie。

-----------------------------------------

###config/environment.rb

当config/application.rb已经将rails加载完成,并且**定义了应用的命名空间**（在一个module中mixin一个Application类）。

而在config/environment.rb文件中，你的**项目被初始化**，如：**Examples::Application.initialize!(在application.rb中已经定义了)**。

想知道初始化做了写什么吗？

------------------------------------------

###railties/lib/rails/application.rb

以下为初始化代码：(你的Application类继承Rails::Application类，而这个类就在railties/lib/rails/application.rb文件中)

``` ru
    def initialize!(group=:default) #:nodoc:
        raise "Application has been already initialized." if @initialized
        run_initializers(group, self)
        @initialized = true
        self
    end
```

Rails会贯穿这整个Rails::Application祖先类，重新排序和运行他们。

###Rack: lib/rack/server.rb

在[上一篇][1]时,有提到Rack::Server类，它里面有一个app方法是这么定义的：

``` ru
    def app
        @app ||= begin
            if !::File.exist? options[:config]
                abort "configuration #{options[:config]} not found"
            end
     
            app, options = Rack::Builder.parse_file(self.options[:config], opt_parser)
            self.options.merge! options
            app
        end
    end
```

这里所指的app就是Rails的app（一种中间件），Rack将会调用所有提供的中间件。

``` ru
    def build_app(app)
        middleware[options[:environment]].reverse_each do |middleware|
            middleware = middleware.call(self) if middleware.respond_to?(:call)
            next unless middleware
            klass = middleware.shift
            app = klass.new(app, *middleware)
        end
        app
    end
```

以上代码build\_app是被Server#start方法调用的，通过这个代码调用：

``` ru
	server.run wrapped_app, options, &blk
```

所以，server.run将会依赖于你所使用的server程序(以上代码有传入一个程序块&blk,他将关系到你运行的服务),
像是如果你是用的是Mongrel数据库，将会通过传入如下程序块运行：

``` ru
    def self.run(app, options={})
        server = ::Mongrel::HttpServer.new(
            options[:Host]           || '0.0.0.0',
            options[:Port]           || 8080,
            options[:num_processors] || 950,
            options[:throttle]       || 0,
            options[:timeout]        || 60)
        # Acts like Rack::URLMap, utilizing Mongrel's own path finding methods.
        # Use is similar to #run, replacing the app argument with a hash of
        # { path=>app, ... } or an instance of Rack::URLMap.
        if options[:map]
            if app.is_a? Hash
                app.each do |path, appl|
                    path = '/'+path unless path[0] == ?/
                    server.register(path, Rack::Handler::Mongrel.new(appl))
                end
            elsif app.is_a? URLMap
                app.instance_variable_get(:@mapping).each do |(host, path, appl)|
                 next if !host.nil? && !options[:Host].nil? && options[:Host] != host
                 path = '/'+path unless path[0] == ?/
                 server.register(path, Rack::Handler::Mongrel.new(appl))
                end
            else
                raise ArgumentError, "first argument should be a Hash or URLMap"
            end
        else
            server.register('/', Rack::Handler::Mongrel.new(app))
        end
        yield server  if block_given?
        server.run.join
    end
``` 

以上代码使用Mongrel::HttpServer定义了server实例，然后使用server.run运行.
