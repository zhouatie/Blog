---
title: vue响应式原理
date: 2018-09-10 01:05:28
categories: 前端
tags: ['vue']
thumbnail: https://s1.ax1x.com/2020/03/30/GemgaV.png
---

## initState

<!-- more -->

new Vue() => \_init() => initState:

```javascript
function initState(vm: Component) {
	vm._watchers = [];
	const opts = vm.$options;
	if (opts.props) initProps(vm, opts.props);
	if (opts.methods) initMethods(vm, opts.methods);
	if (opts.data) {
		initData(vm);
	} else {
		observe((vm._data = {}), true /* asRootData */);
	}
	if (opts.computed) initComputed(vm, opts.computed);
	if (opts.watch && opts.watch !== nativeWatch) {
		initWatch(vm, opts.watch);
	}
}
```

判断该 vue 实例是否存在`props`、`methods`、`data`、`computed`、`watch`进行调用相应的初始化函数

### **initProps 与 initData**

主要工作是调用`defineProperty`给属性分别挂载 get(触发该钩子时，会将当前属性的 dep 实例推入当前的 Dep.target 也就是当前 watcher 的 deps 中即它订阅的依赖，Dep.target 下文会讲到。且该 dep 实例也会将当前 watcher 即观察者推入其 subs 数组中)、set 方法（通知该依赖 subs 中所有的观察者 watcher 去调用他们的 update 方法）。

### **initComputed**

它的作用是将 computed 对象中所有的属性遍历，并给该属性 new 一个 computed watcher（计算属性中定义了个 dep 依赖，给需要使用该计算属性的 watcher 订阅）。也会通过调用`defineProperty`给 computed 挂载 get（get 方法）、set 方法（set 方法会判断是否传入，如果没传入会设置成 noop 空函数）
`computed`属性的 get 方法是下面函数的返回值函数

```javascript
function createComputedGetter(key) {
	return function computedGetter() {
		const watcher = this._computedWatchers && this._computedWatchers[key];
		if (watcher) {
			watcher.depend();
			return watcher.evaluate();
		}
	};
}
```

注意其中的`watcher.depend()`,该方法让用到该属性的 watcher 观察者订阅该 watcher 中的依赖，且该计算属性 watcher 会将订阅它的 watcher 推入他的 subs 中(当计算属性值改变的时候，通知订阅他的 watcher 观察者)
`watcher.evaluate()`，该方法是通过调用 watcher 的 get 方法(其中需要注意的是 watcher 的 get 方法会调用 pushTarget 将之前的 Dep.target 实例入栈，并设置 Dep.target 为该 computed watcher,被该计算属性依赖的响应式属性会将该 computed watcher 推入其 subs 中，所以当被依赖的响应式属性改变时，会通知订阅他的 computed watcher,computed watcher 再通知订阅该计算属性的 watcher 调用 update 方法)，get 方法中调用计算属性 key 绑定的 handler 函数计算出值。

### **initWatch**

该 watcher 为 user watcher（开发人员自己在组件中自定义的）。
initWatch 的作用是遍历 watch 中的属性，并对每个 watch 监听的属性调用定义的\$watch

```javascript
Vue.prototype.$watch = function(
	expOrFn: string | Function,
	cb: any,
	options?: Object
): Function {
	const vm: Component = this;
	if (isPlainObject(cb)) {
		return createWatcher(vm, expOrFn, cb, options);
	}
	options = options || {};
	options.user = true; // 代表该watcher是用户自定义watcher
	const watcher = new Watcher(vm, expOrFn, cb, options);
	if (options.immediate) {
		cb.call(vm, watcher.value);
	}
	return function unwatchFn() {
		watcher.teardown();
	};
};
```

代码中调用 new Watcher 的时候，也会同 render watcher 一样，执行下 watcher 的 get 方法，调用`pushTarget`将当前 user watcher 赋值给 Dep.target,get()中`value = this.getter.call(vm, vm)`这个语句会触发该自定义 watcher 监听的响应式属性的 get 方法，并将当前的 user watcher 推入该属性依赖的 subs 中，所以当 user watcher 监听的属性 set 触发后，通知订阅该依赖的 watcher 去触发 update，也就是触发该 watch 绑定的 key 对应的 handler。然后就是调用 popTarget 出栈并赋值给 Dep.target。

## \$mount

initState 初始化工作大致到这里过，接下去会执行$mount开始渲染工作
$mount 主要工作：new 了一个渲染 Watcher，并将 updateCompent 作为 callback 传递进去并执行

```javascript
updateComponent = () => {
	vm._update(vm._render(), hydrating);
};
new Watcher(
	vm,
	updateComponent,
	noop,
	{
		before() {
			if (vm._isMounted) {
				callHook(vm, 'beforeUpdate');
			}
		}
	},
	true /* isRenderWatcher */
);
```

三种 watcher 中 new Watcher 的时候，只有 computed watcher 不会一开始就执行它的 get()方法。\$mount 里面 new 的这个 render watcher 会调用`get()`方法，调用`pushTarget`将当前 render watcher 赋值给 Dep.target。接下去重头戏来了，调用`updateComponent`,该方法会执行`vm._update(vm._render(), hydrating)`，其中 render 函数会触发 html 中使用到的响应式属性的 get 钩子。get 钩子会让该响应式属性的依赖实例 dep 将当前的 render watcher 推入其 subs 数组中，所以当依赖的响应式属性改变之后，会遍历 subs 通知订阅它的 watcher 去调用 update()。

## 例子

可能大家对 watcher 和 dep 调来调去一头雾水，我讲个实例

```javascript
<div id='app'>
	<div>{{ a }}</div>
	<div>{{ b }}</div>
</div>;
new Vue({
	el: '#app',
	data() {
		return {
			a: 1
		};
	},
	computed: {
		b() {
			return a + 1;
		}
	}
});
```

我直接从渲染开始讲，只讲跟 dep 跟 watcher 有关的
**\$mount**：new 一个渲染 watcher（watcher 的 get 方法中会将渲染 watcher 赋值给 Dep.target）的时候会触发 `vm._update(vm._render(), hydrating)`，render 的时候会获取 html 中用到的响应式属性，上面例子中先用到了 a,这时会触发 a 的 get 钩子,其中`dep.depend()`会将当前的渲染 watcher 推入到 a 属性的 dep 的 subs 数组中。
接下去继续执行，访问到 b（b 是计算属性的值），会触发计算属性的 get 方法。计算属性的 get 方法是调用`createComputedGetter`函数后的返回函数`computedGetter`，`computedGetter`函数中会执行`watcher.depend()`。Watcher 的 depend 方法是专门留给 computed watcher 使用的。刚才上面说过了除了 computed watcher，其他两种 watcher 在 new 完之后都会执行他们的 get 方法，那么 computed watcher 在 new 完之后干嘛呢，它会 new 一个 dep。回到刚才说的专门为 computed watcher 开设的方法`watcher.depend()`，他的作用是执行`this.dep.depend()`（computed watcher 定义的 dep 就是在这里使用到的）。`this.dep.depend()`会让当前的渲染 watcher 订阅该计算属性依赖，该计算属性也会将渲染 watcher 推入到它自己的 subs（[render watcher]）中，当计算属性的值修改之后会通知 subs 中的 watcher 调用`update()`,所以计算属性值变了页面能刷新。回到前面说的触发 b 计算属性的 get 钩子那里，get 钩子最后会执行`watcher.evaluate()`,`watcher.evaluate()`会执行 computed watcher 的`get()`方法。这时候重点来了，会将 Dep.target（render watcher）推入 targetStack 栈中（存入之后以便待会儿取出继续用），然后将这个计算属性的 computed watcher 赋值给 Dep.target。get 方法中`value = this.getter.call(vm, vm)`,会执行 computed 属性绑定的 handler。如上面例子中 return a + 1。使用了 a 那么就一定会触发 a 的 get 钩子，get 钩子又会调用`dep.depend()`，dep.depend()会让 computed watcher 将 dep 存入它的 deps 数组中，a 的 dep 会将**当前的 Dep.target(computed watcher)存入其 subs 数组中，当前例子中 a 的 subs 中就会是[render watcher,computed watcher]**,所以 a 值变化会遍历 a 的 subs 中的 watcher 调用`update()`方法，html 中用到的 a 会刷新，计算属性 watcher 调用`update()`方法会通知他自己的 subs（[render watcher]）中 render watcher 去调用 update 方法，html 中用到的计算属性 b 才会刷新 dom（这里提个醒，我只是粗略的讲，计算属性依赖的属性变化后他不一定会触发更新，他会比较计算完之后的值是否变化）。computed watcher 的 get()方法最后会调用`popTarget()`,将之前存入 render watcher 出栈并赋值给 Dep.target，这时候我例子中 targetStack 就变成空数组了。render watcher 的 get 方法执行到最后也会出栈，这时候会将 Dep.target 赋值会空。
