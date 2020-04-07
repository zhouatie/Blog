---
title: 源码实现系列之Vuex
date: 2020-04-02 23:45:02
tags: ['vuex', '源码实现']
thumbnail: https://s1.ax1x.com/2020/04/03/Gt2PbQ.jpg
categories: 前端
---

这个月会实现一下`Vue`, `Vuex`, `vue-router`。我会以**倒推**的模式边开发边写文章。话不多说开始跟着我一起撸。

> 本文只是实现了一个非常基础版本的`vuex`，日后会进行升级。且本文所写的代码，不会每个地方都做异常判断，只满足我表达`vuex`的实现原理即可。

<!--more-->

回忆下我们平时代码里用到`vuex`必定会有的两行代码

```javascript
Vue.use(Vuex);

store = new Vuex.Store({});
```

所以`vuex`需要导出两个方法，一个提供`use`的`install`方法，一个就是`Store`

```js
// vuex.js

export default {
  install,
  Store,
};
```

诸如`vuex`、`vue-router`等都是使用`Vue.mixin`,往每个组件上挂载`beforeCreate`生命周期来初始化。

上代码

```js
const install = (_Vue) => {
  Vue = _Vue;
  Vue.mixin({
    beforeCreate() {
      if (this.$options.store) {
        this.$store = this.$options.store;
      } else {
        this.$store = this.$parent && this.$parent.$store;
      }
    },
  });
};
```

`Vue`的`use`方法我就不解释了，感兴趣的可以去看下`vue`源码，他是调用`use`参数的`install`方法，并传入`Vue`构造函数。
那么上述为什么会判断`this.$options.store`和`读取$parent.$store`呢？

这个时候请回忆下我们`new Vue`的都会传入`store`,所以当`this.$options.store`获取不到，代表他不是根节点（当然子组件也可以加`store`,这里只是常规分析）。`this.$store = this.$parent && this.$parent.$store;`，那么他很有可能就是子组件，就会向上层获取，上层肯定会有。就这样一层层往下传递。

`install`讲完了，接下来就讲我们的重头戏，`Store`;

### Store

下面我将按照以下几个属性进行讲解

- `state`
- `mapState`
- `getters`
- `mapGetters`
- `mutations`
- `mapMutations`
- `action`
- `mapActions`

#### state

`Store`中的响应式都是借助`Vue`来进行数据响应式。

我先在`Vuex`注册的时候，加个`state`属性。

```js
// store.js
import Vuex from './vuex/index.js';
import Vue from 'vue';

Vue.use(Vuex);

const store = new Vuex.Store({
  state: {
    count: 1,
  },
});

export default store;
```

这个时候，源码上就要支持`state`的获取

所以获取 state 的时候，直接读取`vm.state`,**且`state`只能获取不能修改**

```js
get state() {
  return this.vm.state;
}
```

得到代码如下

```js
let Vue;
const install = (_Vue) => {
  // ...
};

class Store {
  constructor(options) {
    console.log(options, 'options');
    this.vm = new Vue({
      data: {
        state: options.state,
      },
    });
  }

  get state() {
    return this.vm.state;
  }
}

export default {
  install,
  Store,
};
```

编写`vue`组件代码

```html
<template>
  <div id="app">
    <div>{{ $store.state.a }}{{ $store.state.b }}</div>
  </div>
</template>
<script>
export default {
  name: 'App',
  created() {
    console.log(this.$store.state, 'store');
  },
};
</script>
```

打开页面看下效果，页面成功读取到`state`中的值了;

#### mapState

我们一般使用`mapState`会有如下几种用法

```js
export default {
  computed: {
    ...mapState(['count']),
    ...mapState({
      count2: 'count',
      count3: (state) => `${state.count} xixi`,
    }),
  },
};
```

我们可以看到，`mapState`支持数组、对象、对象中的`value`还可以以函数的形式存在。
所以我们实现的时候

- 如果是数组，就直接从`state`中取每个值。
- 如果是对象，就将对象的每个键的值赋值为一个函数，并传入 state 这个对象。

实现方式如下

```js
function normalizeMap(map) {
  return Array.isArray(map)
    ? map.map((key) => ({ key, val: key }))
    : Object.keys(map).map((key) => ({ key, val: map[key] }));
}

export const mapState = (options) => {
  if (typeof options !== 'object') {
    console.error(
      '[vuex] mapActions: mapper parameter must be either an Array or an Object'
    );
    return;
  }

  const res = {};
  normalizeMap(options).forEach((o) => {
    if (typeof o.val === 'function') {
      res[o.key] = function () {
        return o.val.call(this.$store, this.$store.state);
      };
    } else {
      res[o.key] = function () {
        return this.$store.state[o.val];
      };
    }
  });
  return res;
};
```

#### getters

`getters`其实非常的简单，就是调用他的方法，传入`state`参数

```html
<div>getsname: {{ $store.getters.getsname }}</div>
```

```js
class Store {
  constructor(options) {
    console.log(options, 'options');
+   this.getters = options.getters;
    this.vm = new Vue({
      data: {
        state: options.state
      }
    });

+   Object.keys(this.getters).forEach(key => {
+     this.getters[key] = this.getters[key].call(this, this.vm.state);
+   });
+  }

  get state() {
    return this.vm.state;
  }
}
```

#### mapGetters

`mapGetters`其实与`mapState`非常的相似，只是传参发生了变化。变成了`(state, getters)`,我就不过多解释了。

```html
<div>getsname: {{ getsname }}</div>
<div>getsname2: {{ getsname2 }}</div>
<div>getsname3: {{ getsname3 }}</div>
```

```js
export default {
  computed: {
    // ...
    ...mapGetters(['getsname']),
    ...mapGetters({
      getsname2: 'getsname',
      getsname3: (state, getters) =>
        `count: ${state.count}, getsname: ${getters.getsname}`,
    }),
  },
};
```

```js
export const mapGetters = (options) => {
  if (typeof options !== 'object') {
    console.error(
      '[vuex] mapActions: mapper parameter must be either an Array or an Object'
    );
    return;
  }

  const res = {};
  normalizeMap(options).forEach((o) => {
    if (typeof o.val === 'function') {
      res[o.key] = function () {
        return o.val.call(this.$store, this.$store.state, this.$store.getters);
      };
    } else {
      res[o.key] = function () {
        return this.$store.getters[o.val];
      };
    }
  });
  return res;
};
```

#### mutations

`mutations`其实也是非常的简单，只需要写个`commit`方法，并触发下`mutations`里对应的方法，并传入`state`跟用户的参数两个变量就行了。

```html
<button @click="$store.commit('increment', 1)">
  mutation:increment => {{ count }}
</button>
```

```js
class Store {
  // ...
  commit(event, payload) {
    this.mutations[event].call(this, this.state, payload);
  }
}
```

#### mapMutations

`mapMutations`与上面的`mapState, mapGetters`差不多,就是遍历`mutations`返回一个对象。

```html
<button @click="increment(1)">mapMutations mutation:increment => {{ count }}</button>
<button @click="increment2(2)">increment2 mutation:increment => {{ count }}</button>
```

```js
export default {
  methods: {
    ...mapMutations(['increment']),
    ...mapMutations({
      increment2: 'increment'
    })
  }
}
```

```js
export const mapMutations = options => {
  if (typeof options !== 'object') {
    console.log('本实例暂不支持module方式');
    return {};
  }
  const res = {};
  normalizeMap(options).forEach(o => {
    res[o.key] = function(option) {
      return this.$store.commit(o.val, option);
    };
  });
  return res;
};
```

`normalizeMap`在上述`map`方法中使用到了，主要用途就是将数组或者对象处理成一个包含`key, value`对象。

#### actions

Action 类似于 mutation，不同在于：

- Action 提交的是 mutation，而不是直接变更状态。
- Action 可以包含任意异步操作。

乍一眼看上去感觉多此一举，我们直接分发 mutation 岂不更方便？实际上并非如此，还记得 mutation 必须同步执行这个限制么？Action 就不受约束！我们可以在 action 内部执行异步操作;

```js
asyncIncrement(context, payload) {
  setTimeout(() => {
    context.commit('increment', payload.count);
  }, 5000);
}
```

实现其实也是非常简单的，就是触发下注册的`action`而已。

先上代码

```js
dispatch(event, payload) {
  if (isObject(event)) {
    const { type, ...res } = event;
    event = type;
    payload = res;
  }
  this.actions[event].call(this, this, payload);
}
```

这里加了层判断第一个参数是否是对象，如果是对象，事件名就取对象中的`type`字段，`payload`就取剩下的。

三种基本操作(常规操作、参数放在一个对象中传、异步操作)例子如下

```html
<button @click="$store.dispatch('actionIncrement', { count: 3 })">test dispatch ===> actionIncrement</button>
<button @click="$store.dispatch({ type: 'actionIncrement', count: 4 })">test dispatch ===> actionIncrement</button>
<button @click="$store.dispatch({ type: 'asyncIncrement', count: 4 })">test dispatch ===> asyncIncrement</button>
```

```js
const store = new Vuex.Store({
  // ...
  actions: {
    actionIncrement(context, payload) {
      context.commit('increment', payload.count);
    },
    asyncIncrement(context, payload) {
      setTimeout(() => {
        context.commit('increment', payload.count);
      }, 5000);
    }
  }
});
```

#### mapActions

`mapActions`与`mapMutations`基本逻辑是一致的，一个是用`commit`触发`mutations`中的方法，一个是用`dispatch`触发`actions`中的方法。

```js
export const mapActions = options => {
  if (typeof options !== 'object') {
    console.log('本实例暂不支持module');
    return {};
  }
  const res = {};
  normalizeMap(options).forEach(o => {
    res[o.key] = function(option) {
      return this.$store.dispatch(o.val, option);
    };
  });
  return res;
};
```

`vue`页面例子如下

```html
<div>
  <p>测试 mapActions</p>
  <button @click="actionIncrement({count: 5})">常规操作actionIncrement</button>
  <button @click="asyncIncrement({count: 6})">异步操作asyncIncrement</button>
  <button @click="actionIncrement3({count: 7})">更换名字actionIncrement3</button>
</div>
```

```js
export default {
  methods: {
    ...mapActions(['actionIncrement', 'asyncIncrement']),
    ...mapActions({
      actionIncrement3: 'actionIncrement'
    })
  }
}
```

到此`vuex`基本功能已经全部满足了。日后有心思的话，会对文章及代码进行升级，并实现一套完整的`vuex`。

讲解代码如下

```js
let Vue;
const install = _Vue => {
  if (Vue) {
    return console.log('请勿重复安装');
  }
  Vue = _Vue;
  _Vue.mixin({
    beforeCreate() {
      if (this.$options.store) {
        this.$store = this.$options.store;
      } else {
        this.$store = this.$parent && this.$parent.$store;
      }
    }
  });
};

function normalizeMap(map) {
  return Array.isArray(map) ? map.map(key => ({ key, val: key })) : Object.keys(map).map(key => ({ key, val: map[key] }));
}

function isObject(obj) {
  return Object.prototype.toString.call(obj).slice(8, -1) === 'Object';
}
class Store {
  constructor(options) {
    this.getters = options.getters || {};
    this.mutations = options.mutations || {};
    this.actions = options.actions || {};

    this.vm = new Vue({
      data: {
        state: options.state
      }
    });

    Object.keys(this.getters).forEach(key => {
      this.getters[key] = this.getters[key].call(this, this.vm.state);
    });
  }

  get state() {
    return this.vm.state;
  }

  commit(event, payload) {
    this.mutations[event].call(this, this.state, payload);
  }

  dispatch(event, payload) {
    console.log(event, 'event')
    if (isObject(event)) {
      const { type, ...res } = event;
      event = type;
      payload = res;
    }
    this.actions[event].call(this, this, payload);
  }
}

export const mapState = options => {
  if (typeof options !== 'object') {
    console.error('[vuex] mapActions: mapper parameter must be either an Array or an Object');
    return;
  }

  const res = {};
  normalizeMap(options).forEach(o => {
    if (typeof o.val === 'function') {
      res[o.key] = function() {
        return o.val.call(this.$store, this.$store.state);
      };
    } else {
      res[o.key] = function() {
        return this.$store.state[o.val];
      };
    }
  });
  return res;
};

export const mapGetters = options => {
  if (typeof options !== 'object') {
    console.error('[vuex] mapActions: mapper parameter must be either an Array or an Object');
    return;
  }

  const res = {};
  normalizeMap(options).forEach(o => {
    if (typeof o.val === 'function') {
      res[o.key] = function() {
        return o.val.call(this.$store, this.$store.state, this.$store.getters);
      };
    } else {
      res[o.key] = function() {
        console.log(this.$store.state, o.value, '$sotre');
        return this.$store.getters[o.val];
      };
    }
  });
  return res;
};

export const mapMutations = options => {
  if (typeof options !== 'object') {
    console.log('本实例暂不支持module');
    return {};
  }
  const res = {};
  normalizeMap(options).forEach(o => {
    res[o.key] = function(option) {
      return this.$store.commit(o.val, option);
    };
  });
  return res;
};

export const mapActions = options => {
  if (typeof options !== 'object') {
    console.log('本实例暂不支持module');
    return {};
  }
  const res = {};
  normalizeMap(options).forEach(o => {
    res[o.key] = function(option) {
      return this.$store.dispatch(o.val, option);
    };
  });
  return res;
};

export default {
  install,
  Store
};
```
