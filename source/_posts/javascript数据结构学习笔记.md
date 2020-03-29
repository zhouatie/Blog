---
title: javascript数据结构学习笔记
date: 2019-03-04 01:15:02
categories: 前端
tags: ['js']
thumbnail: https://s1.ax1x.com/2020/03/30/Gen8WF.png
---

## 数组

<!-- more -->

### 方法

```javascript
//一、数组
var arr = [];
// 添加元素
arr.push(1, 2); // [1,2]
// 开头插入
arr.unshift(0); // [0, 1, 3]
// 尾部删除
arr.pop(); // [0, 1]
// 头部删除
arr.shift(); // [1]
// 数组合并
[1].concat([2]); // [1,2]
```

### 迭代器

- `every` every 方法会迭代数组中每个元素，直到返回 false。
- `some` some 和 every 类似，不过 some 方法会迭代数组的每个元素，直到函数返回 true
- `forEach` 和 for 循环的结果相同
- `map` 返回新的数组 `[1,2].map(o => o * 2) // [2,4]`
- `filter` 返回新的数组 `[1,2].filter(o => o > 1) // [2]`
- `reduce` `[1,2].reduce((result, current) => result + current) // 3`
- `for of` `for (let n of numbers) { console.log((n % 2 === 0) ? 'even' : 'odd')};`
- `entries`
  ```javascript
  const numbers = [1, 2, 3];
  let aEntries = numbers.entries(); // 得到键值对的迭代器
  console.log(aEntries.next().value); // [0, 1] 位置0的值为1
  console.log(aEntries.next().value); // [1, 2] 位置1的值为2
  console.log(aEntries.next().value); // [2, 3] 位置2的值为3
  ```
- `keys`

  ```javascript
  const numbers = [1, 2, 3];
  console.log(Object.keys(numbers)); // ['0','1','2'];
  ```

- `values`

  ```javascript
  const numbers = [1, 2, 3];
  console.log(Object.values(numbers)); // [1,2,3]
  ```

- `Array.from`
- `Array.of`
- `fill`
- `copyWithin`
- `sort`
- `find`
- `findIndex`
- `includes`

## 栈

> 栈是一种遵从**后进先出**原则的有序集合

### 实现

```javascript
function Stack() {
	let items = [];
	// 向栈添加元素
	this.push = function(element) {
		items.push(element);
	};
	// 从栈移除元素
	this.pop = function() {
		return items.pop();
	};
	// 查看栈顶元素
	this.peek = function() {
		return items[item.length - 1];
	};
	// 检查栈是否为空
	this.isEmpty = function() {
		return items.length == 0;
	};
	this.size = function() {
		return items.length;
	};
	// 清空和打印栈元素
	this.clear = function() {
		items = [];
	};
	this.print = function() {
		console.log(items.toString());
	};
}
```

### 用栈解决问题

存储访问过的任务或路径、撤销的操作等。

## 队列

> 队列是遵循 FIFO(First In First Out, 先进先出，也称为先来先服务)

### 实现

```javascript
function Queue() {
	let items = [];
	// 向队列添加元素
	this.enqueue = function(element) {
		items.push(element);
	};
	// 从队列移除元素
	this.dequeue = function() {
		return items.shift();
	};
	// 查看队列头元素
	this.front = function() {
		return items[0];
	};
	// 检查队列是否为空
	this.isEmpty = function() {
		return items.length == 0;
	};
	this.size = function() {
		return items.length;
	};
	// 打印队列元素
	this.print = function() {
		console.log(items.toString());
	};
}
```

## 链表

> 链表村粗有序的元素集合，但不同于数组，链表中的元素在内存中并不是连续放置的。每个元素由一个存储元素本身的节点和一个指向下一个元素的引用（也称指针或链接）组成。
> 相对于传统的数组，链表的一个好处在于，添加或移除元素的时候不需要移动其他元素。然而，链表需要使用指针，因此实现链表时需要额外注意。数组的另一个细节是可以直接访问任何位置的任何元素，而要想访问链表中间的一个元素，需要从起点（表头）开始迭代列表直到找到所需的元素。

### 实现

```javascript
function LinkedList() {
    let Node = function(element) {
        this.element = element;
        this.next = null;
    };

    let length = 0;
    let head = null;
    // 向链表尾部追加元素
    this.append = function(element) {
        let node = new Node(element),
        current;

        if (head === null) {
            head = node;
        } else {
            current = head;
            // 循环列表，直到找到最后一项
            while (current.next) {
                current = current.next;
            }
            // 找到最后一项，将其next赋为node，建立链接
            current.next = node;
        }
        length++； // 更新列表的长度
    }
    // 从链表中移除元素
    this.removeAt = function() {
        // 检查越界值
        if (position > -1 && position < length) {
            let current = head,
            previous,
            index = 0;

            // 移除第一项
            if (position === 0) {
                head = current.next;
            } else {
                while (index++ < position) {
                    previous = current;
                    current = current.next;
                }
                // 将previous 与 current的下一项链接起来： 跳过current，从而移除它
                previous.next = current.next;
            }
            length--;
            return current.element;
        } else {
            return null;
        }
    }
    // 在任意位置插入元素
    this.insert = function(position, element) {
        // 检查越界值
        if (position >= 0 && position <= length) {
            let node = new Node(element),
            current = head,
            previous,
            index = 0;

            if (position === 0) { // 在第一个位置添加
                node.next = current;
                head = node;
            } else {
                while (index++ < position) {
                    previous = current;
                    current = current.next;
                }
                node.next = current;
                previous.next = node;
            }
            length++; // 更新列表的长度
            return true;
        } else {
            return false;
        }
    }
    // toString方法
    this.toString = function() {
        let current = head,
        string = '';

        while (current) {
            string += current.element + (current.next ? 'n' : '');
            current = current.next;
        }

        return string;
    }
    // indexOf 方法
    this.indexOf = function(elment) {
        let current = head,
        index = 0;

        while(current) {
            if (element === current.element) {
                return index;
            }
            index++;
            current = current.next;
        }

        return -1;
    }
    // remove 方法
    this.remove = function(elment) {
        let index = this.indexOf(element);
        return this.removeAt(index);
    }
    // isEmpty 方法
    this.isEmpty = function() {
        return length == 0;
    }
    // size 方法
    this.size = function() {
        return length;
    }
    // getHead 方法
    this.getHead = function() {
        return head;
    }
}

```

### 双向链表（留给大家自己思考）

## 集合

> 集合是由一组无序且唯一（即不能重复）的项组合的。这个数据结构使用了与有限集合相同的数学概念，但应用在计算机科学的数据结构中。

```javascript
function Set() {
	let items = {};
	// has 方法
	this.has = function(value) {
		return items.hasOwnProperty(value);
	};
	// add 方法
	this.add = function(value) {
		if (!this.has(value)) {
			items[value] = value;
			return true;
		}
		return false;
	};
	// remove 方法
	this.remove = function(value) {
		if (this.has(value)) {
			delete items[value];
			return true;
		}
		return false;
	};
	// clear 方法
	this.clear = function() {
		items = {};
	};
	// size 方法
	this.size = function() {
		return Object.keys(items).length;
	};
	// values 方法
	this.values = function() {
		let values = [];
		for (let i = 0, keys = Object.keys(items); i < keys.length; i++) {
			values.push(items[keys[i]]);
		}
		return values;
	};
	// 并集
	this.union = function(otherSet) {
		let unionSet = new Set();

		let values = this.values();
		for (let i = 0; i < values.length; i++) {
			unionSet.add(values[i]);
		}

		values = otherSet.values();
		for (let i = 0; i < values.length; i++) {
			unionSet.add(values[i]);
		}

		return unionSet;
	};
	// 交集
	this.intersection = function(otherSet) {
		let intersectionSet = new Set();

		let values = this.values();
		for (let i = 0; i < values.length; i++) {
			if (otherSet.has(values[i])) {
				intersectionSet.add(values[i]);
			}
		}

		return intersectionSet;
	};
	// 差集
	this.difference = function(otherSet) {
		let differenceSet = new Set();

		let values = this.values();
		for (let i = 0; i < values.length; i++) {
			if (!otherSet.has(values[i])) {
				differenceSet.add(values[i]);
			}
		}

		return differenceSet;
	};
	// 子集
	this.subset = function(otherSet) {
		if (this.size() > otherSet.size()) {
			return false;
		} else {
			let values = this.values();
			for (let i = 0; i < values.length; i++) {
				if (!otherSet.has(values[i])) {
					return false;
				}
			}
			return true;
		}
	};
}
```

## 字典和散列表

### 实现

```javascript
function Dictionary() {
	var items = {};
	// has 和 set 方法
	this.has = function(key) {
		return items.hasOwnProperty(key);
	};
	this.set = function(key, value) {
		item[key] = value;
	};
	// delete 方法
	this.delete = function(key) {
		if (this.has(key)) {
			delete items[key];
			return true;
		}
		return false;
	};
	// get 和 values 方法
	this.get = function(key) {
		return this.has(key) ? items[key] : undefined;
	};
	this.values = function() {
		var values = [];
		for (var k in items) {
			if (this.has(k)) {
				values.push(items[k]);
			}
		}

		return values;
	};
	// clear 方法
	this.clear = function() {
		items = {};
	};
	// size 方法
	this.size = function() {
		return Object.keys(items).length;
	};
	// keys 方法
	this.keys = function() {
		return Object.keys(items);
	};
	// getItems 方法
	this.getItems = function() {
		return items;
	};
}
```

### 散列表

> HashTable 类 也叫 HashMap 类，它是 Dictionary 类的一种散列表是实现方式。
> 散列算法的作用是尽可能快的在数据结构中找到一个值。

```javascript
function HashTable() {
	var table = [];
	var loseloseHashCode = function(key) {
		var hash = 0;
		for (var i = 0; i < key.length; i++) {
			hash += key.charCodeAt(i);
		}
		return hash % 37;
	};
	this.put = function(key, value) {
		var position = loseloseHashCode(key);
		console.log(position + ' - ' + key);
		table[position] = value;
	};
	this.get = function(key) {
		return table[loseloseHashCode(key)];
	};
	this.remove = function(key) {
		table[loseloseHashCode(key)] = undefined;
	};
}
```

### Map 类

> es6 新增了 Map 类

```javascript
var map = new Map();

map.set('a', 'b');

console.log(map.has('a')); // true
console.log(map.size()); // 输出1
console.log(map.keys()); // ['a']
console.log(map.values()); // ['b'];

// 和Dictionary类不同，es6的Map类的values方法和keys方法都返回Iterator，而不是值或键构成的数组。
```

### es6 --- WeakMap 类 和 WeakSet 类

- WeakMap 和 WeakSet 类没有 entries keys values 等方法
- 只能用对象作为键

```javascript
var map = new WeakMap();
var obj = { name: 'a' };
map.set(obj, 'b');

console.log(map.has(obj)); // 输出true
console.log(map.get(obj)); // 输入'b'
map.delete(obj);
```

## 树

> 一个树结构包含一系列存在父子关系的节点。每个节点都有一个父节点（除了顶部的第一个节点）以及零个或多个子节点；

### 二叉树和二叉搜索树

```javascript
function BinarySearchTree() {
	var Node = function(key) {
		this.key = key;
		this.left = null;
		this.right = null;
	};

	var root = null;

	var insertNode = function(node, newNode) {
		if (newNode.key < node.key) {
			if (node.left === null) {
				node.left = newNode;
			} else {
				insertNode(node.left, newNode);
			}
		} else {
			if (node.right === null) {
				node.right = newNode;
			} else {
				insertNode(node.right, newNode);
			}
		}
	};
	// 向树中插入一个键
	this.insert = function(key) {
		var newNode = new Node(key);

		if ((root = null)) {
			root = newNode;
		} else {
			insertNode(root, newNode);
		}
	};

	var inOrderTraverseNode = function(node, callback) {
		if (node !== null) {
			inOrderTraverseNode(node.left, callback);
			callback(node.key);
			inOrderTraverseNode(node.right, callback);
		}
	};

	// 中序遍历
	this.inOrderTraverse = function(callback) {
		inOrderTraverseNode(root, callback);
	};

	var preOrderTraverseNode = function(node, callback) {
		if (node !== null) {
			callback(node.key);
			preOrderTraverseNode(node.left, callback);
			preOrderTraverseNode(node.right, callback);
		}
	};

	// 先序遍历
	this.preOrderTraverse = function(callback) {
		preOrderTraverseNode(root, callback);
	};

	var postOrderTraverseNode = function(node, callback) {
		if (node !== null) {
			postOrderTraverseNode(node.left, callback);
			postOrderTraverseNode(node.right, callback);
			callback(node.key);
		}
	};

	// 后序遍历
	this.postOrderTraverse = function(callback) {
		postOrderTraverseNode(root, callback);
	};

	// 搜索最小值
	this.min = function() {
		return minNode(root);
	};

	var minNode = function(node) {
		if (node) {
			while (node && node.left !== null) {
				node = node.left;
			}
			return node.key;
		}
		return null;
	};

	// 搜索最大值
	this.max = function() {
		return maxNode(root);
	};

	var maxNode = function(node) {
		if (node) {
			while (node && node.right !== null) {
				node = node.right;
			}
			return node.key;
		}
		return null;
	};

	// 搜索一个特定的值
	this.search = function(key) {
		return searchNode(root, key);
	};

	var searchNode = function(node, key) {
		if (node === null) {
			return false;
		}
		if (key < node.key) {
			return searchNode(node.left, key);
		} else if (key > node.key) {
			return searchNode(node.right, key);
		} else {
			return true;
		}
	};

	// 移除一个节点
	this.remove = function(key) {
		root = removeNode(root, key);
	};

	var removeNode = function(node, key) {
		if (node === null) {
			return null;
		}
		if (key < node.key) {
			node.left = removeNode(node.left, key);
			return node;
		} else if (key > node.key) {
			node.right = removeNode(node.right, key);
			return node;
		} else {
			// 键等于node.key
			// 第一种情况--一个叶节点
			if (node.left === null && node.right === null) {
				node = null;
				return node;
			}

			// 第二种情况--一个只有一个子节点的节点
			if (node.left === null) {
				node = node.right;
				return node;
			} else if (node.right === null) {
				node = node.left;
				return node;
			}

			// 第三种情况---- 一个有两个子节点的节点
			var aux = findMinNode(node.right);
			node.key = aux.key;
			node.right = removeNode(node.rihgt, aux.key);
			return node;
		}

		var findMinNode = function(node) {
			while (node && node.left !== null) {
				node = node.left;
			}
			return node;
		};
	};
}
```

### 自平衡树（AVL)

> 当树很深的时候，添加移除和搜索某个节点时引起一些性能问题。

```JavaScript
var heightNode = function(node) {
    if (node === null) {
        return -1;
    } else {
        return Math.max(heightNode(node.left), heightNode(node.right)) + 1;
    }
}

var rotationRR = function(node) {
    var tmp = node.right;
    node.right = tmp.left;
    tmp.left = node;
    return tmp;
}
var rotationLL = function(node) {
    var tmp = node.left;
    node.left = tmp.right;
    tmp.right = node;
    return tmp;
}

var rotationLR = function(node) {
    node.left = rotationRR(node.left);
    return rotationLL(node);
}

var rotationRL = function(node) {
    node.right = rotationLL(node.right);
    return rotationRR(node);
}

var insertNode = function(node, element) {
    if (node === null) {
        node = new Node(element);
    } else if (element < node.key) {
        node.left = insertNode(node.left, element);

        if (node.left !== null) {
            // 确认是否需要平衡
            if ((heightNode(node.left) - heightNode(node.right) > 1)) {
                if (element < node.left.key) {
                    node = rotationLL(node);
                } else {
                    node = rotationLR(node);
                }
            }
        }
    } else if (element > node.key) {
        node.right = insertNode(node.right, element);

        if (node.right !== null) {
            // 确认是否需要平衡
            if ((heightNode(node.right) - heightNode(node.left) > 1)) {
                if (element > node.right.key) {
                    node = rotationRR(node);
                } else {
                    node = rotationRL(node);
                }
            }
        }
    }
    return node;
}

```

## 图

> 图是网络结构的抽象模型，图是一组由边连接的节点（或顶点）。学习图是重要的，因为任何关系都可以用图来表示

```javascript
function Graph() {
	var vertices = [];
	var adjList = new Dictionary();

	this.addVertex = function(v) {
		vartices.push(v);
		adjList.set(v, []);
	};

	this.addEdge = function(v, w) {
		addList.get(v).push(w);
		addList.get(w).push(v);
	};

	this.toString = function() {
		var s = '';
		for (var i = 0; i < vertices.length; i++) {
			s += vertices[i] + ' -> ';
			var neighbors = adjList.get(vertices[i]);
			for (var j = 0; j < neighbors.length; j++) {
				s += neighbors[j] + ' ';
			}
			s += '\n';
		}
		return s;
	};

	// 广度优先搜索
	var initializeColor = function() {
		var color = [];
		for (var i = 0; i < vertices.length; i++) {
			color[vertices[i]] = 'white';
		}
		return color;
	};

	this.bfs = function(v, callback) {
		var color = initializeColor(),
			queue = new Queue();
		queue.enqueue(v);

		while (!queue.isEmpty()) {
			var u = queue.dequeue(),
				neighbors = adjList.get(u);
			color[u] = 'grey';
			for (var i = 0; i < neighbors.length; i++) {
				var w = neighbors[i];
				if (color[w] === 'white') {
					color[w] = 'grey';
					queue.enqueue(w);
				}
			}
			color[u] = 'black';
			if (callback) {
				callback();
			}
		}
	};

	// 使用BFS寻找最短路径
	this.BFS = function(v) {
		var color = initializeColor(),
			queue = new Queue(),
			d = [];
		pred = [];
		queue.enqueue(v);

		for (var i = 0; i < vertices.length; i++) {
			d[vertices[i]] = 0;
			pred[vertices[i]] = null;
		}

		while (!queue.isEmpty()) {
			var u = queue.dequeue(),
				neighbors = adjList.get(u);
			color[u] = 'grey';
			for (i = 0; i < neighbors.length; i++) {
				var w = neighbors[i];
				if (color[w] === 'white') {
					color[w] = 'grey';
					d[w] = d[u] + 1;
					pred[w] = u;
					queue.enqueue(w);
				}
			}
			color[u] = 'black';
		}
		return {
			distances: d,
			predecessors: pred
		};
	};

	// 深度优先遍历
	var dfsVisit = function(u, color, callback) {
		color[u] = 'grey';
		if (callback) {
			callback(u);
		}
		var neighbors = adjList.get(u);
		for (var i = 0; i < neighbors.length; i++) {
			var w = neighbors[i];
			if (color[w] === 'white') {
				dfsVisit(w, color, callback);
			}
		}
		color[u] = 'black';
	};
	this.dfs = function(callback) {
		var color = initializeColor();

		for (var i = 0; i < vertices.length; i++) {
			if (color[vertices[i]] === 'white') {
				dfsVisit(vertices[i], color, callback);
			}
		}
	};

	// 探索深度优先算法
	var time = 0;
	this.DFS = function() {
		var color = nitializeColor(),
			d = [],
			f = [],
			p = [],
			time = 0;

		for (var i = 0; i < vertices.length; i++) {
			f[vertices[i]] = 0;
			d[vertices[i]] = 0;
			p[vertices[i]] = null;
		}

		for (i = 0; i < vertices.length; i++) {
			if (color[vertices[i]] === 'white') {
				DFSVisit(vertices[i], color, d, f, p);
			}
		}

		return {
			discovery: d,
			finished: f,
			predecessors: p
		};
	};

	var DFSVisit = function(u, color, d, f, p) {
		console.log('discovered ' + u);
		color[u] = 'grey';
		d[u] = ++time;
		var neighbors = adjList.get(u);
		for (var i = 0; i < neighbors.length; i++) {
			var w = neighbors[i];
			if (color[w] === 'white') {
				p[w] = u;
				DFSVisit(w, color, d, f, p);
			}
		}
		color[u] = 'black';
		f[u] = ++time;
		console.log('explored ' + u);
	};
}
```
