---
title: 手写Promise
date: 2019-09-12 01:54:02
categories: 前端
tags: ['promise']
thumbnail: https://s1.ax1x.com/2020/03/30/Gelose.png
---

😆

<!--more-->

```javascript
class TPromise {
	static PENDING = 'PENDING';
	static RESOLVED = 'RESOLVED';
	static REJECTED = 'REJECTED';

	constructor(handler) {
		// 下面队列用于存储方法
		this.resolveQueues = [];
		this.rejectQueues = [];
		this.finallyQueues = [];
		this.state = TPromise.PENDING;
		handler(this._resolve.bind(this), this._reject.bind(this));
	}

	_resolve(val) {
		// 因为promise是微任务，这里使用MutationObserver来模拟微任务
		let observer = new MutationObserver(() => {
			// 一旦状态机修改过状态，那么就无法再改变状态
			if (this.state !== TPromise.PENDING) return;
			this.state = TPromise.RESOLVED;
			this.value = val;
			let handler;
			while ((handler = this.resolveQueues.shift())) {
				handler(this.value);
			}
			while ((handler = this.finallyQueues.shift())) {
				handler();
			}
		});

		observer.observe(document.body, {
			attributes: true
		});
		document.body.setAttribute('_promise', Date.now());
	}

	_reject(val) {
		// 因为promise是微任务，这里使用MutationObserver来模拟微任务
		let observer = new MutationObserver(() => {
			// 一旦状态机修改过状态，那么就无法再改变状态
			if (this.state !== TPromise.PENDING) return;
			this.state = TPromise.REJECTED;
			this.value = val;
			let handler;
			while ((handler = this.rejectQueues.shift())) {
				handler(this.value);
			}
			while ((handler = this.finallyQueues.shift())) {
				handler();
			}
		});

		observer.observe(document.body, {
			attributes: true
		});
		document.body.setAttribute('_promise', Date.now());
	}

	then(resolveHandler, rejectHandler) {
		// 每一个then都是返回一个新的promise
		return new TPromise((resolve, reject) => {
			if (typeof resolveHandler === 'function') {
				const newResolveHandler = val => {
					let result = resolveHandler(val);
					if (result instanceof TPromise) {
						// 如果返回的结果是个promise实例的话，调用下面方法
						result.then(resolve, reject);
					} else {
						resolve(result);
					}
				};
				// promise每次resolve的时候，都会去执行resolveQueues队列中的所有方法
				this.resolveQueues.push(newResolveHandler);
			} else {
				// promise每次resolve的时候，都会去执行resolveQueues队列中的所有方法
				this.resolveQueues.push(resolve);
			}

			if (typeof rejectHandler === 'function') {
				const newRejectHandler = val => {
					let result = rejectHandler(val);
					if (result instanceof TPromise) {
						result.then(resolve, reject);
					} else {
						resolve(result);
					}
				};
				this.rejectQueues.push(newRejectHandler);
			} else {
				this.rejectQueues.push(reject);
			}
		});
	}

	catch(rejectHandler) {
		// catch 实质上是then的一种简写
		return this.then(undefined, rejectHandler);
	}

	finally(finallyHandler) {
		return new TPromise((resolve, reject) => {
			if (typeof finallyHandler === 'function') {
				const newFinallyHandler = () => {
					const result = finallyHandler();
					if (result instanceof TPromise) {
						result.finally(() => {
							if (this.state === TPromise.RESOLVED) resolve(this.value);
							else if (this.state === TPromise.REJECTED) reject(this.value);
						});
					} else {
						if (this.state === TPromise.RESOLVED) resolve(this.value);
						else if (this.state === TPromise.REJECTED) reject(this.value);
					}
				};
				this.finallyQueues.push(newFinallyHandler);
			}
		});
	}

	// all 只有当所有promise都成功返回时，才resolve
	static all(arr) {
		let i = 0;
		const resArr = [];
		return new TPromise((resolve, reject) => {
			arr.forEach((it, index) => {
				it.then(res => {
					i++;
					resArr[index] = res;
					if (i >= arr.length) {
						resolve(resArr);
					}
				}).catch(err => {
					reject(err);
				});
			});
		});
	}

	// race 返回第一个有结果的promise
	static race(arr) {
		return new TPromise((resolve, reject) => {
			arr.forEach((it, index) => {
				it.then(res => {
					resolve(res);
				}).catch(err => {
					reject(err);
				});
			});
		});
	}

	static resolve(val) {
		return new TPromise((resolve, reject) => {
			if (val instanceof TPromise) {
				val.then(resolve, reject);
			} else {
				resolve(val);
			}
		});
	}

	// 注意reject 会原封不动的将参数返回
	static reject(val) {
		return new TPromise((resolve, reject) => {
			reject(val);
		});
	}
}
```
