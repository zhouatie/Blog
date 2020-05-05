---
title: 源码实现系列之vue-router
date: 2020-04-12 02:49:23
tags: ['vue-router', '源码实现']
thumbnail: https://s1.ax1x.com/2020/05/05/YijUdP.jpg
categories: 前端
---

这个月会实现一下`Vue`, `Vuex`, `vue-router`。我会以**倒推**的模式边开发边写文章。话不多说开始跟着我一起撸。[仓库地址](https://github.com/zhouatie/front-end/tree/master/analysis-vue-router)

> 本文只是实现了一个基础版本的`vue-router`.本文所写的代码，不会每个地方都做异常判断。实现一个能够体现`vue-router`核心逻辑即可。

<!--more-->

我大致捋了下`vue-router`的流程图如下：
![vue-router思维导图](https://s1.ax1x.com/2020/05/03/JxW5TS.png)

在写源码之前，我先展示下`routes`的数据结构，在根据这个数据结构来进行`vue-router`的开发

```js
const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home,
  },
  {
    path: '/about',
    name: 'About',
    component: () =>
      import(/* webpackChunkName: "about" */ '../views/About.vue'),
    children: [
      {
        path: 'a',
        name: 'about.a',
        component: {
          render: (h) => <div>this is a</div>,
        },
      },
      {
        path: 'b',
        name: 'about.b',
        component: {
          render: (h) => <div>this is b</div>,
        },
      },
    ],
  },
];
```

## 安装 install

回忆下我们平时代码里用到`vue-router`必定会有的两行代码

```js
import VueRouter from 'vue-router';

Vue.use(VueRouter);
```

`Vue`在`use` `VueRouter`的时候会调用`VueRouter`的`install`方法，并传入`Vue`构造函数。

所以我们接下来第一步就是创建一个`VueRouter`构造函数，并实现一个`install`方法。

```js
// vue-router/index.js
const install = (Vue) => {
  Vue.mixin({
    beforeCreate() {
      if (this.$options.router) {
        this._routerRoot = this;
        this._router = this.$options.router;
      } else {
        this._routerRoot = this.$parent && this.$parent._routerRoot;
      }
    },
  });

  // FIXME:
  Object.defineProperty(Vue.prototype, '$route', {
    get() {
      return 'this is $route';
    },
  });

  Object.defineProperty(Vue.prototype, '$router', {
    get() {
      return this._routerRoot._router;
    },
  });

  // FIXME:
  Vue.component('router-view', {
    render() {
      return <div>this is router-view</div>;
    },
  });
  // FIXME:
  Vue.component('router-link', {
    render() {
      return <div>this is router-link</div>;
    },
  });
};

export default install;
```

`vue-router`也会与`vuex`一样，通过`Vue.mixin`方法添加`beforeCreate`生命周期，通过在这个生命周期函数，向每个组件实例挂载`_routerRoot`根实例对象，根实例会多挂载一个`_router`对象，这样子组件就可以通过`_routerRoot._router`来获取路由实例对象了。同时会往`Vue.prototype`上挂载`$router` `$route`，方便每个子组件获取。

并挂载全局组件`router-view`、`router-link`组件;

## 实例

写完`install`方法后，我们接下来开始动手`VueRouter`。初始化时，该在`constructor`中写些什么呢？初始化当然是为了后续的一些方法做准备了。他将包含以下几点

- 创建`matcher`对象
  - `addRoute`: 用于动态添加路由
  - `match`: 重中之重，用于根据`path`匹配出对应的`route`对象，用于`router-view`的渲染
- 根据对应的`mode`创建对应的`history`对象（我这里开发的`router`默认使用`HashHistory`

```js
class VueRouter {
  constructor(options) {
    this.matcher = createMatcher(options.routes);

    this.history = new HashHistory();
  }

  init() {}
}

```

### createMatcher

那么`createMatcher`到底该怎么实现？还是反推的模式，它肯定返回上面列的两个方法`addRoute` `match`方法。那么就先实现下大致的样子吧。

```js
// create-matcher.js

function createMatcher(routes) {
  function addRoute() {}

  function match() {}

  return {
    addRoute,
    match,
  };
}

export default createMatcher;
```

#### match

接下来就是实现`match`方法了。前面说过`match`是用来匹配`path`所对应的`route`对象。所以他肯定有个`path`参数，然后通过一个路由映射表来筛选出对应的`route`对象。那就开始着手编码吧！

```js
// create-matcher.js
function match(location) {
    return pathMap[location]
  }
```

这个时候你肯定会有个疑惑，这个`pathMap`哪里来的呢？上面我已经说过`pathMap`是一个路由映射表。那就开始着手编码吧！

```js
// create-matcher.js
function createMatcher(routes) {
  const { pathList, pathMap } = createRouteMap(routes);
  function addRoute() {}

  function match(locaiton) {
    return pathMap[locaiton];
  }

  return {
    addRoute,
    match,
  };
}
```

`match`方法中并不会那么简单的返回`route`对象就ok了。肯定是经过一个函数保证过返回`matched` `path`等字段。这里先忽略，等讲到`history`的时候再讲解。

##### createRouteMap

你可以看到代码中调用了`createRouteMap`，那么这个方法该怎么实现呢？首先我们已知这个方法是用来创建`pathMap`的，那么先搭一个大致的函数框架

```js
// create-route-map.js
function createRouteMap(routes) {
  const pathList = [];
  const pathMap = Object.create(null);

  return {
    pathList,
    pathMap,
  };
}

export default createRouteMap;
```

大致会是这个样子。执行返回路由映射表。那么该怎么创建`pathList` `pathMap`对象呢？当然用传入的`routes`来构造了。那就开始着手编码吧！

```js
// create-route-map.js
function createRouteMap(routes) {
  const pathList = [];
  const pathMap = Object.create(null);

  routes.forEach(route => {
    addRouteRecord(route, pathList, pathMap)
  })
  return {
    pathList,
    pathMap,
  };
}

function addRouteRecord (route, pathList, pathMap) {
  const path = route.path
  const record = {
    path,
    component: route.component
  }
  if (!pathList.includes(path)) {
    pathList.push(path)
    pathMap[path] = record
  }
}
export default createRouteMap;
```

上面我通过遍历`routes`，调用`addRouteRecord`方法，并传入`route`。`addRouteRecord`根据传入的`route`构建`pathList` `pathMap`，我想大家看代码应该就理解这段代码是什么意思了。当然上面只是非常非常基础版本的。还有一些场景没考虑到，比如传入的`route`对象还有`children`,就需要继续遍历他的`children`并调用`addRouteRecord`方法。那就开始着手编码吧！

```js
// create-route-map.js
function addRouteRecord(route, pathList, pathMap, parent) {
  const path = parent ? `${parent.path}/${route.path}` : route.path;
  const record = {
    path,
    component: route.component,
    parent,
  };
  if (!pathList.includes(path)) {
    pathList.push(path);
    pathMap[path] = record;
  }

  if (route.children) {
    route.children.forEach((o) => {
      addRouteRecord(o, pathList, pathMap, record);
    });
  }
}
```

你会发现`addRouteRecord`多了个`parent`参数，是因为有子路由。给`record`增加`parent`属性是因为方便后面父子组件递归渲染。

`match` 实现完了，就该实现下`addRoute`。那么`addRoute`又该怎么实现呢？其实非常的简单，就是给`pathList` `pathMap`增加`routes`对象罢了。那么我们就开始动手吧！

```js
// create-matcher.js
function addRoute(routes) {
  createRouteMap(routes, pathList, pathMap);
}
```

你会发现这里的`createRouteMap`增加了2个参数，所以`createRouteMap`方法就需要去兼容了。那么我们就开始动手吧！

```js
// create-route-map.js

function createRouteMap(routes, oldPathList, oldPathMap) {
  - const pathList = [];
  + const pathList = oldPathList || [];
  - const pathMap =Object.create(null);
  + const pathMap = oldPathMap || Object.create(null);
  // ...
}
```

只需要对原先创建`pathList` `pathMap`对象进行兼容即可。

### History

`constructor`中的`matcher`对象创建讲完之后，接下来我们来讲讲`constructor`中的`HashHistory`。

`HashHistory`该怎么实现？`new HashHistory`后做了什么呢？

因为`history`模式有多种，我们需要实现一个`base`版的`history`，然后`HashHistory`基于这个基类进行扩展。

```js
// history/base.js
export default class History {
  constructor() {}
}
```

```js
import History from './base';

export default class HashHistory extends History {
  constructor(router) {
    super(router);
    this.router = router;
  }
}
```

### init

#### init 哪里调用

这个`init`会在哪里调用呢？当然是安装的时候调用。所以我们在`install`的时候加上`init`的调用。

`init`为什么要传入根实例呢？因为需要监听当前`url`对应的路由变化。当他变化之后，会主动将根实例的`_route`赋值成当前的根路由。那根实例的`_route`哪来的呢？可能有人忘记了。可以翻到上面`install`的章节，那里讲过了。

```js
// install.js
const install = (Vue) => {
  Vue.mixin({
    beforeCreate() {
      if (this.$options.router) {
        this._routerRoot = this;
        this._router = this.$options.router;
        + this._router.init(this)
      } else {
        this._routerRoot = this.$parent && this.$parent._routerRoot;
      }
    },
  });
}
```

加号位置就是我添加的代码。调用下`init`初始化下`VueRouter`

那么`init`方法中做了哪些事情呢？

- 挂载当前`url`对应的路由组件
- 监听路由的变化（`hashchange`等事件）

话不多说，进入编码模式

```js
// vue-router/index.js
import HashHistory from './history/hash';
import install from './install';
import createMatcher from './create-matcher';

class VueRouter {
  constructor(options) {
    // ...
  }
  init(app) {
    const history = this.history;
    const setupListener = () => {
      history.setupListener();
    };
    history.transitionTo(history.getCurrentLocation(), setupListener);
    history.listen((route) => {
      app._route = route;
    });
  }
}

```

#### init中做了哪些工作

##### transitionTo

`init`中调用了`history`中的`transitionTo`,那么这个方法是干嘛的呢？用于路由的跳转，根据传入的`path`从`pathMap`中筛选出对应的`route`，这个方法会触发下面讲到的`listen`方法。这个方法触发后，会修改根实例的`_route`。修改之后，`router-view`就会响应式的改变，以达到刷新路由渲染页面的目的。因为调用`transitionTo`方法会有多种途径。一种是主动调用`push`方法等，需要主动修改浏览器地址栏`hash`值，一种是页面初始化调用，这个时候又需要监听`hashchange`等事件，所以`transitionTo`增加第二个参数用于回调。这样每个调用`transitionTo`后，可执行自己的逻辑。

话不多说上代码

```js
// history/base.js
export default class History {
  constructor(router) {
    + this.router = router
  }

  + transitionTo(location, callback) {
  +  const r = this.router.match(location)
  +  console.log(r)
  +  callback && callback()
  + }
}
```

上面简短的代码是不是就实现了上面描述中的几个功能了。
还有两点功能没实现。

- 怎么响应式刷新
- 怎么实现匹配路由（`match`)
  
我们先来讲**怎么响应式刷新**。那么怎样才能实现`router-view`响应式的刷新呢？我们根据倒推的模式，`router-view`是根据根实例的`_route`做刷新的。所以需要增加个`current`对象用来表示当前路由。上代码🐶

```js
// history/base.js
export default class History {
  constructor(router) {
    this.router = router
    + this.current = createRoute(null, {
    +  path: '/'
    + })
  }

  transitionTo(location, callback) {
    const r = this.router.match(location)
    + this.current = r
    callback && callback()
    console.log(r)
  }
}
```

可是光设置`current`还不能实现`router-view`响应式刷新。因为`route-view`是根据`$route`做响应式的。还记得在`install`的时候设置过`$route`吗？我们将它修改下。

```js
// install.js

const install = (Vue) => {
  Vue.mixin({
    beforeCreate() {
      if (this.$options.router) {
        // ...
        this._router.init(this)
        + Vue.util.defineReactive(this, '_route', this._router.history.current);
      } else {
        this._routerRoot = this.$parent && this.$parent._routerRoot;
      }
    },
  });

  Object.defineProperty(Vue.prototype, '$route', {
    get() {
      - return 'this is $route';
      + return this._routerRoot._route;
    },
  });
}
```

这样 我们一旦修改`current`，页面的`$route`就会响应式更新了。刷新下页面试试看吧🐶。

竟然报错了。提示`createRoute is not defined`。

抄下上面贴过的代码

```js
// history/base.js
export default class History {
  constructor(router) {
    this.router = router
    + this.current = createRoute(null, {
    +  path: '/'
    + })
  }

  transitionTo(location, callback) {
    const r = this.router.match(location)
    + this.current = r
    callback && callback()
    console.log(r)
  }
}
```

我们来讲讲`createRoute`的作用。他的作用是将匹配到的`route`进行处理，返回个包含`path`、`matched`字段。`matched`字段包含了，从匹配到的一级路由一直到最后一级路由。`router-view`也是根据这一数组进行父组件到子组件的渲染的。`match`方法中也用到的这个方法。下面再讲`match`方法。接下来我们就来实现`createRoute`方法。

```js
// history/base.js
export function createRoute(record, location) {
  const matched = [];
  while (record) {
    matched.unshift(record);
    record = record.parent;
  }

  return {
    matched,
    ...location,
  };
}
```

第一个参数`record`其实就是上文`createRouteMap`中`addRouteRecord`使用到的`record`。其中包含了`parent`字段就是在这个时候用到的。所以`/about/a`就可以生成`matched: [{path: '/about', component: componentAbout}, {path: '/about/a', component: componentAboutA}]`了。再次提醒下，`matched`用于`router-view`的层层渲染。话不多说，上代码🐶

```js
// vue-router/index.js
class VueRouter {
  constructor(options) {
    this.matcher = createMatcher(options.routes);
    // ...
  }

  match(location) {
    return this.matcher.match(location);
  }

  // ...
}
```

提醒下，这里的`matcher`上面讲过了。用来返回`match`跟`addRoute`方法。

然后再完善下上面写过的`createMatcher`

```js
// create-matcher.js
+ import { createRoute } from './history/base';

function createMatcher(routes) {
  const { pathList, pathMap } = createRouteMap(routes);
  // ...

  function match(locaiton) {
    - return pathMap[locaiton];
    + return createRoute(pathMap[locaiton], {
    +   path: locaiton,
    + });
  }
  
  // ...
}
```

看着好像没什么问题了，再刷新下页面。

提示`history.setupListener is not a function`

我们再把思绪拉回到`init`方法中调用的`history.setupListener`

`setupListener`其实非常的简单，就是添加监听事件

```js
export default class HashHistory extends History {
  constructor(router) {
    super(router);
    this.router = router;
  }

  getCurrentLocation() {
    return window.location.hash.slice(1);
  }

  + setupListener() {
  +   window.addEventListener('hashchange', () => {
  +     this.transitionTo(this.getCurrentLocation());
  +   });
  + }
}
```

监听到`hashchange`后，主动调用下`transitionTo`去切换路由。

然后我们再刷新下页面，又报错 excuse me？？？

提示: `TypeError: history.listen is not a function`

##### listen

```js
class VueRouter {
  constructor(options) {
    // ...
    this.history = new HashHistory(this);
  }

  init(app) {
    const history = this.history;
    // ...
    history.transitionTo(history.getCurrentLocation(), setupListener);
    > history.listen((route) => {
    >   app._route = route;
    > });
  }
}
```

`init`中`history.listen`其实也是非常的简单，就是添加订阅。当调用`transitionTo`后，触发下订阅的事件。并传入`location`对应的`route`。话不多说，上代码🐶

```js
// history/base.js
export default class History {
  transitionTo(location, callback) {
    const r = this.router.match(location);
    // ...
    this.cb && this.cb(r)
  }

  listen(cb) {
    this.cb
  }
}
```

这个时候再刷新下页面。打印下`.vue`文件里的`this.$route`看了下好像没问题😆。

上文说过`router-view`是根据`$route`层层渲染的，既然`$route`都冇问题了，那就开始编写`router-view`组件吧。

### router-view

在我看`router-view`源码之前，我根本不知道`router-view`怎么实现。原来它是根据`$route`的`matched`匹配到的组件进行层层渲染的。
举个🌰，就举上面打印出来的`matched`来讲好了，`app.vue`中的`router-view`会渲染`matched`的第一项中的`component`对应的`about.vue`组件，`about.vue`中的`router-view`会渲染`matched`中第二个`component`对应的`about/a`组件。可能上代码更简单易懂🐶,那我就开始开发了（本文都是边开发边写文章的）。

`router-view`组件是用的函数式组件，因为[函数式组件](https://cn.vuejs.org/v2/guide/render-function.html#%E5%87%BD%E6%95%B0%E5%BC%8F%E7%BB%84%E4%BB%B6)无状态 (没有响应式数据)，也没有实例 (没有 this 上下文)

```js
// components/router-view.js
export default {
  functional: true,

  render(h, {parent, data}) {
    console.log(parent, 'parent')
  }
}
```

修改`install`中`rotuer-view`组件定义。

```js
// install.js
import RouterView from './components/router-view';

const install = (Vue) => {
  - Vue.component('router-view', {
  -   render() {
  -     return <div>this is router-view</div>;
  -   },
  - });
  + Vue.component('router-view', RouterView);

};

export default install;
```

上面代码写完之后，好像一切正常，打印出来也是正常的。接下来就根据我上面说的，他是根据`$route.matched`来渲染的去实现它吧。

```js
export default {
  functional: true,

  render(h, { parent, data }) {
    console.log(parent, 'parent');
    const route = parent.$route;
    let depth = 0;
    // 判断是否渲染过，如果没有渲染过，就渲染对应matched里的组件，并将该组件data.routerView = 1。以达到不会重复渲染。
    while (parent) {
      if (parent.$vnode && parent.$vnode.data.routerView) {
        depth++;
      }
      parent = parent.$parent;
    }

    data.routerView = 1;

    const record = route.matched[depth];

    if (!record) {
      return h();
    }

    return h(record.component, data);
  },
};
```

`while`的作用是判断是否渲染过，如果没有渲染过，就渲染对应matched里的组件，并将该组件data.routerView = 1。以达到不会重复渲染。刷新下页面看看吧。好像子页面都出来了。

### router-link

`router-link`非常的简单。我这里就实现下比较常见的一些操作。比如参数`tag`,`to`等

```js
export default {
  props: {
    tag: {
      type: String,
      default: 'a',
    },
  },

  render() {
    const tag = this.tag;
    return <tag>{this.$slots.default}</tag>;
  },
};
```

我这里先写个比较简单的架子。非常的简单将文字展示在`tag`中

然后再修改下`install`中的引入

```js
// install.js
import RouterLink from './components/router-link';

const install = (Vue) => {
  - Vue.component('router-link', {
  -   render() {
  -     return <div>this is router-link</div>;
  -   },
  - });
  + Vue.component('router-link', RouterLink);

};

export default install;
```

刷新下页面试试。页面一切展示正常🐶。这就大功告成了吗？当然没有啦，还有`router-link`的点击事件没写，那就开始动手吧。

```js
// router-link.js

export default {
  props: {
    tag: {
      type: String,
      default: 'a',
    },

    + to: {
    +   type: String,
    +   required: true,
    + },
  },

  + methods: {
  +   handler() {
  +     this.$router.push(this.to);
  +   },
  + },

  render() {
    const tag = this.tag;
    - return <tag>{this.$slots.default}</tag>;
    + return <tag onClick={this.handler}>{this.$slots.default}</tag>;
  },
};
```

给`tag`增加点击事件。我上面的例子将`to`定义成`String`类型了。其实这个`to`可以是对象类型。我的例子只要能体现出`router-link`的功能就行了。

接下来刷新下页面试试看。

报了个错: `TypeError: this.$router.push is not a function`

原来我`push`忘记写了。那就在`vue-router`实例中加一个吧。

```js
// vue-router/index.js

class VueRouter {
  constructor(options) {
    // ...
    this.history = new HashHistory(this);
  }

  + push(location) {
  +   this.history.transitionTo(location, () => {
  +     window.location.hash = location;
  +   });
  + }
}
```

调用下`history.transitionTo`进行路由的匹配替换，触发`router-view`渲染后，还有在回调中主动修改下`hash`地址。

刷新下页面。一切正常。再点击`router-link`标签。emmmm 一切正常。（重复点击一个link跳转没有处理，我就不处理。我觉得没有必要，我的目的就是能表达我对router的理解就够了）

## 最后

非常感谢你能读完这篇文章。我已经非常努力写这篇文章了（vue-router写了三遍才开始写文章的）。但是读起来好像还是不是很顺。
