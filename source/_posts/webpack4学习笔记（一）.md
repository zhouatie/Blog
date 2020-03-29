---
title: webpack4学习笔记（一）
date: 2019-05-15 01:25:22
categories: 前端
tags: ['webpack']
thumbnail: https://s1.ax1x.com/2020/03/30/Geu0ns.png
---

## 前言

这是我花了几个星期学习 webpack4 的学习笔记。内容不够细，因为一些相对比较简单的，就随意带过了。希望文章能给大家带来帮助。如有错误，希望及时指出。例子都在[learn-webpack](https://github.com/zhouatie/learn-webpack)仓库上。如果你从中有所收获的话，希望你能给我的`github`点个`star`。

<!--more-->

### 小知识

`npm info webpack` 查看 webpack 的历史版本信息等
`npm init -y` 跳过那些选项，默认
全局安装的`webpack` ： `webpack index.js` 打包
项目中安装的`webpack`：`npx webpack index.js` 打包
`script`中脚本打包 ： `npm run build`
命令行中打包：`npx webpack index.js -o bundle.js` 入口是`index.js`, 出口是`bundle.js`
`webpack4`设置`mode：production` 会压缩代码 `development` 就不压缩代码
打包`output`里面`[name].js loader`中的`name`变量 其实就是`entry:{main: index.js} 中的key => main`

### source-map

`devtool: source-map`
`source-map`: `dist`文件夹里会多生成个`map`后缀文件，这样页面报错的时候，点击报错后面的地址，会跳转到代码写的地方。而不会跳转到打包后的代码里。
`inline-source-map`: 不会新生成.map 文件，会插入到打包的文件底部
`cheap-inline-source-map`: 因为`inline-source-map`报错会告诉你第几行第几个字符。前面加上`cheap`的话 只会告诉你第几行
`cheap-module-inline-source-map`: 本来 map 只会映射打包出来的 index.js 跟业务代码中的关系。第三方引入库报错映射不到。中间加了 module 这个参数就可以了。比如`loader`也会有`source-map`
开发的时候建议使用：`cheap-module-eval-source-map`,`eval`表示不会独立生成 map 文件，而是打包进代码里。
一般`development`环境用 `cheap-module-eval-source-map`
`production`环境用`cheap-module-source-map`

### loader

```javascript
// style.css
body {
    color: red;
}
```

当你想给项目添加样式的时候，`import ./style.css`导入`css`，并执行打包命令的时候。页面并报错

```javascript
ERROR in ./style.css 1:5
Module parse failed: Unexpected token (1:5)
You may need an appropriate loader to handle this file type.
```

这个时候就需要使用`loader`来编译了。
安装:`npm i style-loader css-loader -D`
为什么还需要安装`style-loader`呢？因为`style-loader`会将你的样式通过`style`标签插入到页面中
配置`webpack.config.js`

```javascript
// webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
	mode: 'development',
	entry: './index.js',
	output: {
		filename: 'bundle.js'
	},
	module: {
		rules: [
			{
				test: /\.css$/,
				use: ['style-loader', 'css-loader']
			}
		]
	},
	plugins: [new HtmlWebpackPlugin()]
};
```

上面使用了`HtmlWebpackPlugin`是因为我用它来自动生成`index.html`方便页面访问

```javascript
// package.json
"scripts": {
  "build": "webpack --config webpack.config.js"
},
```

执行`npm run build`进行打包，访问`dist`目录下的`index.html`,可以看到页面显示成功。
**`loader`执行顺序是从下到上，右到左。**

### webpack-dev-server

##### webpack-dev-server

> `webpack-dev-server`会对你的代码进行打包，将打包的内容放到内存里面，并不会自动给你打包放进 dist 里。
> `webpack —watch`页面会刷新下，内容会自动更新
> `webpack-dev-server`会自动更新当前页面
> `webpack-dev-server`, 现在的`webpack-dev-server`比以前好多了 `vue-cli3` 和`react`都是用这个了

```javascript
devServer: {
    contentBase: './dist', // 有这个就够了，
    open: true, // 自动打开浏览器
  port: 8080, // 端口不必填
  proxy: {'/api': http://localhost:3000}
}
```

##### 启动服务来热更新

`npm install express webpack-dev-middleware -D`
在`output`中添加`publicPath`

```javascript
const express = require('express');
const webpack = require('webpack');
const webpackDevMiddleWare = require('webpack-dev-middleware');
const config = require('./webpack.config.js');
const complier = webpack(config); // 帮我们做编译的东西，webpack传入config之后会申请一个编译器
app.use(
	webpackDevMiddleware(complier, {
		publicPath: config.output.publicPath // 意思是只要文件改变了，就重新运行
	})
);
const app = express();
app.listen(3000, () => {
	console.log('server is running 3000');
});
```

> 现在这样子写太麻烦了（vue-cli2 也是如此）。因为以前版本`webpack-dev-server`还不够强大，现在不一样了。非常的强大了。

### Hot Module Replacement

> 热替换，就是不会刷新整个页面。当不使用热更新的时候，操作一些功能，新增了三个元素，修改样式页面自动刷新后，刚才新增的元素会消失。如果开启了热替换，那么原先的 dom 会还在。

```javascript
const webpack = require('webpack')
    // ....
  devServer: {
    contentBase: './dist',
      open: true,
      hot: true,
      hotOnly: true // 以防hot失效后，页面被刷新，使用这个，hot失效也不刷新页面
  },
  // ...
  plugins: [
     new webpack.HotModuleReplacementPlugin()
  ]
```

```javascript
import number from './number.js';
if (module.hot) {
	// 如果热更新存在
	// 监听的文件改变，会触发后面的回调方法
	module.hot.accept('./number', () => {
		// dosomething
	});
}
```

> 为什么修改了 css 文件不需要写 module.hot。而写 js 需要写呢，因为 css-loader 已经自动帮你处理了。

### babel

#### 基本用法

将高版本的 js 代码转换成低版本的 js 代码。比如 ie 浏览器不兼容 es6，需要使用 babel 把 es6 代码把 js 转换成低版本的 js 代码。
安装：`npm install --save-dev babel-loader @babel/core`

```javascript
module: {
	rules: [{ test: /\.js$/, exclude: /node_modules/, loader: 'babel-loader' }];
}
```

`babel-loader`并不会帮助你把 es6 语法转换成低级的语法。它只是帮助打通 webpack 跟 babel 之间的联系。
转换成 es5 的语法：
安装：`npm install @babel/preset-env --save-dev`
`@babel/preset-env`包含了 es6 转换成 es5 的所有规则。

```javascript
module: {
	rules: [
		{
			test: /\.js$/,
			exclude: /node_modules/,
			loader: 'babel-loader',
			options: {
				presets: ['@babel/preset-env']
			}
		}
	];
}
```

如果还需要降到更低版本就得使用`babel-polyfill`
安装：`npm install --save @babel/polyfill`
页面顶部引入`import "@babel/polyfill";`就可以将高级版本语法转换成低级语法。但是直接`import`会让打包后的文件非常大。
这个时候就需要再配置`webpack.config.js`
**useBuiltIns**

```javascript
{
    test: /\.js$/,
    exclude: /node_modules/,
    loader: "babel-loader",
    options: {
            // "presets": ["@babel/preset-env"]
        // 当你做babel-polyfill往浏览器填充的时候，根据业务代码用到什么加什么，并不会全部写入,
          // 数组中，后面的对象是对数组前面的babel做配置
        "presets": [["@babel/preset-env", {
            useBuiltIns: 'usage'
        }]]
    }
}
```

**如果开发一个第三方库，类库。使用`babel-polyfill`会注入到全局，污染到全局环境**。
安装：`npm install --save-dev @babel/plugin-transform-runtime`
安装：`npm install --save @babel/runtime`

```javascript
{
    test: /\.js$/,
    exclude: /node_modules/,
    loader: "babel-loader",
    options: {
      "plugins": ["@babel/plugin-transform-runtime"]
    }
}
```

当你要添加配置时

```javascript
{
    test: /\.js$/,
    exclude: /node_modules/,
    loader: "babel-loader",
    options: {
      "plugins": [["@babel/plugin-transform-runtime", {
        "absoluteRuntime": false,
        "corejs": false,
        "helpers": true,
        "regenerator": true,
        "useESModules": false
      }]]
    }
}
```

如果你要将 corejs 赋值为 2
安装：`npm install --save @babel/runtime-corejs2`

```javascript
{
    test: /\.js$/,
    exclude: /node_modules/,
    loader: "babel-loader",
    options: {
      "plugins": [["@babel/plugin-transform-runtime", {
        "absoluteRuntime": false,
        "corejs": 2,
        "helpers": true,
        "regenerator": true,
        "useESModules": false
      }]]
    }
}
```

`@babel/plugin-transform-runtime`不会污染到全局环境。
当 babel 配置非常多的时候，可以将他们放到`.babelrc`文件里
在根目录下创建`.babelrc`文件
将`options`里的代码放到这个文件中，如下：

```javascript
{
  "plugins": [["@babel/plugin-transform-runtime", {
    "absoluteRuntime": false,
    "corejs": 2,
    "helpers": true,
    "regenerator": true,
    "useESModules": false
  }]]
}
```

#### react 中应用 babel

安装：`npm install --save-dev @babel/preset-react`
往刚刚的`.babelrc`文件中添加配置

```javascript
// presets对应一个数组，如果这个值需要做配置，那么这个值在包装进一个数组，放到第一项，第二项是个对象，用于描述该babel
{
    "presets": [
        ["@babel/preset-env", {
            "useBuiltIns": "usage"
        }],
        "@babel/preset-react"
    ]
}
```

**注意：转换是先下后上，就是使用 preset-react 转换 react 代码，再用 preset-env 将 js 代码转换为 es5 代码**
