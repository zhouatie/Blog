---
title: javaScript排序算法学习笔记
date: 2019-03-12 01:17:42
categories: 前端
tags: ['js']
thumbnail: https://s1.ax1x.com/2020/03/30/Genae1.png
---

直接上demo

<!-- more -->

```javascript
// 用于创建数组
function createNonSortedArray(size) {
	var array = new ArrayList();
	for (var i = size; i > 0; i--) {
		array.insert(i);
	}
	return array;
}

function ArrayList() {
	var array = [];

	this.insert = function(item) {
		console.log(item, 'insert');
		array.push(item);
	};

	this.toString = function() {
		console.log('tostring');
		return array.join();
	};
	/*
	 * 冒泡排序
	 * 冒泡排序比较任何两个相邻的项，如果第一个比第二个大，则交换它们。
	 * 元素项向上移动至正确的顺序，就好像气泡升至表面一样，冒泡排序因此得名。
	 * 第一轮比过之后最后一个就一定是最大的 无需再比较。所以下次要 - i
	 */

	this.bubbleSort = function() {
		var length = array.length;
		for (var i = 0; i < length; i++) {
			for (var j = 0; j < length - 1 - i; j++) {
				if (array[j] > array[j + 1]) {
					swap(array, j, j + 1);
				}
			}
		}
	};

	/*
	 * 选择排序
	 * 选择排序算法是一种原址比较排序算法。选择排序大致的思路是找到数据结构中的最小值
	 * 并将其放置在第一位，接着找到第二小的值并将其放在第二位，以此类推。比如，第一个
	 * 的时候，会遍历后面所有想跟其比较，找到最小的更其交换。所以第一个此时一定是最小的。
	 * 随意第二个的时候，只会循环后面的几个。如果找到一个比第二个更小的，那么交换位置。
	 */

	this.selectionSort = function() {
		var length = array.length,
			indexMin;
		for (var i = 0; i < length - 1; i++) {
			indexMin = i;
			for (var j = i; j < length; j++) {
				if (array[indexMin] > array[j]) {
					indexMin = j;
				}
			}
			if (i !== indexMin) {
				swap(array, i, indexMin);
			}
		}
	};

	/*
	 * 插入排序
	 * 插入排序每次排第一个数组项，以此方式构建最后的排序数组。假定第一项已经排序了，接着，
	 * 它和第二项进行比较，第二项是应该待在原位还是插到第一项之前呢？这样，头两项就已正确
	 * 排序，接着和第三项比较（它是该插入到第一、第二还是第三位置呢？），以此类推。
	 * 简而言之，就是遍历数组的每一项，拿这一项去跟前面的项比较，如果比他小就插入到它前面。
	 */

	this.insertionSort = function() {
		var length = array.length,
			j,
			temp;
		for (var i = 1; i < length; i++) {
			j = i;
			temp = array[i];
			while (j > 0 && array[j - 1] > temp) {
				array[j] = array[j - 1];
				j--;
			}
			array[j] = temp;
		}
	};

	// 归并排序
	// 归并排序是一种分治算法。其思想是将原始数组切分成较小的数组，直到每个
	// 小数组只有一个位置，接着将小数组归并成较大的数组，直到最后一个排序完毕的大数组。
	this.mergeSort = function() {
		array = mergeSortRec(array);
	};

	var mergeSortRec = function(array) {
		var length = array.length;
		if (length === 1) {
			return array;
		}
		var mid = Math.floor(length / 2),
			left = array.slice(0, mid),
			right = array.slice(mid, length);

		return merge(mergeSortRec(left), mergeSortRec(right));
	};

	var merge = function(left, right) {
		var result = [],
			il = 0,
			ir = 0;
		// 完成下列操作的前提是left、right数组均已经完成。所以采用递归的形式
		// 在数组长度为1的时候先开始排序，然后在通过merge left与right数组
		while (il < left.length && ir < right.length) {
			if (left[il] < right[ir]) {
				result.push(left[il++]);
			} else {
				result.push(right[ir++]);
			}
		}
		// 上面是将left与right数组排完序，那么其中之一数组必然为空，
		// 下面的操作就是将剩下的right或者left全部推入result数组中
		while (il < left.length) {
			result.push(left[il++]);
		}

		while (ir < right.length) {
			result.push(right[ir++]);
		}

		return result;
	};

	// 快速排序
	// 首先，从数组中选择中间一项作为主元
	// 创建两个指针，左边一个指向数组的第一项，右边一个指向数组的最后一个项。
	// 移动左指针直到我们找到一个比主元大的元素，接着，移动右指针直到找到一个
	// 比主元小的元素，然后交换他们，重复这个过程，直到左指针超过右指针。这个
	// 过程将使得比主元小的值都排在主元之前，而比主元大的值都排在主元之后。这一步
	// 叫做划分操作。
	// 接着，算法对划分后的小数组（较主元小的值组成的子数组，以及较主元大的值
	// 组成的子数组）重复之前的两个步骤，直到数组已完全排序。

	// 简而言之，先分治，不断的细化下去，到最后一个数组无法再交换位置进行排序位置
	this.quickSort = function() {
		quick(array, 0, array.length - 1);
	};

	var quick = function(array, left, right) {
		var index;
		if (array.length > 1) {
			index = partition(array, left, right);
			if (left < index - 1) {
				quick(array, left, index - 1);
			}
			if (index < right) {
				quick(array, index, right);
			}
		}
	};

	var partition = function(array, left, right) {
		var pivot = array[Math.floor((left + right) / 2)],
			i = left,
			j = right;

		while (i <= j) {
			while (array[i] < pivot) {
				i++;
			}
			while (array[j] > pivot) {
				j--;
			}
			if (i <= j) {
				swap(array, i, j);
				i++;
				j--;
			}
		}

		return i;
	};

	var swap = function(array, index1, index2) {
		var aux = array[index1];
		array[index1] = array[index2];
		array[index2] = aux;
	};
}

var array = createNonSortedArray(9);
console.log(array.toString());
array.bubbleSort();
// array.selectionSort();
// array.insertionSort();
// array.mergeSort();
// array.quickSort();
console.log(array.toString());
```
