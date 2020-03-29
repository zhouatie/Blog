---
title: 阻止中文输入法输入拼音的时候触发input事件
date: 2018-11-12 01:12:21
categories: 前端
tags: ['js']
thumbnail: https://s1.ax1x.com/2020/03/30/GenCZt.png
---

## 前言

最近看 element-ui 源码的时候看到 el-input 发现的这个少见的事件。

<!--more-->

### `compositionstart`、`compositionend`事件（MDN 解释)

`compositionstart`事件触发于一段文字的输入之前（类似于 keydown 事件，但是该事件仅在若干可见字符的输入之前，而这些可见字符的输入可能需要一连串的键盘操作、语音识别或者点击输入法的备选词）。
当文本段落的组成完成或取消时, `compositionend` 事件将被触发 (具有特殊字符的触发, 需要一系列键和其他输入, 如语音识别或移动中的字词建议)。

```javascript
/**
 * @param {Element} elem input元素
 * @param {Function} callback input事件绑定的回调
 */
function inputEvent(elem, callback) {
	let isOnComposition = false;
	elem.addEventListener('compositionstart', function(event) {
		isOnComposition = true;
	});
	elem.addEventListener('compositionend', function(event) {
		isOnComposition = false;
		const val = event.target.value;
		handleInput(val);
	});
	elem.addEventListener('input', function(event) {
		const val = event.target.value;
		handleInput(val);
	});
	function handleInput(val) {
		if (isOnComposition) return;
		callback(val);
	}
}

window.onload = function() {
	const input = document.getElementById('input');
	inputEvent(input, function(val) {
		console.log(val);
	});
};
```
