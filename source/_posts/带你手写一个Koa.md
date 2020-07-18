---
title: 带你手写一个Koa
date: 2020-07-06 21:11:47
tags: ['koa', '源码实现']
thumbnail: https://s1.ax1x.com/2020/07/19/U2fpV0.jpg
categories: 前端
---

Koa 是一个新的 web 框架，由 Express 幕后的原班人马打造， 致力于成为 web 应用和 API 开发领域中的一个更小、更富有表现力、更健壮的基石。 通过利用 async 函数，Koa 帮你丢弃回调函数，并有力地增强错误处理。 Koa 并没有捆绑任何中间件， 而是提供了一套优雅的方法，帮助您快速而愉快地编写服务端应用程序。

<!-- more -->

## 1. 带着问题写

1. 如何耗时统计
2. 什么是洋葱模型

## 2. 文章讲解顺序

**本文会将 koa 文档介绍的内容大致框架实现一遍**。将分以下清单，来介绍、分析、手写`Koa`

1. 基础用法
2. 属性（request、response、上下文 `context`）
3. 洋葱模型

会举四个例子来实现koa，参考我的例子仓库。

```js
.
├── example
│   ├── 1.index.js // 用来泡基础用法
│   ├── 2.index.js // 用来介绍属性与扩展属性、上下文context、洋葱
│   ├── 3.index.js // 洋葱模型、中间件
│   ├── 4.index.js // 中间件例子实现
│   ├── bodyparser.js
│   └── public
│       └── index.html
├── lib
│   ├── application.js
│   ├── context.js
│   ├── request.js
│   └── response.js
├── package-lock.json
└── package.json
```

## 3. 基础用法

```js
// koa的核心功能就是创建一个服务，没了

const Koa = require('koa');
const app = new Koa();

app.use(async (ctx) => {
  ctx.body = 'hello world';
});

app.listen(3000);
```

下面我们来实现这个最基本的用法

### 3.1 例子1

```js
// example/1.index.js
const Koa = require('../lib/application.js');
const app = new Koa();

// 例子1.index.js 还未实现到ctx.body 所以暂时先用req,res 代替
app.use(async (req, res) => {
  res.end('hello wrold')
});

app.listen(5000);
```

### 3.2 **代码实现环节**

```js
const http = require('http');
const EventEmitter = require('events');

class Application extends EventEmitter {
  constructor() {
    super();
    this.middleware = function () {};
  }

  use(callback) {
    // 使用use的时候存储下函数
    this.middleware = callback;
  }

  // 服务响应的方法，用来触发中间件
  handleRequest(req, res) {
    this.middleware(req, res);
  }

  listen(arg) {
    http.createServer(this.handleRequest.bind(this)).listen(arg);
  }
}

module.exports = Application;
```

上述代码我就不过多解释了，一看就能懂。可以满足上述例子的正常运行。

## 4. 介绍属性

本文只举几个常见的，其他可参考
[官方文档-context](https://koa.bootcss.com/#context)
[官方文档-request](https://koa.bootcss.com/#request)
[官方文档-response](https://koa.bootcss.com/#response)

**koa相比于express使用非常的简单。**

比如:
获取`path`: `ctx.request.path` / `ctx.path`
设置path: `ctx.request.path =` / `ctx.path =`

`path`在原生的`req`里是获取不到的，`koa` 自己封装了一个`request` 及 `response` 等，功能大于原生`req`，且在`context`访问属性时，会自动代理`request`与`response`上。另外 `context` `request` 上都会挂载原生 `req`
所以获取`path`可以通过 `ctx.request.paht/ctx.path` 获取

```js
const Koa = require('koa')
const app = new Koa()

app.use(async(ctx, next) => {
  console.log(ctx.req.path, 1)
  console.log(ctx.request.req.path, 2)
  console.log(ctx.request.path, 3)
  console.log(ctx.path, 4)
  ctx.body = 'hello world'
})

app.listen(5000)
```

我们先用使用原生的koa包，跑下看下控制台输出来验证下我上面的说法

```js
undefined 1
undefined 2
/ 3
/ 4
```

可以看到，原生req上是没有path属性的，而koa自己封装的request上有。且ctx跟request上读取的是一样的。

### 4.1 例子2

```js
// 例子2.index.js
const Koa = require('../lib/application.js')
const app = new Koa()

app.use(async(ctx) => {
  console.log(ctx.req.path, 1)
  console.log(ctx.request.req.path, 2)
  console.log(ctx.request.path, 3)
  console.log(ctx.path, 4)
  // ctx.body = 'hello world'
  // ctx.body下面再实现，这里暂时先用ctx.res.end
  ctx.res.end('hello world')
})

app.listen(5000)
```

#### 4.2 代码实现

```js
const http = require('http');
const EventEmitter = require('events');
+ const context = require('./context.js')
+ const request = require('./request.js')
+ const response = require('./response.js')

class Application extends EventEmitter {
  constructor() {
    super();
    this.middleware = function () {};
    + this.context = Object.create(context)
    + this.request = Object.create(request)
    + this.response = Object.create(response)
  }

  use(callback) {
    // 中间件赋值
    this.middleware = callback;
  }

  createContext(req, res) {
    const context = Object.create(this.context)
    const request = Object.create(this.request)
    const response = Object.create(this.response)

    context.request = request
    context.response = response

    context.request.req = context.req = req
    context.response.res = context.res = res

    return context
  }

  handleRequest(req, res) {
    + const ctx = this.createContext(req, res)
    this.middleware(ctx);
  }

  listen(arg) {
    http.createServer(this.handleRequest.bind(this)).listen(arg);
  }
}

module.exports = Application;

```

大家重点看 `createContext` 通过调用这个方法来构建一个context，返回给中间件。

> **createContext 将req,res分别挂载在context,request,response上，并把request,response挂载在context上返回给客户端**
> 大家可能注意到为什么用了两次`Object.create()`，第一个是房子多个 `koa` 实例公用 `request、response、context`，第二个是多个请求公用这三个参数

在lib文件夹下创建 `request.js、context.js、response.js`

#### 4.2.1 context

```js
const context = {}

function defineGetter(target, key) {
  context.__defineGetter__(key, function() {
    return this[target][key]
  })
}

defineGetter('request', 'path')
module.exports = context;
```

这个文件主要作用就是通过 `__defineGetter__` 代理，访问 `context` 上的属性时候，代理到对应的`target`上(`request/response`)

上述例子上讲`context.path`代理到`context.request.path`上

#### 4.2.2 request

```js
// lib/request.js
const url = require('url')

module.exports = {
  get path() {
    return url.parse(this.req.url).pathname
  }
}
```

request这里没做什么特殊处理，主要提供对应的属性

#### 4.2.3 response

```js
// lib/response.js
module.exports = {}
```

运行下 `2.index.js` 输出日志

```js
undefined 1
undefined 2
/url 3
/url 4
```

可以看到与原生koa跑出的日志是一致的。

#### 4.2.4 ctx.body

在实现完request上的属性之后，接下来实现下response上的属性

修改如下代码

```js
// 2.index.js
const Koa = require('../lib/application.js')
const app = new Koa()

app.use(async(ctx) => {
  console.log(ctx.req.path, 1)
  console.log(ctx.request.req.path, 2)
  console.log(ctx.request.path, 3)
  console.log(ctx.path, 4)
  - ctx.res.end('hello world')
  + ctx.body = 'hello'
})

app.listen(5000)
```

```js
// lib/application.js

class Application extends EventEmitter {
  // ... 省略

  handleRequest(req, res) {
    const ctx = this.createContext(req, res)
    - this.middleware(ctx)
    + Promise.resolve(this.middleware(ctx)).then(() => {
    +   res.end(ctx.body)
    + });
  }
}

module.exports = Application;

```

这里可以改动主要是执行中间件后，执行下res.end,并返回在中间件设置的ctx.body内容

```js
// lib/context.js
const context = {}

function defineGetter(target, key) {
  context.__defineGetter__(key, function() {
    return this[target][key]
  })
}
+ function defineSetter(target, key) {
+   context.__defineSetter__(key, function(val) {
+     this[target][key] = val
+   })
+ }

defineGetter('request', 'path')
+ defineGetter('response', 'body')
+ defineSetter('response', 'body')

module.exports = context;
```

将对ctx.body的修改代理到对ctx.response.body的修改

```js
// lib/response.js
module.exports = {
  _body: '',
  get body() {
    return this._body
  },
  set body(val) {
    this._body = val
  }
};
```

修改body

运行下代码，**success** 😁

到这里属性章节就介绍并实现完了。

## 5. 洋葱模型

上面的代码实现中举的例子都是只有一个中间件，其实koa是支持多中间件共同使用的。

我们先借助下原生koa来看下洋葱模型大致是什么个样子

```js
// 洋葱模型
app.use((ctx, next) => {
  console.log(1)
  next();
  console.log(2)
  ctx.body = 'hello1';
});

app.use((ctx, next) => {
  console.log(3);
  next();
  console.log(4);
  ctx.body = 'hello2';
});

app.use((ctx, next) => {
  console.log(5);
  next();
  console.log(6);
  ctx.body = 'hello3';
});
```

```js
1
3
5
6
4
2
```

可以根据输出理解下，我画个图给大家理解下

![洋葱模型](https://s1.ax1x.com/2020/07/19/U2R7p4.jpg)

用代码简便下就更好理解了，就是用下一个中间件函数体替换掉上一个函数体的next()

```js
// 洋葱模型
app.use((ctx, next) => {
  console.log(1)
  console.log(3);
  console.log(5);
  console.log(6);
  ctx.body = 'hello3';
  console.log(4);
  ctx.body = 'hello2';
  console.log(2)
  ctx.body = 'hello1';
});
```

所以上面问题实现的第一个问题计算接口耗时统计就可以在第一个中间件来实现

```js
app.use(async (ctx, next) => {
  console.time()
  await next()
  console.timeend()
})
```

### 5.1 代码实现环节

```js
const http = require('http');
const EventEmitter = require('events');
const context = require('./context.js')
const request = require('./request.js')
const response = require('./response.js')

class Application extends EventEmitter {
  constructor() {
    super();
    - this.middleware = function() {};
    + this.middlewares = [];
    this.context = Object.create(context)
    this.request = Object.create(request)
    this.response = Object.create(response)
  }

  use(callback) {
    - this.middleware = callback;
    + this.middlewares.push(callback);
  }

  createContext(req, res) {
    const context = Object.create(this.context)
    const request = Object.create(this.request)
    const response = Object.create(this.response)

    context.request = request
    context.response = response

    context.request.req = context.req = req
    context.response.res = context.res = res

    return context
  }

  + compose(ctx) {
  +   const dispatch = (i) => {
  +     if (i === this.middlewares.length) return Promise.resolve()
  +     return Promise.resolve(this.middlewares[i](ctx, () => dispatch(i + 1)))
  +   }
  +   return dispatch(0)
  + }

  handleRequest(req, res) {
    const ctx = this.createContext(req, res)
    - Promise.resolve(this.middleware(ctx)).then(() => {
    + this.compose(ctx).then(() => {
      res.end(ctx.body)
    });
  }

}

module.exports = Application;
```

可以看出来，我上面的middleware不再是一个函数体了，而是一个数组middlewares，所以use的时候是将callback推入到数组中

调用compose执行middlewares中所有的中间件，传入ctx并传入next即下一个中间件函数

跑下程序，看了下输出结果，**success** 😆

## 总结

我总结了下koa的源码无非做了就以下几点

1. 简化了属性的获取与设置
2. 通过利用 async 函数，Koa 帮你丢弃回调函数
3. 由各种中间件构成一个应用程序
