---
title: webpack4学习笔记（四）
date: 2019-05-15 01:29:12
categories: 前端
tags: ['webpack']
thumbnail: https://s1.ax1x.com/2020/03/30/Geu0ns.png
---

## 前言

这是我花了几个星期学习 webpack4 的学习笔记。内容不够细，因为一些相对比较简单的，就随意带过了。希望文章能给大家带来帮助。如有错误，希望及时指出。例子都在[learn-webpack](https://github.com/zhouatie/learn-webpack)仓库上。如果你从中有所收获的话，希望你能给我的`github`点个`star`。

<!--more-->

### 编写 loader

```javascript
// index.js
console.log('hello, atie');
```

配置`webpack.config.js`

```javascript
// webpack.config.js

module: {
  rules: [
    {
      test: /\.js$/,
      include: /src/,
      loader: path.resolve(__dirname, './loaders/replaceLoader.js')
    }
  ]
},
```

```javascript
// 函数不能使用箭头函数
module.exports = function(source) {
	console.log(source, 'source');
	return source.replace('atie', 'world');
};
```

`loader`文件其实就是导出一个函数，`source`就是`webpack`打包出的`js`字符串。这里的`loader`就是将上面的`console.log('hello, atie')`替换为`console.log('hello, world')`

打包下代码，不出所料。控制台就会打印出`hello, world`

当你想要给 loader 传参时，可配置如下

```javascript
module: {
  rules: [
    {
      test: /\.js$/,
      include: /src/,
      use: [{
        loader: path.resolve(__dirname, './loaders/replaceLoader.js'),
        options: {
          name: 'haha'
        }
      }]
    }
  ]
},
```

通过给`loader`添加`options`

这样`loader`中就可以通过`this.query`获取该参数了

```javascript
module.exports = function(source) {
	// 控制台输出：console.log('hello atie') { name: 'haha' } source
	console.log(source, this.query, 'source');
	return source.replace('atie', 'world');
};
```

当然变量不一定非要通过`this.query`来获取

可通过`loader-utils`这个包来获取传入的变量

安装: `npm i loader-utils -D`

```javascript
const loaderUtils = require('loader-utils');

// 函数不能使用箭头函数
module.exports = function(source) {
	// console.log(source, this.query, 'source')
	const options = loaderUtils.getOptions(this);
	console.log(options, 'options'); // { name: 'haha' } 'options'
	return source.replace('atie', 'world');
};
```

打印出来的与上面`this.query`一致

上面都是直接通过`return`返回的，那么我们还有没有其他方法返回`loader`翻译后的代码呢？`

这里就会用到`callback`

```javascript
this.callback(
  err: Error | null,
  content: string | Buffer,
  sourceMap?: SourceMap,
  meta?: any
);
```

上面的代码就可以改写成

```javascript
module.exports = function(source) {
	const options = loaderUtils.getOptions(this);
	const result = source.replace('atie', options.name);
	this.callback(null, result);
};
```

`callback`优势在于它可以传递多余的参数

```javascript
module.exports = function(source) {
	setTimeout(() => {
		return source.replace('atie', 'world');
	}, 1000);
};
```

当我们把`return`包到异步方法里，打包的时候就会报错，那么我们该怎么办呢？

这个时候就需要用到`this.async()`

```javascript
module.exports = function(source) {
	const callback = this.async();
	setTimeout(() => {
		callback(null, source.replace('atie', 'world'));
	}, 2000);
};
```

通过调用`this.async()`返回的`callback`方法来返回结果

**use 中的 loader 执行顺序，先右后左，先下后上**

### 编写 plugin

在根目录下新建`plugins`文件夹，并新建`copyright-webpack-plugin.js`，内容如下：

```javascript
class Copyright {
	constructor() {
		console.log('this is plugin');
	}
	apply(compiler) {}
}
module.exports = Copyright;
```

注意：apply 这个方法必须存在，不然插件被执行的时候会报错。

配置`webpack.config.js`,如下：

```javascript
const Copyrgiht = require('./plugins/copyright-webpack-plugin.js')

...

plugins: [
  new Copyrgiht()
]
```

执行下打包命令后

```javascript
this is plugin
Hash: 479baeba2207182096f8
Version: webpack 4.30.0
Time: 615ms
Built at: 2019-05-08 23:05:08
     Asset       Size  Chunks             Chunk Names
 bundle.js   3.77 KiB    main  [emitted]  main
index.html  182 bytes          [emitted]
```

控制台打印出了`this is plugin`

接下来，我们继续探索插件的奥秘

在使用插件的时候还可以传参

```javascript
// webpack.config.js
plugins: [
	new Copyrgiht({
		name: 'atie'
	})
];
```

```javascript
class Copyright {
	constructor(options) {
		// console.log(options, 'this is plugin')
		this.options = options;
	}
	apply(compiler) {
		console.log(this.options);
	}
}
```

执行下打包命令：

```javascript
{ name: 'atie' }
Hash: 479baeba2207182096f8
Version: webpack 4.30.0
Time: 742ms
Built at: 2019-05-08 23:24:10
     Asset       Size  Chunks             Chunk Names
 bundle.js   3.77 KiB    main  [emitted]  main
index.html  182 bytes          [emitted]
```

控制就会输出 `{name: 'atie'}`

`webpack`在调用`apply`会传递一个`compiler`参数，这个参数可以做很多事情，具体可以参考`webpack`官网

这里介绍下钩子

```javascript
class Copyright {
	apply(compiler) {
		compiler.hooks.emit.tapAsync('Copyright', (compilation, callback) => {
			console.log(compilation.assets, '以具有延迟的异步方式触及 run 钩子。');
			compilation.assets['copyright.txt'] = {
				source: function() {
					return 'copyright by atie';
				},
				size: function() {
					return 17;
				}
			};
			callback();
		});
	}
}

module.exports = Copyright;
```

该钩子是在文件生成前触发的。我们在文件生成前，在`asset`对象上在加一个文件对象

打包之后

```javascript
.
├── bundle.js
├── copyright.txt
└── index.html
```

可以看到多了一个`copyright.txt`，也就是我们上面创建的文件。点开该文件还会看到里面的内容正是`copyright by atie`
