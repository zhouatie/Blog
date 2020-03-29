---
title: webpack4学习笔记（二）
date: 2019-05-15 01:27:04
categories: 前端
tags: ['webpack']
thumbnail: https://s1.ax1x.com/2020/03/30/Geu0ns.png
---

## 前言

这是我花了几个星期学习 webpack4 的学习笔记。内容不够细，因为一些相对比较简单的，就随意带过了。希望文章能给大家带来帮助。如有错误，希望及时指出。例子都在[learn-webpack](https://github.com/zhouatie/learn-webpack)仓库上。如果你从中有所收获的话，希望你能给我的`github`点个`star`。

<!--more-->

### tree shaking

一个模块里会导出很多东西。把一个模块里没有被用到的东西都给去掉。不会把他打包到入口文件里。tree shaking 只支持 es6 的方式引入(`import`)，使用`require`无法使用`tree shaking`。

`webpack`的`development`无法使用`tree shaking`功能。除非在打包的配置里加上

```javascript
// 开发环境需要加如下代码
optimization: {
	usedExports: true;
}
```

当你需要 import 某个模块，又不想`tree shaking`把他给干掉，就需要在 package.json 里修改`sideEffects`参数。比如当你`import './console.js'`, `import './index.css'`等没有`export`(导出)模块的文件。又不想`tree shaking`把它干掉。

```javascript
// package.json
sideEffects: ['./console.js', './index.css'];
// 反之
sideEffects: false;
```

**在`development`环境即使你使用`tree shaking`，它也不会把其他多余的代码给干掉。他只会在打包的文件里注明某段代码是不被使用的。**

### `development` 和 `production` 区别

`development`代码不压缩，`production`代码会压缩

省略…☺

`webpack-merge`

`react`和`vue`都会区分环境进行不同的`webpack`配置,但是它们一定会有相同的部分。这个时候需要通过使用`webpack-merge`进行抽离。

```javascript
// webpack.base.config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
	mode: 'production',
	// mode: 'development',
	entry: './index.js',
	output: {
		filename: 'bundle.js',
		path: path.resolve(__dirname, './dist')
	},
	module: {
		rules: [
			{
				test: /\.css$/,
				use: ['style-loader', 'css-loader']
			}
		]
	},
	optimization: {
		usedExports: true
	},
	plugins: [
		new HtmlWebpackPlugin({
			template: './index.html'
		})
	]
};

// webpack.dev.config.js
const merge = require('webpack-merge');
const baseConfig = require('./webpack.base.config');
const devConfig = {
	mode: 'development'
};

module.exports = merge(baseConfig, devConfig);
```

这里就不重复把`production`环境在配置出来了，主要介绍下`webpack-merge`用法。

- 安装`npm i webpack-merge -D`

- 新建一个公共的文件如：`webpack.base.config.js`

- 将`development`和`production`两个`webpack`配置相同的抽离到`webpack.base.config.js`文件中

- 在环境配置文件中(具体代码如上)

  - `const merge = require('webpack-merge')`

  - `const baseConfig = require('./webpack.base.config.js')`

  - `module.exports = merge(baseConfig, devConfig)`

### `code splitting`和`splitChunks`

当你把所有的代码都打包到一个文件的时候，每次改一个代码都需要重新打包。且用户都要重新加载下这个 js 文件。但是如果你把一些公共的代码或第三方库抽离并单独打包。通过缓存加载，会加快页面的加载速度。

1. 异步加载的代码，webpack 会单独打包到一个 js 文件中
2. 同步加载的代码有两种方式

原始代码

```javascript
import _ from 'lodash';

console.log(666);
```

打包后的文件：

`main.js 551 KiB main [emitted] main`
可以看到，webpack 将业务代码跟 lodash 库打包到一个 main.js 文件了

方法一：

创建一个新文件

```javascript
import _ from 'lodash';
window._ = _;
```

将文件挂载到`window`对象上,这样其他地方就可以直接使用了。

然后在 webpack 配置文件中的 entry 增加一个入口为该文件。让该文件单独打包。

```javascript
    Asset      Size  Chunks             Chunk Names
lodash.js   551 KiB  lodash  [emitted]  lodash
  main.js  3.79 KiB    main  [emitted]  main
```

方法二：

通过添加`optimization`配置参数

`optimization`: 会将诸如`lodash`等库抽离成单独的`chunk`,还会将多个模块公用的模块抽离成单独的`chunk`

```javascript
optimization: {
  splitChunks: {
    chunks: 'all'
  }
},
```

打包后文件：

```javascript
          Asset      Size        Chunks             Chunk Names
        main.js  6.78 KiB          main  [emitted]  main
vendors~main.js   547 KiB  vendors~main  [emitted]  vendors~main
```

可以看到，webpack 将 lodash 抽成公共的 chunk 打包出来了。

`splitChunks`里面还可以在添加个参数`cacheGroups`

```javascript
optimization: {
    splitChunks: {
        chunks: 'all',
        cacheGroups: {
          	// 下面的意思是：将从node_modules中引入的模块统一打包到一个vendors.js文件中
            vendors: {
                test: /[\\/]node_modules[\\/]/,
                priority: -10,
                filename: 'vendors.js'
            },
            default: false
        }
    }
}
```

`cacheGroups`中`vendors`配置表示将从`node_modules`中引入的模块统一打包到一个 vendors.js 文件中

`splitChunks`的`vendors`的`default`参数：

根据上下文来解释，如上配置了`vendors`，打包`node_modules`文件夹中的模块，

那么`default`将会打包自己编写的公共方法。

当不使用`default`配置时。

```javascript
     Asset     Size             Chunks             Chunk Names
   main.js  315 KiB               main  [emitted]  main
   test.js  315 KiB               test  [emitted]  test
```

添加如下配置之后：

```javascript
optimization: {
    splitChunks: {
        chunks: 'all',
        cacheGroups: {
          	// 下面的意思是：将从node_modules中引入的模块统一打包到一个vendors.js文件中
            vendors: {
                test: /[\\/]node_modules[\\/]/,
                priority: -10,
                filename: 'vendors.js'
            },
            // 打包除上面vendors以外的公共模块
            default: {
                priority: -20,
                reuseExistingChunks: true, // 如果该chunk已经被打包进其他模块了，这里就复用了，不打包进common.js了
                filename: 'common.js'
            }
        }
    }
}
```

打包后的文件体积为

```javascript
     Asset      Size             Chunks             Chunk Names
 common.js   308 KiB  default~main~test  [emitted]  default~main~test
   main.js  7.03 KiB               main  [emitted]  main
   test.js  7.02 KiB               test  [emitted]  test
```

**配置说明**

```js
splitChunks: {
  chunk: 'all', // all(全部)， async(异步的模块)，initial(同步的模块)
  minSize: 3000, // 表示文件大于3000k的时候才对他进行打包
  maxSize: 0,
  minChunks: 1, // 当某个模块满足minChunks引用次数时，才会被打包。例如,lodash只被一个文件import，那么lodash就不会被code splitting,lodash将会被打包进 被引入的那个文件中。如果满足minChunks引用次数，lodash会被单独抽离出来，打出一个chunk。
  maxAsyncRequests: 5, // 在打包某个模块的时候，最多分成5个chunk，多余的会合到最后一个chunk中。这里分析下这个属性过大过小带来的问题。当设置的过大时，模块被拆的太细，造成并发请求太多。影响性能。当设置过小时，比如1，公共模块无法被抽离成公共的chunk。每个打包出来的模块都会有公共chunk
  automaticNameDelimiter: '~', // 当vendors或者default中的filename不填时，打包出来的文件名就会带~
  name: true,
  cashGroups: {} // 如上
}
```

[maxAsyncRequests](https://www.jianshu.com/p/91e1082bce20)

### Lazy Loading

异步`import`的包会被单独打成一个`chunk`

```javascript
async function getComponent() {
	const { default: _ } = await import(/* webpackChunkNanem:'lodash */ 'lodash');
	const element = document.createElement('div');
	element.innerHTML = _.join(['Dell', 'Lee'], '-');
	return element;
}
document.addEventListener('click', () => {
	getComponent().then(element => {
		document.body.appendChild(element);
	});
});
```

[lazy loading](https://webpack.js.org/guides/lazy-loading)

### chunk

每一个`js`文件都是一个`chunk`

`chunk`是使用`Webpack`过程中最重要的几个概念之一。在 Webpack 打包机制中，编译的文件包括 entry（入口，可以是一个或者多个资源合并而成，由 html 通过 script 标签引入）和 chunk（被 entry 所依赖的额外的代码块，同样可以包含一个或者多个文件）。从页面加速的角度来讲，我们应该尽可能将所有的 js 打包到一个 bundle.js 之中，但是总会有一些功能是使用过程中才会用到的。出于性能优化的需要，对于这部分资源我们可以做成按需加载。

### 打包分析

打包分析：

安装：`npm install --save-dev webpack-bundle-analyzer`

```javascript
// package.json => scripts

"analyz": "NODE_ENV=production npm_config_report=true npm run build"
```

```javascript
// webpack.config.js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer')
	.BundleAnalyzerPlugin;

plugins: [new BundleAnalyzerPlugin()];
```

执行命令`npm run analyz`

浏览器就会自动打开`localhost:8888`，分析图就会展现在你眼前

非常清晰直观的看出

![image-20190421142354243](/Users/zhouatie/Library/Application Support/typora-user-images/image-20190421142354243.png)

### CSS 文件的代码分割

我们之前写的 css 文件都会被打包进 js 文件中，要想把 css 单独打包成一个 css 文件该怎么做呢？

这个时候就需要用到`MiniCssExtractPlugin`

开发环境用不到这个功能，一般都是用在生产环境中。

安装：`npm install --save-dev mini-css-extract-plugin`

```javascript
// webpack.config.js
const MiniCssExtractPlugin = require('mini-css-extract-plugin')

module: {
  rules: [
    {
      test: /\.css$/,
      use: [{
        loader: MiniCssExtractPlugin.loader,
        options: {
          // 可以在此处指定publicPath
          // 默认情况下，它在webpackoptions.output中使用publicPath
          publicPath: '../',
          // hmr: process.env.NODE_ENV === 'development',
        },
      }, 'css-loader']
    }
  ]
},
plugins: [
  new MiniCssExtractPlugin({
    // 与webpackoptions.output中相同选项类似的选项
    // 两个选项都是可选的
    filename: '[name].css',
    chunkFilename: '[id].css',
  }),
]

// index.js
import './index.css';
console.log('haha')

// index.css
body {
    background: green;
}
```

这样打包之后，css 会被单独打包成一个 css 文件。

### 缓存

目前为止，我们每次修改内容，打包出去后的文件名都不变。线上环境的文件是有缓存的。所以当你文件名不变的话，更新内容打包上线。有缓存的电脑就无法获取到最新的代码。

这个时候我们就会用到`contenthash`

我们先记录配置`contenthash`之前打包的文件名。

```javascript
     Asset       Size  Chunks             Chunk Names
index.html  180 bytes          [emitted]
   main.js   3.81 KiB    main  [emitted]  main
```

接下来我们来配置下`contenthash` (就是根据你文件内容生成的 hash 值)

```javascript
// webpack.config.js
output: {
  path: path.resolve(__dirname, '../dist'),
  filename: '[name][contenthash].js'
},
```

打包完之后会在`main`后面接上`hash`值。

```javascript
                      Asset       Size  Chunks             Chunk Names
                 index.html  200 bytes          [emitted]
mainf5faa2d3d1e119256290.js   3.81 KiB    main  [emitted]  main
```

当你不更新内容重新打包后，`contenthash`还会维持不变。所以线上用户访问的时候就不会去服务器重新拿取代码，而是从缓存中取文件。

#### `shimming` (预置依赖)

以`jquery`为例，代码如下

```javascript
// index.js
import $ from 'jquery';
$('body').html('HHAHAH');
import func from './test.js';
func();

// test.js
export default function func() {
	$('body').append('<h1>2</h1>');
}
```

当你不在 test.js 中引入`import $ from 'jquery'`

那么浏览器访问的时候，会报

`test.js:5 Uncaught ReferenceError: $ is not defined`

这个时候就需要使用垫片功能

```javascript
const webpack = require('webpack');

plugins: [
	new webpack.ProvidePlugin({
		$: 'jquery'
	})
];
```

当你加上这段代码后，模块在打包的时候，发现你使用了`$`就会在你模块顶部自动加入`import $ from 'jquery'`

**其他关于`shimming`的内容参考`webpack`官网 [shimming](https://webpack.js.org/guides/shimming)**
