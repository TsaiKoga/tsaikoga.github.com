---
layout: post
title: "Rails Enviroments and Configuration"
date: 2015-07-25 13:03
comments: true
categories: [Rails]
---

对于 Rails 的环境和配置，这里翻译了一篇文章。如果有翻译错的地方，欢迎指出。

## 目录：
#### [1.1 Bunlder](#1.1)

##### [1.1.1 Gemfile](#1.1.1)

###### [1.1.1.1 直接从Git仓库加载 gems](#1.1.1.1)

###### [1.1.1.2 从文件系统加载gem](#1.1.1.2)

##### [1.1.2 安装Gem](#1.1.2)

##### [1.1.3 Gem Locking](#1.1.3)

##### [1.1.4 Gem Packages](#1.1.4)

#### [1.2 启动和应用设置](#1.2)

##### [1.2.1 application.rb](#1.2.1)

###### [1.2.1.1 修改加载路径](#1.2.1.1)

###### [1.2.1.2 Time Zones](#1.2.1.2)

###### [1.2.1.3 Localization](#1.2.1.3)

###### [1.2.1.4 Generator 的默认设置](#1.2.1.4)

##### [1.2.2 Initializers](#1.2.2)

###### [1.2.2.1 Backtrace Silencers](#1.2.2.1)

###### [1.2.2.2 过滤参数日志](#1.2.2.2)

###### [1.2.2.3 Inflections](#1.2.2.3)

###### [1.2.2.4 自定义MIME Types](#1.2.2.4)

###### [1.2.2.5 Secret Token](#1.2.2.5)

###### [1.2.2.6 Session Store](#1.2.2.6)

###### [1.2.2.7 参数包装](#1.2.2.7)

##### [1.2.3 额外的配置](#1.2.3)

###### [1.2.3.1 Log-Level覆盖](#1.2.3.1)

###### [1.2.3.2 Schema Dumper](#1.2.3.2)

#### [1.3 开发模式](#1.3)

##### [1.3.1 自动加载类](#1.3.1)

##### [1.3.2 Eager Load](#1.3.2)

##### [1.3.3 错误报告](#1.3.3)

##### [1.3.4 缓存](#1.3.4)

##### [1.3.5 抛出发送错误](#1.3.5)

##### [1.3.6 反对通知](#1.3.6)

##### [1.3.7 慢查询的解释](#1.3.7)

##### [1.3.8 待迁移错误页面](#1.3.8)

##### [1.3.9 Assets的Debug模式](#1.3.9)

#### [1.4 测试模式](#1.4)

#### [1.5生产模式](#1.5)

##### [1.5.1 Assets](#1.5.1)

#### [1.6 日志](#1.6)

##### [1.6.1 Rails的日志文件](#1.6.1)

##### [1.6.2 标记日志](#1.6.2)

##### [1.6.3 日志文件分析](#1.6.3)

###### [1.6.3.1 Rails::Subscriber.colorize_logging](#1.6.3.1)

#### [1.6.4 总结](#1.6.4)


除了 RAILS_ENV 可以开启所需的环境，你也可以通过设置RACK_ENV 可以设置默认环境(development, production, test)


<h2 id="1.1"> 1.1 Bundler</h2>

Bundler 虽然不是Rails4 的特定工具，但是它却能够很好地解决你应用的gems依赖。
Bundler 通过一个配置文件Gemfile 中解决一些列版本的gems依赖。


<h3 id="1.1.1"> 1.1.1 Gemfile</h3>

---------------------------------------
在你的Rails应用的根目录下，这个文件定义了你的Rails应用所有依赖，包括你所用的Rails 版本。Gemfile的基础语法很简单，如下：

``` ru
    gem 'kaminari'
    gem 'nokogiri'
```

可以将这些gem依赖放在一个或多个环境中，具体做法是定义一个或多个环境符号的块，如下例子：

``` ru
    group :development do
      gem 'pry-rails'
    end

    group :test do
      gem 'capybara'
      gem 'database_cleaner'
    end

    group :development, :test do
      gem 'rspec-rails'
      gem 'factory_girl_rails'
    end
```

这个gem 方法还接受第二个可选参数，也就是Rubygem的版本号，如果此参数为空，则会自动下载最新稳定的版本。
如何定义这第二个参数呢？看以下例子：

``` ru
    gem 'nokogiri', '1.5.6'
    gem 'pry-rails', '> 0.2.2'
    gem 'decent_exposure', '~>2.0.1'
    gem 'draper', '1.0.0.beta6'
```

有些时候，你的gem需要用require语句加载进来，在这种情况下，你只需要在gem 的最右边声明就行了。
如：

``` ru
    gem 'webmock', require: 'webmock/rspec'
```


<h4 id="1.1.1.1"> 1.1.1.1 直接从Git仓库加载 gems</h4>

---------------------------------------
现在我们可以从https://rubygems.org 下载gems,也可以非常简单的从gem的仓库获取这个gem，如下代码：

``` ru
    gem 'carrierwave', git: 'git@github.com:jnicklas/carrierwave.git'
```

如果这个gem的源代码放在github上面，可以直接用:github ：

``` ru
    gem 'carrierwave', github:'jnicklas/carrierwave'
```

如果有几个gemspec共用一个git仓库，可以像如下代码操作:mZ    git 'git://github.com/rails/rails.git'

``` ru
    gem 'railties'
    gem 'action_pack'
    gem 'active_model'
```

另外，你可以通过gem的所属仓库，指定对应的ref，branch或者是tag：

``` ru
    git 'git://github.com/rails/rails.git', ref: "4aded"
    git 'git://github.com/rails/rails.git', branch: '3-2-stable'
    git 'git://github.com/rails/rails.git', tag: 'v3.2.11'

    gem 'nokogiri', git: 'git://github.com/tenderlove/nokogiri.git', ref => '0eec4'
```


<h4 id="1.1.1.2"> 1.1.1.2 从文件系统加载gem</h4>

---------------------------------------
有时候你在开发模式下，想要加载一个gem（例如你fork了一个gem，作了修改，但是没有提交，想测试一下）可以使用:path：

``` ru
    gem 'nokogiri', path: '~/code/nokogiri'
```



<h3 id="1.1.2"> 1.1.2 安装Gem</h3>

---------------------------------------
当你对Gemfile做了修改，调用install命令确保Gemfile所有依赖都有效。

``` sh
    $ bundle install
    Fetching gem metadata from https://rubygems.org/.........
    Fetching gem metadata from https://rubygems.org/.........
    Fetching git://github.com/rails/rails.git
    Fetching git://github.com/rails/activerecord-deprecated_finders.git
    Fetching git://github.com/rails/arel.git
    Fetching git://github.com/rails/coffee-rails.git
    Fetching git://github.com/rails/sprockets-rails.git
    Fetching git://github.com/rails/sass-rails.git
    Installing rake (10.0.3)
    Installing i18n (0.6.1)
    .........
```

install命令会更新所有你的Gemfile中不与其他依赖冲突的最新版本依赖。





<h3 id="1.1.3"> 1.1.3 Gem Locking</h3>
---------------------------------------
每一次更新或安装Gem，Bundler都会计算你项目的gem依赖树，并将结果存储在Gemfile.lock文件中。所以，从这点看，在使用Gem时，Gemfile将会被锁定，Bundler只会加载特定的版本的Gem。




<h3 id="1.1.4"> 1.1.4 Gem Packages</h3>

---------------------------------------
你可以将你所有的gems打包到vendor/cache目录中，使用如下命令：

``` sh
    bundle Package
```

当运行bundle install时，将会直接使用vendor/cache目录中的gem而不连接https://rubygems.org

注意：无Rails的脚本运行要加上bundle exec 来初始化，如：

``` sh
    bundle exec guard
```

Bundler还可以为你所有的可执行bundle创建binstubs。通过调用bundle install带上参数--binstubs，就会在你的根目录下创建一个bin目录，其中包括一些RubyGems环境的可执行脚本：

``` ru
    #!/usr/bin/env ruby
    #
    # This file was generated by Bundler.
    #
    # The application 'guard' is installed as part of a gem, and
    # this file is here to facilitate running it.
    require 'pathname'
    ENV['BUNDLE_GEMFILE'] ||= File.expand_path("../../Gemfile",
                                               Pathname.new(__FILE__).realpath)
mZ    require 'rubygems'
    require 'bundler/setup'

    load Gem.bin_path('guard','guard')
```

使用binstubs，可以不用加bundle exec 而直接从bin/目录执行脚本.

``` sh
    $ bin/guard
```

你还可以将./bin加入$PATH路径，然后就可以轻松的执行：

``` sh
    $ guard
```

使用Rails4创建项目，会自动创建binstubs，创建了bin/bundle, bin/rails,bin/rake，更新特定的stubs，可以使用下列命令：

``` sh
    rake rails:update:bin
```





<h2 id="1.2"> 1.2 启动和应用设置</h2>

---------------------------------------
用Rails开启进程处理请求（例如：rails sever），第一件事就是加载config/boot.rb文件。
建立整个rails栈，主要引用三个文件：
> boot.rb: 建立bunlder和加载路径
> application.rb: 加载rails的gems，为gems定义rails环境，进行应用配置
> environments.rb:运行所有初始化




<h3 id="1.2.1"> 1.2.1 application.rb</h3>

---------------------------------------
config/application.rb是Rails应用设置的主要文件，也是config/environments.rb最先加载的文件。
让我们一步一步来看这个文件，application.rb的下一行，一旦config/boot.rb加载，轮子就转动

``` ru
    require File.expand_path('../boot',__FILE__)
```

对于下面这行代码：

``` ru
    require 'rails/all'
```

这你可以更改，将你所想要的组件加载进来，而不是加载所有的rails组件。

``` ru
    # To pick the frameworks you want, remove 'require "rails/all"'
    # and list the framework railties that you want:
    #
    # require "active_model/railtie"
    # require "active_record/railtie"
    # require "action_controller/railtie"
    # require "action_mailer/railtie"
    # require "action_view/railtie"
    # require "sprockets/railtie"
    # require "rails/test_unit/railtie"
```




<h4 id="1.2.1.1"> 1.2.1.1 修改加载路径</h4>

---------------------------------------
默认的rails会寻找标准目录如：app/models 或是app/controllers,自动作为加载路径，你也可以自己添加目录到加载路径：

``` ru
    # Custom directories with classes and modules you want to be auto loadable
    #config.autoload_paths += %W(#{config.root}/extras)
```

注意：config.root指向你的rails应用的根目录，因此如果你需要，想要用一个presenters的目录代替原有的models，可以这么做：

``` ru
    config.autoload_paths += %W(#{config.root}/app/presenters)
```

这里的%W方法作为以空白隔开元素的数组，使代码构建更加方便。




<h4 id="1.2.1.2"> 1.2.1.2 Time Zones</h4>

---------------------------------------
rails4默认的time zones是utc，你也可以覆盖它的默认值：

``` ru
    # Set Time.zone default to the specified zone and make activerecord
    # auto-convert to this zone.
    # Run "rake-Dtime" for a list of tasks for finding time zone names.
    config.time_zone = 'CentralTime(US&Canada)'
```




<h4 id="1.2.1.3"> 1.2.1.3 Localization</h3>

---------------------------------------
Rails默认的语言Locale是:en 英文，你也可以在配置文件进行修改：

``` ru
    # The default locale is :en and all translations from
    # config/locales/*.rb,yml are auto loaded.
    # config.i18n.load_path += Dir[Rails.root.join('my','locales',
                                                 # '*.{rb,yml}')]
    # config.i18n.default_locale = :de
```




<h4 id="1.2.1.4"> 1.2.1.4 Generator 的默认设置</h4>

---------------------------------------
generator脚本可以根据你的工具链做出特定假定；在这里设置确定的值可以减少命令行参数的输入。
例如：运用没有features的Rspec和haml作为模板引擎时，我们可以这么做：

``` ru
    # Configure generators values. Many other options are available,
    # be sure to check the documentation.
    config.generators do |g|
      g.template_engine :haml
      g.test_framework :rspec, fixture: false
    end
```

注意：rubygems的一些gems像rspec-rails,haml-rails,factory_gril_rails已经为你做了这些处理的。



<h3 id="1.2.2"> 1.2.2 Initializers</h3>

---------------------------------------
Rails2介绍了一个概念，将配置设置放入config/initializers目录中建立的一个小的ruby文件，config/initializers目录中的文件会在程序启动后被自动加载。

你可以增加自己需要的配置设置，通过再initializers目录中建立ruby文件，很多人喜欢在这个目录中建立文件设置常量，为了安全这些常量的值放在yml文件中，通过initializers目录中的rb文件进行引入并赋予常量。




<h4 id="1.2.2.1">1.2.2.1 Backtrace Silencers</h4>

---------------------------------------
没人喜欢很长的exception的回溯跟踪，rails 有个机制可以减少回溯的层数（这种方法是删除行）。

``` ru
    # You can add backtrace silencers for libraries that you're using but
    # don't wish to see in your backtraces.
    Rails.backtrace_cleaner.add_silencer{|line|line=~/my_noisy_library/}

    # You can also remove all the silencers if you're trying to debug a
    # problem that might stem from framework code.
    Rails.backtrace_cleaner.remove_silencers!
```




<h4 id="1.2.2.2"> 1.2.2.2 过滤参数日志</h4>

---------------------------------------
当有请求到你的rails应用，默认的，rails的logs文件有一些细节：请求路径,http 方法，Ip地址和参数。倘若黑客利用某些方法得到你的日志文件，那么他就能看到这些敏感信息，包括密码信息。

filter_parameter_logging.rb 初始化器可以让你设置哪些请求参数需要从你的log文件中过滤掉。
如果rails 接收到请求参数被你设置的filter_parameters收集了，将会在你的log文件中标记为[FILTERED]。

``` ru
    # Configure sensitive parameters which will be filtered from the log file.
    Rails.application.config.filter_parameters += [:password]
```




<h4 id="1.2.2.3"> 1.2.2.3 Inflections<h4>

---------------------------------------
Rails 有一个Infector类，它负责将字符串变为单复数，类名变表名，模块化类名，类名变外键名等（其中一些操作有滑稽的名字，如：dasherize）
默认的，对于一些不可数的名词，ActiveSupport这个gem的infections.rb文件中。
大多数时候，Infector类有一份得体的工作，就是把类名转化为相应复数的表名。

``` sh
    $ rails console
    >> ActiveSupport::Inflector.pluralize "project"
    => "projects"
    >> ActiveSupport::Inflector.pluralize "virus"
    => "viri"
    >> "pensum".pluralize # Inflector features are mixed into String by default
    => "pensums"
```

然而，你可以为infector添加新规则：在config/initializers/inflections.rb 文件：

``` ru
    ActiveSupport::Inflector.inflections(:en) do |inflect|
      inflect.plural /^(ox)$/i, '\1en'
      inflect.singular /^(ox)en/i, '\1'
      inflect.irregular 'person', 'people'
      inflect.uncountable %w( fish sheep  )
    end
```

``` ru
activesupport/test/inflector_test_cases.rb文件有一长串用Infector正确处理的复数转化列表，例如：
    "datum" => "data",
    "medium" => "media",
    "analysis" => "analyses"
```



<h4 id="1.2.2.4"> 1.2.2.4 自定义MIME Types</h4>

---------------------------------------
Rails支持标准的MIME TYPES设置(\*/\*, text/html, text/plain, text/javascript, text/css, text/calendar, text/csv, application/xml, application/rss+xml, application/atom+xml, application/x-yaml, multipart/form-data, application/x-www-form-urlencoded, application/json)：


<table><tbody>
<tr><td>*Short name*</td> <td>respond_to* symbol*</td>  <td>*Aliases and Explanations* </td></tr>
<tr><td>text/html</td>  <td>:html, :xhtml </td>      <td>application/xhtml+xml</td></tr>
<tr><td>text/plain</td> <td>:text, :txt  </td>   <td>                 </td></tr>
<tr><td>text/javascript</td> <td>:js      </td> <td> application/javascript, application/x-javascript </td></tr>
<tr><td>text/css</td> <td>:css</td>   <td> Cascading style sheets     </td></tr>
<tr><td>text/calendar</td> <td>:ics</td>  <td>iCalendar format for sharing meeting requests and tasks </td></tr>
<tr><td>text/csv</td>  <td>:csv</td> <td> Comma-separated values </td></tr>
<tr><td>application/xml</td> <td>:xml</td> <td>   text/xml, application/x-xml </td></tr>
<tr><td>application/rss+xml</td> <td>:rss</td> <td>Really Simple Syndication format for web feeds</td></tr>
<tr><td>application/atom+xml</td> <td>:atom</td> <td>Atom Syndication Format for web feeds</td></tr>
<tr><td>application/x-yaml</td> <td>:yaml</td> <td>text/yaml - The human-readable data serialization format</td></tr>
<tr><td>application/x-www-form- urlencoded</td> <td>url_encoded_form</td> <td>The default content type of HTML forms</td></tr>
<tr><td>multipart/form-data</td> <td>:multipart_form</td> <td>Used for HTML forms that contain files, non-ASCII data, and
binary data</td></tr>
<tr><td>application/json</td> <td>:json</td> <td>text/x-json, application/jsonrequest-JavaScript Object Notation</td></tr>
</tbody></table>

如果项目需要响应其他MIME TYPE，你需要再mime_types.rb文件中进行登记：

``` ru
    # Add new mime types for use in respond_to blocks:
    # Mime::Type.register "text/richtext" ,:rtf
    # Mime::Type.register_alias "text/html", :iphone
```



<h4 id="1.2.2.5">1.2.2.5 Secret Token</h4>

---------------------------------------
有些的黑客可以在服务器不知情的情况下修改该cookies内容，通过数字签名发送给浏览器cookies,而Rails能探测他们是否被捣鼓。
这和secret_token.rb文件有关，这个initializer有secret key，伴随着你的应用随机产生，用来标记cookie。

``` ru
    # Your secret key for verifying the integrity of signed cookies.
    # If you change this key, all old signed cookies will become invalid!

    # Make sure the secret is at least 30 characters and all random,
    # no regular words or you'll be exposed to dictionary attacks.
    # You can use `rake secret` to generate a secure secret key.

    # Make sure your secret_key_base is kept private
    # if you're sharing your code publicly.
    Example::Application.config.secret_key_base = 'f32b1a3755e05a3d...'
```

rake secret 可以重新生成secret key。




<h4 id="1.2.2.6">1.2.2.6 Session Store</h4>

------------------------------------------
Rails4 session cookies加密使用的是最新cookis加密存储策略，session_store.rb这个initializer通过设置session store的type和key来配置你的应用的session存储。

``` ru
    Example::Application.config.session_store :encrypted_cookie_store, key: '_example_session'
```

session cookies 是用secret_token.rb中的secret_key_base来标记的，如果你感到困惑，可以更改secret_token.rb 的secret key值，或者运行rake secret 来重新生成secret key。




<h4 id="1.2.2.7">1.2.2.7 参数包装<h4>

-----------------------------------------
Rails3.1后引入的，wrap_parameters.rb这个initializer能配置你的应用与javascript mvc框架交互。（例如：backbone.js）

``` ru
    # Enable parameter wrapping for JSON. You can disable this by setting
    # :format to an empt yarray.
    ActiveSupport.on_load(:action_controller) do
      wrap_parameters format: [:json] if respond_to?(:wrap_parameters)
    end

    # To enable root element in JSON for ActiveRecord objects.
    # ActiveSupport.on_load(:active_record) do
    #   self.include_root_in_json = true
    # end
```

这样到底会发生什么呢？当你向controller提交json参数时，rails将参数包裹到一个嵌套的hash中，并用controller的名字作为这个hash的key，例如：

``` ru
    {"title": "The Rails4 Way"}
```

将上诉代码提交到ArticlesController,将会转化成下列样子：

``` ru
    { "title": "The Rails 4 Way","article"=>{"title":" The Rails 4 Way"}  }
```


<h3 id="1.2.3">1.2.3 额外的配置</h3>

----------------------------------------
大多数配置选项我们可以在application.rb文件和标准的initializers文件中找到例子，这里还有一些额外的选项，你可以将它添加到额外的intializers文件中。

<h4 id="1.2.3.1">1.2.3.1 Log-Level覆盖</h4>

---------------------------------------
默认的log Level是:debug，如果你有需要，也可以覆盖它；

``` ru
    # Force all environments to use the same logger level
    # (by default production uses:info, the others :debug)
    config.log_level = :debug
```



<h4 id="1.2.3.2">1.2.3.2 Schema Dumper</h4>

--------------------------------------
每一次，当你运行测试代码，Rails将会用自动导出schema.rb的脚本把你的development数据库的schema导出到你的test数据库中，类似ActiveRecord的迁移脚本，事实上，他们用了相同的API。

如果你正在做一个与schema dumper有冲突的事，你有可能需要用SQL将它转化为旧方式schema dumper。（见注释）

``` ru
    # Use SQL instead of ActiveRecord's schema dumper when creating the
    # test database. This is necessary if your schema can't be completely
    # dumped by the schema dumper, for example, if you have constraints
    # or db-specific column types
    config.active_record.schema_format = :sql
```


<h2 id="1.3">1.3 开发模式</h2>

开发模式是rails的默认模式，也是作为开发者最长时间使用的模式，

``` ru
    # File: config/environments/development.rb
    Example::Application.configure do
      # Settings specified here will take precedence over those in
      # config/application.rb.
```


<h3 id="1.3.1">1.3.1 自动加载类</h3>

-------------------------------------
当你在开发模式下的一个好处是快速反馈机制，当你改变代码时，只要重新刷新浏览器，就能立即看见所改的变化。

这个行为是由config.cache_classes设置的：

``` ru
    # In the development environment your application's code is reloaded on
    # every request. This slows down response time but is perfect for
    # development since you don't have to restart the webserver when you
    # make code changes.
    config.cache_classes = false
```

当cache_classes为true时，Rails会运用Ruby的require去加载类，如果是false，则会用load去加载类。

- 当require 一个ruby文件时，ruby解释器会执行它并且进行缓存；如果文件再次被requirmZ会忽略这次require。

- 而当你load 一个ruby文件时，不管它之前被load过几次，ruby解释器都会再次执行它。



<h4 id="1.3.1.1">1.3.1.1 The Rails Class Loader类加载器</h4>

-----------------------------------
在平常老版本的ruby中，不需要为了匹配脚本的内容而更改脚本的名字。在Rails中你会发现Ruby文件的名字与类或模块有联系。
Rails利用了Ruby对未知常量的回调机制，当Rails遇到未定义的常量时，它会利用类加载器(Class Loader)基于文件名的惯例去寻找并require需要的Ruby脚本。

那么你一定会问了，类加载器是如何找到需要的Ruby脚本的？我们已经在前面讨论了initializer.rb在Rails启动进程中的作用，Rails有一个加载路径(LOAD_PATH)的概念，默认加载路径包括你的Rails应用中任何地方的目录。

想要看到项目加载路径的内容，在console中输入：$LOAD_PATH

``` sh
    $rails console
    Loading development environment.
    >>$LOAD_PATH
    => ["/usr/local/lib/ruby/... # about 20 lines of output"]
```

典型的Rails项目至少有60条加载路径，你可以试一试。


<h4 id="1.3.1.2">1.3.1.2 Rails, Modules, and Auto-Loading Code</h4>

----------------------------------
在Ruby中，如果你想从应用中加载其他文件的代码，你需要require声明；然而，Rails为Ruby增加一个默认行为通过建立一个简单的惯例能让Rails在大多数情况下自动加载你的代码。
如果你用过rails console，你就会发现你不用明确地require 任何代码。

现在我来告诉你这是如何实现的：
如果Rails遇到未定义的类或模块，它将会用以下惯例去猜测如何加载该类或模块：

如果该类或模块没有嵌套，则在常量名之间插入下划线：

- EstimationCalculator 将会插入：require "estimation_calculator"

- KittTurboBoost 将会插入：require "kitt_turbo_boost"

如果类或模块被嵌套，则会对每个模块做如上操作，并且形成对应的子目录，例如：
- MacGyver::SwissArmyKnife 变成：require "mac_gyver/swiss_army_knife"

- Example::ReallyRatherDeeply::NestedClass 变成："example/really_rather_deeply/nested_class"，如果还没加载，Rails将会去example/really_rather_deeply/目录下寻找nested_class.rb文件，这个example/really_rather_deeply/目录会在Ruby的加载路径中查找（app/,lib/或者插件的lib/目录）

所以，如果你按照rails这个规则，就基本不用怎么明确地指mZe)要加载的文件了。


<h3 id="1.3.2">1.3.2 Eager Load</h3>

---------------------------------
为了在开发模式下加快启动Rails服务的时间，你的代码一般被禁止eager loaded，你可以通过config.eager_load进行设置。

``` ru
    # Do not eager load code on boot.
    config.eager_load = false
```

在生产模式下，你会将这个值设置为true，它会复制你的应用的大部分内容到内存中，提高web服务器的读写性能，就像Unicorn一样。


<h3 id="1.3.3">1.3.3 错误报告</h3>

--------------------------------来自localhost的请求，当你想要在开发时，能够产生有用的错误信息包括debug信息（错误出现的行号等）那么设置consider_all_requests_local为true就能使rails产生这些友好的信息。

``` ru
    config.consider_all_requests_local = true
```



<h3 id="1.3.4">1.3.4 缓存</h3>

-------------------------------
在开发模式下你一般不希望缓存行为，你真正需要它的时候是你想要去测试缓存是否正确。

``` ru
    config.action_controller.perform_caching = true # for testing in development model.
```

记住：当你测试完要设置为false。


<h3 id="1.3.5">1.3.5 抛出发送错误</h3>

--------------------------------
在开发模式下，rails会假定你不需要ActionMailer去抛出发送错误的异常，这个是基于config.action_mailer.raise_delivery_errors设置的。邮件发送功能不需要在开发模式下运行，特别是windows等其他平台。

``` ru
    # Don't care if the mailer can't send.
    config.action_mailer.raise_delivery_errors = false
```

如果你真的需要在开发模式下发送邮件用来测试或排除错误，那么你可以将它设置为true。



<h3 id="1.3.6">1.3.6 反对通知</h3>

------------------------------
当你想停止使用特定功能时，反对警告是非常有用的，config.active_support.deprecation允许你设置如何收到反对警告。在开发模式下，默认是出现在log文件中。

``` ru
    # Print deprecation notices to the Rails logger.
    config.active_support.deprecation = :log
```



<h3 id="1.3.7">1.3.7 慢查询的解释</h3>

-----------------------------
Rails3.2后引入的，Active Record现在监视SQL查询的门槛已经被设立。当查询花费时间超过这个设定的门槛，这个查询计划将被记录为警告。
而这个门槛的时间是由config.active_record.auto_explain_threshold_in_seconds 设置的：

``` ru
    # Log the query plan for queries taking more than this (works)
    # with SQLite, MySQL, and PostgreSQL).
    config.active_record.auto_explain_threshold_in_seconds = 0.5
```



<h3 id="1.3.8">1.3.8 待迁移错误页面</h3>

----------------------------
在Rails前面版本中，如果待迁移任务需要执行，web服务将会启动失败。
在Rails4中将会以一个错误页面展示，指示开发者运行rake db:migrate RAILS_ENV=development去解决这个问题。

``` ru
    # Raise an error on page load if there are pending migrations
    config.active_record.migration_error = :page_load
```


<h3 id="1.3.9">1.3.9 Assets的Debug模式</h3>

---------------------------
Rails3.1引入Assets Pipeline，一个可以连接和缩小javascript和css的assets。
默认的，在开发模式下，javascript和css还是放在各自不同的目录中。
设置config.assets.debug 为false，将会导致Sprockets对所有assets进行连接和运行预处理器。

``` ru
    # Debug mode disables concatenation and preprocessing of assets.
    config.assets.debug = true
```



<h2 id="1.4">1.4 测试模式</h2>

无论你在测试环境下运行rails， RAILS_ENV的值为test，然后下面的设置将会起作用。

``` ru
    # File: config/environments/test.rb
    Example::Application.configure do
      # Settings specified here will take precedence over those in
      # config/application.rb.

      # The test environment is used exclusively to run your application's
      # test suite. You never need to work with it otherwise. Remember that
      # your test database is "scratch space" for the test suite and is wiped
      # and recreated between test runs. Don't rely on the data there!
      config.cache_classes = true

      # Do not eager load code on boot. This avoids loading your whole
      # application just for the purpose of running a single test. If you are
      # using a tool that preloads Rails for running tests, you may have to
      # set it to true.
      config.eager_load = false

      # Configure static asset server for tests with Cache-Control for
      # performance.
      config.serve_static_assets = true
      config.static_cache_control = "public, max-age=3600"

      # Show full error reports and disable caching.
      config.consider_all_requests_local = true
      config.action_controller.perform_caching = false

      # Raise exceptions instead of rendering exception templates.
      config.action_dispatch.show_exceptions = false

      # Disable request forgery protection in test environment.
      config.action_controller.allow_forgery_protection = false

      # Tell Action Mailer not to deliver emails to the real world.
      # The :test delivery method accumulates sent emails in the
      # ActionMailer::Base.deliveries array.
      config.action_mailer.delivery_method = :test

      # Print deprecation notices to the stderr.
      config.active_support.deprecation = :stderr
    end
```

### i: 自定义环境
*如果有需要，你可以给你的rails应用自定一个额外的环境，通过复制一个已经存在的环境文件(在config/environments目录中)。
自定义环境的一个共同的用法是建立一个额外的生产模式配置（例如：staging和QA development模式）。
在开发平台，你拥有进入正式环境数据库的权利吗？哈哈，使用 triage 环境就在合理不过了。运用开发模式作为正常的环境，但是数据库连接生产服务器的数据库。当你想要快速诊断生产环境下的问题，这非常有用。*



<h2 id="1.5">生产模式</h2>

最后，当你部署自己的项目并接受公共请求时，生产模式是你的rails应用运行的环境。
生产模式与其他模式有一些显著的区别，不仅仅是速度的提升。

``` ru
    # File:config/environments/production.rb
      # Settings specified here will take precedence over those in
      # config/application.rb.

      # Code is not reloaded between requests.
      config.cache_classes = true

      # Eager load code on boot. This eager loads most of Rails and
      # your application in memory, allowing both thread web servers
      # and those relying on copy on write to perform better.
      # Rake tasks automatically ignore this option for performance.
      config.eager_load = true

      # Full error reports are disabled and caching is turned on.
      config.consider_all_requests_local = false
      config.action_controller.perform_caching = true

      # Enable Rack::Cache to put a simple HTTP cache in front of your
      # application
      # Add `rack-cache` to your Gemfile before enabling this.
      # For large-scale production use, consider using a caching reverse proxy
      # like nginx, varnish or squid.
      # config.action_dispatch.rack_cache = true

      # Disable Rails's static asset server (Apache or nginx will
      # already do this).
      config.serve_static_assets = false

      # Compress JavaScripts and CSS.
      config.assets.js_compressor = :uglifier
      # config.assets.css_compressor = :sass

      # Whether to fallback to assets pipeline if a precompiled
      # asset is missed.
      config.assets.compile = false

      # Generate digests for assets URLs.
      config.assets.digest = true

      # Version of your assets, change this if you want to expire
      # all your assets.
      config.assets.version = '1.0'

      # Specifies the header that your server uses for sending files.
      # config.action_dispatch.x_sendfile_header = "X-Sendfile" # for Apache
      # config.action_dispatch.x_sendfile_header = 'X-Accel-Redirect'
      # for nginx

      # Force all access to the app over SSL, use Strict-Transport-Security,
      # and use secure cookies.
      # config.force_ssl = true

      # Set to :debug to see everything in the log.
      config.log_level = :info

      # Prepend all log lines with the following tags.
      # config.log_tags = [ :subdomain, :uuid  ]

      # Use a different logger for distributed setups.
      # config.logger = ActiveSupport::TaggedLogging.new(SyslogLogger.new)

      # Use a different cache store in production
      # config.cache_store = :mem_cache_store

      # Enable serving of images, stylesheets, and JavaScripts from an
      # asset server.
      # config.action_controller.asset_host = "http://assets.example.com"

      # Precompile additional assets.
      # application.js, application.css, and all non-JS/CSS in app/assets
      # folder are already added.
      # config.assets.precompile += %w( search.js  )

      # Ignore bad email addresses and do not raise email delivery errors.
      # Set this to true and configure the email server for immediate delivery
      # to raise delivery errors.
      # config.action_mailer.raise_delivery_errors = false

      # Enable locale fallbacks for I18n (makes lookups for any locale fall
      # back to the I18n.default_locale when a translation can not be found).
      config.i18n.fallbacks = true

      # Send deprecation notices to registered listeners.
      config.active_support.deprecation = :notify

      # Log the query plan for queries taking more than this (works
      # with SQLite, MySQL, and PostgreSQL).
      # config.active_record.auto_explain_threshold_in_seconds = 0.5

      # Disable automatic flushing of the log to improve performance.
      # config.autoflush_log = false

      # Use default logging formatter so that PID and timestamp
      # are not suppressed.
      config.log_formatter = ::Logger::Formatter.new
    end
```



<h3 id="1.5.1">1.5.1 Assets</h3>

----------------------------------
在生产模式下，assets是默认由assets pipeline预处理的。所有的文件包括application.js和application.css的assets清单被压缩和连接到他们各自的同名文件中，位于public/assets目录中。

如果被请求的assets不存在public/assets目录中，rails将会抛出异常。
设置config.assets.compile为true，可以在生产模式下启用live assets做后备。

在assets pipeline预处理时，application.css和application.js是被唯一加载的stylesheets/javascript清单文件。
想要加载其他assets，在定义config.assets.precompile的地方设置就行了：

``` ru
    config.assets.precompile += %w( administration.css  )
```

像Rails大多数特征一样，Asset Pipeline 的用法完全是可选的，config.assets.enabled设置为false



<h3 id="1.5.2">1.5.2 Asset 主机</h3>

---------------------------------
默认地，Rails通过当前主机的public/ 目录连接assets，但是你可以直接将assets连接到一个assets服务器。config.action_controller.asset_host在后面章节会讲到。



<h2 id="1.6">1.6 日志</h2>

Rails的大多数编程环境*（models, controllers, view templates）有logger属性*，遵从Log4r接口或默认Ruby1.8+Logger类。

无法在你的代码中获得日志的引用？利用Rails.logger 方法你可以在任何地方引用日志。
我们非常容易创建一个新的日志，见下面例子：

``` sh
    $ irb
    > require 'logger'
    => true

    irb(main):002:0> logger = Logger.new STDOUT
    =>#<Logger:0x32db4c@level=0,@progname=nil,@logdev=
    #<Logger::LogDevice:0x32d9bc...>

    > logger.warn "do not want!!!"
    W, [2013-02-01T10:59:26.896825 #6243] WARN -- : do not want!!!
    => true

    > logger.info "in your logger, giving info"
    I, [2013-02-01T10:59:42.478325 #6243] INFO -- : in your logger, giving info
    => true
```

你可以利用logger将信息加入日志无论它什么时候产生，这些日志信息运用对应程度的方法。
标准的日志程度有：

- **debug** : 运用debug捕获数据和有用的应用状态来排除错误。这种级别通常不会被捕获到production日志中。

- **info** : 运用info捕获信息报文。我喜欢运用这种日志级别在良好应用程序情况下记录非正常事件的时间戳。

- **warn** : 用warn级别捕获那些不平常的可能值得调查的事件。有时候，我会抛出warning日志去阻止用户做一些我不允许做的事情。
我的目的是对那些坏恶意的用户或是漏洞继续保持我的应用的界面友好性。如下例子：

``` ru
    def create
      begin
        group.add_member(current_user)
        flash[:notice] = "Successfully joined #{scene.display_name}"
      rescue ActiveRecord::RecordInvalid
        flash[:error] = "You are already a member of #{group.name}"
        logger.warn "A user tried to join a group twice. UI should
        not have allowed it."
      end
      redirect_to :back
    end
```

- **error** : 运用error级别捕获那些不需要重新启动服务的错误信息。

- **fatal** : 可以想象最坏情况的发生：你的应用挂了，并且需要手动重启。


<h3 id="1.6.1">1.6.1 Rails的日志文件</h3>

------------------------------------
Rails的log目录中有三个对应三种标准环境的log文件，随着时间的增长，这些文件会不断变大。这里介绍一个rake任务，可以简单地清理这些log文件：

``` sh
    $ rake log:clear # Truncates all *.log files in log/ to zero bytes
```

当你在工作时，log/development.log显得非常有用，许多程序员会在他们编程时打开另一个终端并持续显示开发模式下的日志尾部信息：

``` sh
    $ tail -f log/development.log

    Article Load (0.2ms)  SELECT "articles".* FROM "articles" WHERE
      "articles"."id" = $1 LIMIT 1  [["id", "1"]]
```

在development的日志中各种有价值的信息是非常有用的。举个例子，每次你发一个请求，一串有用的信息将会出现在你的日志中，这是我项目的样本：

``` sh
    Started GET"/user_photos/1" for 127.0.0.1 at 2007-06-0617:43:13
      Processing by UserPhotosController#show as HTML
      Parameters: {"/users/8-Obie-Fernandez/photos/406"=>nil,
      "action"=>"show", "id"=>"406", "controller"=>"user_photos",
      "user_id"=>"8-Obie-Fernandez"}
      User Load (0.4ms)  SELECT * FROM users WHERE (users.'id' = 8)
      Photo Load (0.9ms)  SELECT * FROM photos WHERE (photos.'id' = 406
      AND (photos.resource_id = 8 AND photos.resource_type = 'User'))
      CACHE (0.0ms)   SELECT * FROM users WHERE (users.'id' = 8)
    Rendered adsense/_medium_rectangle(1.5ms)
      User Load (0.5ms) SELECT * FROM users WHERE (users.'id' = 8)
      LIMIT 1
      SQL (0.4ms) SELECT count(*) AS count_all FROM messages WHERE
      (messages.receiver_id = 8 AND (messages.'read' = 0))
    Rendered layouts/_header(25.3ms)
    Rendered adsense/_leaderboard(0.4ms)
    Rendered layouts/_footer(0.8ms)
    Rendered photos/show.html.erb within layouts/application.html.erb(38.9ms)
    Completed in 99ms(Views:37.4ms|ActiveRecord:12.3ms) with 200
```

下列列出日志中所有数据项包括的内容：

- controller和action的调用

- 发出请求的的远程计算机IP地址

- 请求发生的时间戳

- 请求的*session ID*

- 请求的参数hash

- 请求的数据库信息，包括话费的时间和数据库执行语句

- 缓存查询的命中信息：包括话费的时间和SQL语句从缓存中触发结果而不是往返数据库。

- 渲染每个模板视图的输出及其话费的时间

- 完成一个请求的合计时间

- 数据库操作和渲染的时间分析

- HTTP的状态码和响应客户端的URL



<h3 id="1.6.2">1.6.2 标记日志</h3>

------------------------------------
日志信息有大范围数量信息，想要跟踪问题或特殊请求显得非常困难。为了减轻这种情况，Rails3.2引入了一种可以标记日志信息的能力。
最简单的方法去添加标记到你的日志文件，就是去包含一个响应请求对象的方法，设置config.log_tags。
举例，设置config.log_tags 去包含:subdomain：

``` ru
    config.log_tags = [:subdomain]
```

这将导致对每个日志信息请求前面都加上一个子域：

``` sh
    [admin]Started GET "/articles" for 127.0.0.1 at 2013-02-01 11:49:09-0500
```



<h3 id="1.6.3">1.6.3 日志文件分析</h3>

-----------------------------------
一些非正式的分析，可以使用development日志的输出和一些常识判断。
**性能：** 更明显的分析之一是研究你的应用的性能。你的请求越快，你的rails进程就能处理越多的请求。这就是为什么性能数据通常用每秒来表示请求，这样你就可以通过查找渲染和查询语句来分析为什么花费那么长时间。

要意识到日志文件记录的报告并不是非常准确，这点很重要。事实上，如果单单从内部本身很难衡量事物的时间，错误可能更多。
为任何给定的请求加上渲染和数据库时间百分比，不会总是接近100%的。

但是，尽管在纯粹客观的意义上不准确，在同一个应用中对数据报告时间进行主观判断也是完美的。它给你一个途径去判断这个action是不是比之前花费更长的时间；或者是不是比其他的action速度更快或更慢，等等。

**SQL查询：**
Active Record 没有像预期的那样执行？事实上，记录在日志文件的Active Record可以帮助你排除一些由复杂的查询导致的问题。

**鉴别N+1选择问题：**
无论什么时候，当你在展示一条关联其他记录的记录时，你可能会面对一种叫做N+1的选择问题。你将会意识到这些*选择语句只有主键的值是不同的*。

举个例子，下面是一个真实的项目中日志文件片段，当FlickrPhoto 实例被加载时产生的N+1问题：

``` sh
    FlickrPhoto Load (1.3ms) SELECT * FROM flickr_photos WHERE
    (flickr_photos.resource_id = 15749 AND flickr_photos.resource_type =
     'Place' AND (flickr_photos.'profile' = 1)) ORDER BY updated_at desc
    LIMIT 1

    FlickrPhoto Load (1.7ms) SELECT * FROM flickr_photos WHERE
    (flickr_photos.resource_id = 15785 AND flickr_photos.resource_type =
    'Place' AND (flickr_photos.'profile' = 1)) ORDER BY updated_at desc
    LIMIT 1

    FlickrPhoto Load (1.4ms) SELECT * FROM flickr_photos WHERE
    (flickr_photos.resource_id = 15831 AND flickr_photos.resource_type =
    'Place' AND (flickr_photos.'profile' = 1)) ORDER BY updated_at desc
    LIMIT 1
```

等等，是不是觉得很熟悉？
幸运的是，这些语句执行速度还是非常快的，每条接近0.0015 秒。缺少减轻的因素，我将面临一连串待处理的性能问题。当你的数据库在一个单独的机器上，这个问题将非常严重，他将处理每一条查询而造成网络延迟。

N+1问题并不可怕，很多时候只需要在特定的查询语句使用includes方法就能解决问题。


### i: 关注分离（Separation of Concerns）

 *一个精心设计的MVC应用遵循一定的协议：逻辑层数据库操作必然在model中执行，视图完成渲染任务。一般而言，你的controller需要渲染视图，必须通过model到数据库获取数据。在Rails中，controller完成对model查询的数据的处理并进行视图的渲染。*

在渲染过程中进行数据库的访问是一种非常不好的做法。直接从模板的代码中调用数据库方法背离了**关注分离**原则，简直是一个恶梦。

然而有其他方法可以在你渲染模板时含蓄的访问数据库，通过model进行封装，可能用lazy loading关联触发。
我们可以不容置疑地说这是不好的做法吗？其实这很难说。有一些情况是需要在渲染时访问数据库的（如利用片段缓存的）。



### i: 使用备用的Logging schemes
这很简单，仅仅只要指定一个和Ruby的Logger兼容的类，像ActiveRecord::Base.logger。David曾经说过一个厉害的黑客必须具备交换日志的能力。在控制台会话期间，为了在控制台看到你的SQL语句生成正确，需要指定一个新的Logger实例指向STDOUT到ActiveRecord::Base.logger。
Jamis也完整地写了关于这个的技术博文：http://weblog.jamisbuck.org/2007/1/31/more-on-watching-activerecord



<h4 id="1.6.3.1">1.6.3.1  Rails::Subscriber.colorize_logging</h4>

----------------------------------------
告诉Rails是否使用ANSI代码使日志语句有颜色，颜色可以帮助我们更好地阅读日志文件（除了windows），当你使用系统日志时，也可能使问题复杂化。
默认是true，如果你发现你的软件不支持ANSI代码日志文件颜色化功能，请将它改为false。

以下片段是带有ANSI代码的日志片段：

``` sh
    ^[[4;36;1mSQL (0.0ms)^[[0m ^[[0;1mMysql::Error: Unknown table
    'expense_reports':DROPTABLEexpense_reports^[[0m
      ^[[4;35;1mSQL (3.2ms)^[[0m ^[[0mCREATE TABLE expense_reports ('id'
    int(11) DEFAULT NULL auto_increment PRIMARYKEY,'user_id' int(11))
                        ...]]..]]..]]..]]..]]..]]..]]
```


#### Wilson 说：
> 我见过的几乎没有人知道怎样将日志颜色化，-R选项。


#### i: 系统日志syslog
像Unix系统的系统都有一个系统服务叫做系统日志。因为各种原因，它可能是一个更好的选择。

- 更细粒度控制日志级别和内容

- 多个Rails应用的日志整合输出

- 如果你使用多系统远程日志功能，多个rails应用的日志整合输出是可能的，比起单独处理每个应用程序服务器上的单独日志文件好得多。



<h3 id="1.6.4">1.6.4 总结</h3>

-----------------------------------
我们结束了Rails之旅，这章简要覆盖了bundler的一些细节，并且评估了不同环境下rails的执行，和如何加载它的依赖，包括你的应用代码。并且深入查看application.rb文件和如何更改每种模式来自定义符合我们胃口的Rails的行为。
