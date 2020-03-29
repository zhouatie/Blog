---
title: webpack4学习笔记（三）
date: 2019-05-15 01:28:10
categories: 前端
tags: ['webpack']
thumbnail: https://s1.ax1x.com/2020/03/30/Geu0ns.png
---

## 前言

这是我花了几个星期学习 webpack4 的学习笔记。内容不够细，因为一些相对比较简单的，就随意带过了。希望文章能给大家带来帮助。如有错误，希望及时指出。例子都在[learn-webpack](https://github.com/zhouatie/learn-webpack)仓库上。如果你从中有所收获的话，希望你能给我的`github`点个`star`。

<!-- more -->

### library

当你要开发第三方库供别人使用时，就需要用到`library`和`libraryTarget`这两个配置了。

**`library`**

```javascript
output: {
    filename: 'library.js',
    library: 'library',
    libraryTarget: 'umd'
},
```

`library`: 当配置了这个`library`参数后，会把`library`这个`key`对应的`value`即上面代码`library`挂载到全局作用域中。`html`用`script`标签引入，可以通过访问全局变量`library`访问到我们自己开发的库。

`libraryTarget`:这个默认值为`var`。意思就是让 library 定义的变量挂载到全局作用域下。当然还有浏览器环境的`window`,`node`环境的`global`,`umd`等。当设置了`window`、`global`,`library`就会挂载到这两个对象上。当配置了`umd`后，你就可以通过`import`,`require`等方式引入了。

**`externals`**

`exterals`是开发公共库很重要的一个配置。当你的公共库引入了第三方库的时候，公共库会把该第三方库也打包进你的模块里。当使用者也引入了这个第三方库后，这个时候打包就会又打了一份第三方库进来。

所在在公共模块库中配置如下代码

```javascript
externals: {
    // 前面的lodash是我的库里引入的包名 比如 import _ from 'lodash'
    // 后面的lodash是别人业务代码需要注入到他自己模块的lodash 比如 import lodash from 'lodash',注意不能import _ from 'lodash',因为配置项写了lodash 就不能import _
    lodash: 'lodash'
},
```

**前面的`lodash`是我的库里引入的包名 比如`import _ from 'lodash'`,后面的`lodash`是别人业务代码需要注入到他自己模块的`lodash` 比如 `import lodash from 'lodash'`,注意不能`import _ from 'lodash'`,因为配置项写了`lodash` 就不能`import _`。**

本人做了个试验，当自己开发的包不引入`lodash`,业务代码中也不引入`lodash`,那么打包业务代码的时候，`webpack`会把`lodash`打进你业务代码包里。

当然`externals`,配置还有多种写法，如下

```javascript
externals: {
    lodash: {
        commonjs: 'lodash',
        commonjs2: 'lodash',
        amd: 'lodash',
        root: '_'
    }
}

externals: ['lodash', 'jquery']

externals: 'lodash'
```

具体请参考官网[externals](https://webpack.js.org/configuration/externals#externals)

### 发布自己开发的 npm 包

学了上面的配置后，就需要学习下如何将自己的包发布到`npm`仓库上了。

- `package.json` 的入口要改成`dist`目录下的 js 文件如： `"main": "./dist/library.js"`

- 注册 npm 账号。npm 会发送一份邮件到你的邮箱上，点击下里面的链接进行激活。
- 命令行输入`npm login` 进行登录，或者`npm adduser` 添加账号
- `npm publish`

当出现如下提示代表发布成功

```javascript
// 当出现类似如下代码时，表示你已经发布成功
➜  library git:(master) ✗ npm publish
+ atie-first-module-library@1.0.0
```

遇到的问题：

当你遇到`npm ERR! you must verify your email before publishing a new package`说明你还没有激活你的邮箱，去邮箱里点击下链接激活下就 ok 了

当你已经登录了提醒`npm ERR! 404 unauthorized Login first`,这个时候你就要注意下你的`npm`源了，看看是否设置了淘宝源等。记得设置回来`npm config set registry https://registry.npmjs.org/`

### PWA

http-server

workbox-webpack-plugin

相信很多朋友都有耳闻过`PWA`这门技术,`PWA`是`Progressive Web App`的英文缩写， 翻译过来就是渐进式增强 WEB 应用， 是 Google 在 2016 年提出的概念，2017 年落地的 web 技术。目的就是在移动端利用提供的标准化框架，在网页应用中实现和原生应用相近的用户体验的渐进式网页应用。

优点：

1. **可靠** 即使在不稳定的网络环境下，也能瞬间加载并展现
2. **快** 快速响应，并且 动画平滑流畅

应用场景：

当你访问正常运行的服务器页面的时候，页面正常加载。可当你服务器挂了的时候，页面就无法正常加载了。

这个时候就需要使用到 pwa 技术了。

这里我编写最简单的代码重现下场景：

```javascript
// webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin')
const CleanWebpackPlugin = require('clean-webpack-plugin')

module.exports = {
    mode: 'production',
    entry: './index.js',
    output: {
        filename: 'bundle.js'
    },
    plugins: [
        new CleanWebpackPlugin(),
        new HtmlWebpackPlugin()
    ]
}

// index.js
console.log('this is outer console')

// package.json
{
  "name": "PWA",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "build": "webpack --config webpack.config.js",
    "start": "http-server ./dist"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "clean-webpack-plugin": "^2.0.1",
    "html-webpack-plugin": "^3.2.0",
    "http-server": "^0.11.1",
    "webpack": "^4.30.0",
    "webpack-cli": "^3.3.1",
  }
}

```

执行下`npm run build`

```javascript
.
├── bundle.js
└── index.html
```

为了模拟服务器环境，我们安装下`http-server`

`npm i http-server -D`

配置下`package.json`，`"start": "http-server ./dist"`

执行`npm run start`来启动 dist 文件夹下的页面

这个时候控制台会正常打印出`'this is outer console'`

当我们断开`http-server`服务后，在访问该页面时，页面就报 404 了

这个时候就需要使用到 pwa 技术了

使用步骤：

安装： `npm i workbox-webpack-plugin -D`

webpack 配置文件配置：

```javascript
// webpack.config.js
const { GenerateSW } = require('workbox-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const CleanWebpackPlugin = require('clean-webpack-plugin');

module.exports = {
	mode: 'production',
	entry: './index.js',
	output: {
		filename: 'bundle.js'
	},
	plugins: [
		new CleanWebpackPlugin(),
		new HtmlWebpackPlugin(),
		new GenerateSW({
			skipWaiting: true, // 强制等待中的 Service Worker 被激活
			clientsClaim: true // Service Worker 被激活后使其立即获得页面控制权
		})
	]
};
```

这里我们写一个最简单的业务代码，在注册完 pwa 之后打印下内容：

```javascript
// index.js
console.log('this is outer console');

// 进行 service-wroker 注册
if ('serviceWorker' in navigator) {
	window.addEventListener('load', () => {
		navigator.serviceWorker
			.register('./service-worker.js')
			.then(registration => {
				console.log('====== this is inner console ======');
				console.log('SW registered: ', registration);
			})
			.catch(registrationError => {
				console.log('SW registration failed: ', registrationError);
			});
	});
}
```

执行下打包命令：`npm run build`

```javascript
.
├── bundle.js
├── index.html
├── precache-manifest.e21ef01e9492a8310f54438fcd8b1aad.js
└── service-worker.js
```

打包之后会生成个`service-worker.js`与`precache-manifest.e21ef01e9492a8310f54438fcd8b1aad.js`

接下来再重启下`http-server`服务：`npm run start`

页面将会打印出

```javascript
this is outer console
====== this is inner console ======
...
```

然后我们再断开`http-server`服务

刷新下页面，竟然打印出了相同的代码。说明 pwa 离线缓存成功。

### typescript

使用 webpack 打包 ts 文件，就需要安装`ts-loader`

安装：`npm i ts-loader typescript -D`

`webpack.config.js`文件中添加解析`typescript`代码的`loader`

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin');
const CleanWebpackPlugin = require('clean-webpack-plugin');

module.exports = {
	mode: 'production',
	entry: './src/index.ts',
	output: {
		filename: 'bundle.js'
	},
	module: {
		rules: [
			{
				test: /\.ts$/,
				loader: 'ts-loader',
				exclude: /node_modules/
			}
		]
	},
	plugins: [new CleanWebpackPlugin(), new HtmlWebpackPlugin()]
};
```

配置了`webpack.config.js`还不行，还得在根目录文件下新增个`.tsconfig.json`文件

```javascript
{
    "compilerOptions": {
        "outDir": "./dist/", // 默认解析后的文件输出位置
        "noImplicitAny": true, // 存在隐式 any 时抛错
        "module": "es6", // 表示这是一个es6模块机制
        "target": "es5", // 表示要讲ts代码转成es5代码
        "allowJs": true // 表示允许引入js文件。TS 文件指拓展名为 .ts、.tsx 或 .d.ts 的文件。如果开启了 allowJs 选项，那 .js 和 .jsx 文件也属于 TS 文件
    }
}
```

新建`index.ts`

```javascript
class Greeter {
	greeting: string;
	constructor(message: string) {
		this.greeting = message;
	}
	greet() {
		return 'Hello, ' + this.greeting;
	}
}

let greeter = new Greeter('world');

let button = document.createElement('button');
button.textContent = 'Say Hello';
button.onclick = function() {
	alert(greeter.greet());
};

document.body.appendChild(button);
```

执行打包命令，访问打包后的页面，页面正常执行。

当需要使用`lodash`等库时，

需安装：`npm i @types/lodash -D`

修改页面代码 引入 `lodash`

```javascript
import * as _ from 'lodash';

class Greeter {
	greeting: string;
	constructor(message: string) {
		this.greeting = message;
	}
	greet() {
		return 'Hello, ' + this.greeting;
	}
}

let greeter = new Greeter('world');

let button = document.createElement('button');
button.textContent = 'Say Hello';
button.onclick = function() {
	alert(_.join(['lodash', greeter.greet()], '-'));
};

document.body.appendChild(button);
```

**提醒：ts 使用的包，可通过`https://microsoft.github.io/TypeSearch` 这个网址去查对应的包使用指南**

### 使用`WebpackDevServer`实现请求转发

当我们工作本地开发某一个需求的时候，需要将这块需求的请求地址转发到某个后端同事的本地服务器或某个服务器上，就需要用到代理。然后其他页面的请求都走测试环境的请求。那么我们该怎样拦截某个请求，并将其转发到我们想要转发的接口上呢？

这个时候就需要用到`webpack-dev-server`

主要看`devServer`配置：

```javascript
// webpack.config.js
const CleanWebpackPlugin = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
	mode: 'development',
	entry: './index.js',
	output: {
		filename: 'bundle.js'
	},
	devServer: {
		contentBase: './dist',
		open: true,
		hot: true
	},
	plugins: [new HtmlWebpackPlugin(), new CleanWebpackPlugin()]
};
```

```javascript
// package.json

scripts: {
  "server": "webpack-dev-server"
}
```

```javascript
// index.js
import axios from 'axios';

const div = document.createElement('div');
div.innerHTML = 'hahahha';
div.addEventListener('click', () => {
	alert('hahah');
	axios.get('/list').then(res => {
		console.log(res);
	});
});

document.body.appendChild(div);
```

在写一个本地启动的服务端代码

```javascript
const express = require('express');
const app = express();

app.get('/api/list', (req, res) => {
	res.json({
		success: true
	});
});

app.listen(8888, () => {
	console.log('listening localhost:8888');
});
```

执行`npm run server`命令，浏览器会自动打开页面。点击 div 后，会发起请求。

浏览器提示`http://localhost:8080/api/list 404 (Not Found)`,表示该接口不存在。

因为我们`webpack`启动静态资源服务器默认端口为 8080，所以他求会直接请求到 8080 的/api/list 接口。所以会提示找不到该接口。

为了解决这个问题，我们就需要将该请求从 8080 端口代理到 8888 端口(也就是我们自己本地启动的服务)

配置`webpack.config.js`

这里我只展示`devServer`代码

```javascript
// webpack.config.js
devServer: {
    contentBase: './dist',
    open: true,
    hot: true,
    proxy: {
        '/api': {
            target: 'http://localhost:8888'
        }
    }
},
```

配置`devServer`的`proxy`字段的参数，将请求`/api`开头的请求都转发到`http://localhost:8888`,

通过这个方法可以解决一开始提到的本地开发的时候，只想把部分接口转发到某台部署新需求的服务器上。比如当你这个项目请求很多，不同接口部署在不同的端口或者不同的服务器上。那么就可以通过配置**第一个路径**，来区分不同的模块。并转发到不同的服务上。如：

```javascript
// webpack.config.js
devServer: {
    contentBase: './dist',
    open: true,
    hot: true,
    proxy: {
        '/module1': {
            target: 'http://localhost:8887'
        },
        '/module2': {
            target: 'http://localhost:8888'
        },
        '/module3': {
            target: 'http://localhost:8889'
        }
    }
},
```

当你要代理到某个 https 的接口上，就需要设置`secure: false`

```javascript
// webpack.config.js
devServer: {
    proxy: {
        '/api': {
            target: 'https://other-server.example.com',
            secure: false
        }
    }
}
```

```javascript
target: ''， // 代理的目标地址
secure: false, // 请求https的需要设置
changeOrigin: true,  // 跨域的时候需要设置
headers: {
  host: 'http://www.baidu.com', //修改请求域名
  cookie: ''
}
...
```

其他关于`devServer`的配置详见[devServer](https://webpack.js.org/configuration/dev-server#devserverproxy)

### WebpackDevServer 解决单页面路由 404 问题

相信大家都是开发过 vue 或者 react 单页面带路由的应用。这里就忽略业务代码，介绍下`devServer`的`historyApiFallback`参数

```javascript
devServer: {
  historyApiFallback: true, // 当设置为true时，切换路由就不会出现路由404的问题了
}
```

详见[historyApiFallback](https://webpack.js.org/configuration/dev-server#devserverhistoryapifallback)

### eslint

安装`eslint`: `npm i eslint -D`

目录下新建`.eslintrc.json`文件。

`environment`: 指定脚本的运行环境

`globals`: 脚本在执行期间访问的额外全局变量。

`rules`: 启动的规则及其各自的错误级别。

`解析器选项`: 解析器选项

编辑你的`eslint`的规则

```javascript
{
    "parserOptions": {
        "ecmaVersion": 6,
        "sourceType": "module",
        "ecmaFeatures": {
            "jsx": true
        }
    },
    "rules": {
        "semi": 2
    }
}
```

`vscode`安装`eslint`插件。

配置下`webpack.config.js`配置。

```javascript
...
devServer: {
    overlay: true,
    contentBase: './dist',
    hot: true,
    open: true
},
module: {
    rules: [{
        test: /\.js$/,
      	exclude: /node_modules/,
        use: ['eslint-loader']
    }]
}
...
```

`eslint-loader`是用于检查`js`代码是否符合`eslint`规则。

这里`devServer`中的`overlay`的作用是，当你 eslint 报错的时候，页面会有个报错蒙层。这样就不局限于编辑器（vscode)的报错提醒了。

如果 js 代码使用了多个 loader，那么 eslint-loader 一定要写在最右边。如果不写在最后一个的话，需在里面添加`enforce: "pre"`,这样不管写在哪个位置都会优先使用`eslint-loader`校验下代码规范。

```javascript
{
    loader: 'eslint-loader',
    options: {
        enforce: "pre",
    }
}
```

### 提升`webpack`打包速度的方法

#### 1. 跟上技术的迭代

- 升级`webpack`版本 `node`版本`npm`等版本

#### 2. 尽可能少的模块上应用`loader`

- `include` `exclude`

#### 3. 尽可能少的使用`plugin`

#### 4. `resolve`

```javascript
resolve: {
    extensions: ['.js'],
    alias: {
        'src': path.resolve(__dirname, '../src')
    }
}
```

`extensions`: 可以让你 import 模块的时候不写格式，当你不写格式的时候，webpack 会默认通过 extensions 中的格式去相应的文件夹中找

`alias`:当你`import`的路径很长的时候，最好使用别名，能简化你的路径。

比如：`import index.js from '../src/a/b/c/index.js'`

设置别名：

```javascript
resolve: {
    alias: {
        '@c': path.resolve(__dirname, '../src/a/b/c')
    }
}
```

这样你的`import`导入代码就可以改成`import index.js from '@c/index.js'`

#### 5. dllPlugin

我们先记录下不使用`dll`打包时间`787ms`：

```javascript
Time: 787ms
Built at: 2019-05-04 18:32:29
     Asset       Size  Chunks             Chunk Names
 bundle.js    861 KiB    main  [emitted]  main
index.html  396 bytes          [emitted]
```

接下来我们就尝试使用`dll`技术

我们先配置一个用于打包`dll`文件的`webpack`配置文件，生成打包后的`js`文件与描述动态链接库的`manifest.json`

```javascript
// webpack.dll.config.js
const path = require('path');
const webpack = require('webpack');

module.exports = {
	entry: {
		vendor: ['jquery', 'lodash'] // 要打包进vendor的第三方库
	},
	output: {
		filename: '[name].dll.js', // 打包后的文件名
		path: path.resolve(__dirname, './dll'), // 打包后存储的位置
		library: '[name]_[hash]' // 挂载到全局变量的变量名，这里要注意 这里的library一定要与DllPlugin中的name一致
	},
	plugins: [
		new webpack.DllPlugin({
			// 用于打包出一个个单独的动态链接库文件
			name: '[name]_[hash]', // 引用output打包出的模块变量名，切记这里必须与output.library一致
			path: path.join(__dirname, './dll', '[name].manifest.json') // 描述动态链接库的 manifest.json 文件输出时的文件名称
		})
	]
};
```

**重点：这里引入的 DllPlugin 插件，该插件将生成一个 manifest.json 文件，该文件供 webpack.config.js 中加入的 DllReferencePlugin 使用，使我们所编写的源文件能正确地访问到我们所需要的静态资源（运行时依赖包）。**

配置下`package.json`文件的`scripts`: `"build:dll": "webpack --config webpack.dll.config.js"`

执行下 `npm run build:dll`

```javascript
Time: 548ms
Built at: 2019-05-04 18:54:09
        Asset     Size  Chunks             Chunk Names
vendor.dll.js  157 KiB       0  [emitted]  vendor
```

除了打包出`dll`文件之外，还得再主`webpack`配置文件中引入。这里就需要使用到`DllReferencePlugin`。具体配置如下：

```javascript
// webpack.config.js
new webpack.DllReferencePlugin({
  manifest: require('./dll/vendor.manifest.json')
}),
```

这里的`manifest`：需要配置的是你`dllPlugin`打包出来的`manifest.json`文件。让主`webpack`配置文件通过这个

描述动态链接库`manifest.json`文件，让`js`导入该模块的时候，直接引用`dll`文件夹中打包好的模块。

看似都配置好了，接下来执行下命令 `npm run build`

使用`dll`打包后时间：

```javascript
Time: 471ms
Built at: 2019-05-04 18:19:49
     Asset       Size  Chunks             Chunk Names
 bundle.js   6.43 KiB    main  [emitted]  main
index.html  182 bytes          [emitted]
```

**直接从最开始的`787ms`降低到`471ms`，当你抽离的第三方模块越多，这个效果就越明显。**

浏览器跑下`html`页面，会报错

`Uncaught ReferenceError: vendor_e406fbc5b0a0acb4f4e9 is not defined`

这是因为`index.html`还需要通过`script`标签引入这个`dll`打包出来的`js`文件

我们如果每次自己手动引入的话会比较麻烦，如果`dll`文件非常多的话，就难以想象了。

这个时候就需要借助`add-asset-html-webpack-plugin`这个包了。

```javascript
const AddAssetHtmlPlugin = require('add-asset-html-webpack-plugin');

new AddAssetHtmlPlugin({
	filepath: path.resolve(__dirname, './dll/vendor.dll.js')
});
```

通过这包，`webpack`会将`dll`打包出来的`js`文件通过`script`标签引入到`index.html`文件中

这个时候你在`npm run build`，访问下页面就正常了

#### 6. 控制包文件大小

tree-shaking 等

#### 7. thread-loader parallel-webpack happypack 等多进程打包

#### 8. 合理使用 sourceMap

#### 9. 结合 stats 分析打包结果

借助线上或者本地打包分析工具

#### 10. 开发环境内存编译

开发环境的时候不会生成 dist 文件夹，会直接从内存中读取，因为内存读取比硬盘读取快

#### 11. 开发环境无用插件剔除

F
