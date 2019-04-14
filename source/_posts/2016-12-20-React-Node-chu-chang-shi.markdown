---
layout: post
title: "React Node 初尝试"
date: 2016-12-20 20:29
comments: true
categories: [Javascript]
---

这是第一次写React和Node，选用的是前端Material-ui框架，后端使用的是Express框架，数据库采用的是Mongodb。

项目代码在：[GitHub/lilu_movie](https://github.com/TsaiKoga/lilu_movie)

这是一个通过从电影天堂抓取数据并显示的电影网站，demo部署在heroku上面。


## 安装

------------

首先安装express框架；

``` sh
npm install express --save
```

生成文件后，可以通过 npm start 启动应用。

注意：ejs 从3.x后不支持layout，可以通过express-partials ，但是不支持4，4之后用include

紧接着我迫不及待安装material-ui：

``` sh
npm install material-ui --save
```

然后出现错误：

![material-ui-error](/images/posts/2016-12-20/material-ui-error.png)

所以必须安装react依赖：

``` sh
npm install react@^15.0.0 --save

npm install react-dom@^15.0.0 --save

npm install react-tap-event-plugin@^1.0.0 --save
```

安装nodemon

``` sh
nodemon ./bin/www #更改会自动重启服务
```

本地安装数据库mongodb

然后npm安装操作mongodb的mongoose

``` sh
npm install mongoose

npm install express-mongoose
```

接着你会发现按照material-ui的import引入报错，

使用es6查看系统支持哪些es6语法

``` sh
npm install es6-checker
```

因为react使用es6和jsx语法，所以需要转化，安装如下包：

``` sh
npm install babel-loader babel-core babel-preset-es2015 —save-dev

npm install jsx-loader —save-dev

npm install babel-preset-react
```

安装webpack

``` sh
npm install webpack —save-dev


npm install css-loader —save-dev


npm install webpack-dev-server —save-dev  # 前端文件更新就能自动热启动，会启动端口，默认8080
```

这里有必要提一下：-save-dev代表安装的包适用于开发的，类似于rails中安装Gem放在:development环境下，这样生产环境就不会安装

然后在webpack配置文件中babels的loaders中query加入presets

因为需要一些css文件，react通过require style文件(js 文件中直接require css 文件)，需要安装

``` sh
npm install style-loader —save-dev

npm install css-loader —save-dev // 这个和style一起用才有效果
```

在webpack中的config 加上loader: "style-loader!css-loader”，就不用require使用style!css!了

<br/>


## 启动脚本：

-----------

配置package.json文件，给script添加命令

``` js
 "start": ["node ./bin/www", "webpack"],
```

编写webpack.config.js配置文件，更改html引入文件

在webpack-dev-server 没有真正生成文件，还得要引入<script src=“localhost:8080/assets/bundle.js"></script>

运行

``` sh
npm run dev
```

看webpack-dev-server效果


<br/>

## Express后端流程改变：

-----------

刚开始，我用一贯的后台思路通过routes渲染页面，页面html引入react的js文件，reactjs文件link后台js响应；后台相应通过连接mongodb获取数据库内容。



很成功，获取到相应的内容了，但是因为使用react(React适合做SPA)，所以不好每次都取加载一次内容，所以就不用引擎模板，这些数据如何放入state让react用diff算法自己计算呢？怎样变成单页面应用呢？



这时候我想到就是ajax；上网google一下，发现用fetch能实现像ajax那样的请求。同一个component可以很容易实现fetch数据改变this的state。

详情更改请看我的这个commit：[add client store to fetch movie's data from server](https://github.com/TsaiKoga/lilu_movie/commit/16fef9918a0fc1a2a34e851d88328fcd99b1c2c8)


这时候发现不知道怎么通过点击侧边栏标签，渲染新页面（注意：侧边栏是一个组件，而右边的电影列表又是一个组件，他们是两个不同的组件）


![侧边栏组件](/images/posts/2016-12-20/sidebar.png)


<br/>

## 使用React-Router

--------

由于react规定父元素只能改变子元素，但是不好将子元素改变父元素；

一般我们都会把许多内容都搞到最顶级那个父元素的state，这样其他都有可能与他有关联，而子元素改变父元素的state的方法就是通过回调setState；

所以这里我们可以将movies放在最顶的component，然后点击标签，就去回调去改变这个父元素的state，用到这个state的子元素就会刷新。

例如：父组件有一个属性callbackParent=某个callback函数。然后 子组件 通过事件 触发this.props.callbackParent这个函数，把你要的值传给callback，callback立马设置给父组件state

 方案：[stackoverflow](http://stackoverflow.com/questions/22639534/pass-props-to-parent-component-in-react-js)

 代码详见这个 [commit](https://github.com/TsaiKoga/lilu_movie/commit/248258b5faf0161f66655c745c2bc8e33ffc9798)

**但是这样太hacky了，违背react理念，代码难理解；**

<br/>

有没有其他更好的办法呢？

Google查找答案发现有两种方法：

1. 使用react-router

2. 如果不是用react-router，则得这样写https://github.com/ReactTraining/react-router/blob/master/docs/Introduction.md

react-route根据history传入的链接，找到你对应routes的component，然后改变children，成功渲染改组件。

<br/>
<br/>

对于不同组件改变同个内容还是使用react-router

<br/>
<br/>

使用react-router发现client端通过router的链接，局部更新内容；

这样子说，完全不需要server后台每个路由每次渲染不同页面了，只需要server不同链接给出不同内容，然后渲染同一个页面，这个页面通过react-router去改变内容即可。

所以删除后端所有router路由；

按照React Router官方教程实现相应代码。

<br/>
<br/>

这时候发现一个问题：

渲染同一个页面就要在后端引入前端的routes，也就需要到es6了，但是之前后端没有通过webpack进行es6的转化，所以还要对后端的入口文件进行webpack转化。

<br/>
<br/>

对后端server.js进行webpack的bundle后，很容易报错，首先要在web pack中排除掉node_module的文件，然后需要引入各种loader；
<br/>

server端要import client端的routes过来，但是route的component会引用相应的component。如果遇到client的内容，有些react-router/server是处理不了的，会报没有window错误。
<br/>

以下是最终的webpack.server.config.js

``` js
var webpack = require("webpack");
var fs = require("fs");
var path = require("path");

module.exports = {
  entry: [
    path.resolve(__dirname, 'server.js')
  ],

  output: {
    filename: 'server.bundle.js'
  },
  target: 'node',

  externals: fs.readdirSync(path.resolve(__dirname, 'node_modules')).concat([
    'react-dom/server', 'react/addons',
  ]).reduce(function (ext, mod) {
    ext[mod] = 'commonjs ' + mod
    return ext
  }, {}),

  node: {
    __filename: true,
    __dirname: true
  },
  module: {
    loaders: [
      {
        test: /\.css$/,
        loader: "style-loader!css-loader"
      }, {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: "babel",
        query: { presets: ['react', 'es2015'] }
      }, {
        test: /.json$/, loader: 'json-loader'
      }, {
        test: /.node$/, loader: 'node-loader'
      }]

  }
}
```

事实上使用react react-router已经足够完成功能，但是考虑到如果之后想要扩展，例如现在我又要增加用户注册，登录页面；那么很有可能随着页面组件增多，这个项目将会变成意大利面；

<br/>

## 使用Redux

----------

页面逐渐增多，涉及到了不相关或者层级比较多的组件之间的通信；

以上父子组件通信已经不能满足需求，可以使用 Redux；

他是 **观察者模式**，使用 **订阅分发** 实现：

引入redux;

``` sh

npm install react-redux

npm install redux

```

关于redux，其实应用中所有的 state 都以一个对象树的形式储存在一个单一的 store 中。惟一改变 state 的办法是触发 action，一个描述发生什么的对象。

为了描述 action 如何改变 state 树，你需要编写 reducers，reducers 是一棵操作 state 的树，我用变量 [RootReducer](https://github.com/TsaiKoga/lilu_movie/blob/master/redux/reducers/reducer.js) 命名更加形象。

通过写reducer，可以很好的管理 actions，这些 actions 操作形成了新的state，这样更加条理。可以看我的redux目录

> actions：用来fetch后台数据或者直接返回改变的内容
>
> reducers：通过actions返回的数据，改变state
>
> constants：每个actions的常量名
>
> configureStore.js:通过传入reducers和初始的states，返回store（包含改变的states）

然后前台app.js渲染之前，必须先通过<Provide>传入store；

即：通过 connect() 方法订阅了 reducers 树，触发 reducers 中某个 action，action 请求后台数据，返回 state 对象给添加订阅的 Provider store

<br/>

``` js
const store = configureStore(window.__INITIAL_STATE__)

/* 这里store存储某一时刻的state树，所以这里面通过action得到的states是不一样的
 * 例如：如果一开始给tags，后面action没有给tags，则不会有原来的初始值tags,所以不用担心数据冗余。
 */

ReactDOM.render(
  <Provider store={store} >
    <Router routes={routes} history={browserHistory} />
  </Provider>,
  document.getElementById('app')
)
```
<br/>

后台呢？也要相应根据么次fetch数据返回新的store。


``` js
      const store = configureStore(initialState)
      const state = store.getState()
      const params = Object.assign(req.query, renderProps.params)

      fetchComponentDataBeforeRender(store.dispatch, renderProps.components, params)
      .then(() => {
        // 这里redux的store给Provider，再通过mapStateToProps给对应的容器组件
        const html = renderToString(
          <Provider store={store}>
            <RouterContext {...renderProps} />
          </Provider>
        )
        res.send(renderFullPage(html, store.getState()));
      })
```

有了后台传回来新的store，如何应用到前台组件呢？
<br/>

就在组件中引入connect：


``` js
import { connect } from 'react-redux'
```

``` js
class MoviesGridList extends React.Component {
... ....
  // 将store最新的state.movies给props，
  function mapStateToProps(state, props) {
    let tags = props.location.query.tags
    let movies
    if (tags === undefined) {
      movies = state.movies
    } else {
      movies = getMoviesByTag(state, tags)
    }
    return {
      tags: tags,
      movies: movies
    }
  }
}

export default connect(mapStateToProps)(MoviesGridList)
```

## 总结

----------------

在数据为王的网络世界中，数据永远都是页面的核心，那么对于数据管理的模式；

react采用的是 **单向数据流模式**，

单向也就是数据只能从一个方向流向另外一个方向而不能反过来，

如果把dom想象成一颗树，**单向数据流就是将数据自上向下的流动**，为了让数据流到尽可能多的dom中，肯定要把数据尽可能放的高一点。这里的 **根数据可以简单理解为state**。

而对于流到 **下面的数据，dom通过props接收**。这样模式就很显而易见了，尽可能高的组件对state进行更新，子组件的props也会随即更新，数据单向流动，这时候如果想通过子组件来反向更新state，就要通过上层组件传递一个函数，在函数中通过setState等方法来达到反向数据的更新。

其实到这已经可以应付一些简单的应用了，但是对于复杂的应用，组件之间的数据通信有可能是交叉而又错综复杂的，这时候就希望通过一种统一的方式将数据好好管理起来，那么出现了redux。

使用 **Redux 实现观察者模式，进行 订阅分发**；
