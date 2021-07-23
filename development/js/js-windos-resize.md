---
title: js-windows-resize
date: 2021-01-31 20:00:00
tags: 'JavaScript'
categories:
  - ['开发', 'JavaScript']
permalink: js-windows-resize
mathjax: false
---

https://juejin.cn/post/6844903961888047112

[! [](data: image/png; base64, iVBORw0KGgoAAAANSUhEUgAAADwAAAA8CAYAAAA6/NlyAAAAAXNSR0IArs4c6QAAAERlWElmTU0AKgAAAAgAAYdpAAQAAAABAAAAGgAAAAAAA6ABAAMAAAABAAEAAKACAAQAAAABAAAAPKADAAQAAAABAAAAPAAAAACL3+ lcAAAAV0lEQVRoBe3QMQEAAADCoPVP7WkJiEBhwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGPjAADh8AAFUUgPVAAAAAElFTkSuQmCC)](/user/3298190614879704)

2019 年 10 月 12 日 阅读 1752

关注

# JS 监听窗口变化和元素变化

# resize 事件

## 简介

可用来监听 window 对象的变化

## 应用: 根据窗口改变 rem

```
// 基准大小
const baseSize = 41.4;
// 设置 rem 函数
function setRem(width) {
  // 当前页面宽度相对于 414 宽的缩放比例, 可根据自己需要修改.
  const scale = width / 414;
  // 设置页面根节点字体大小
  document.documentElement.style.fontSize = baseSize * Math.min(scale, 2) + 'px';
}
window.addEventListener('resize', debounce(setRem));
复制代码
```

需要注意的是 resize 是高频事件, 可使用 `debounce` 防止多次触发

# MutationObserver

## 简介

*   `MutationObserver(callback)` 为构造函数, 可用于实例化监听 DOM 节点变化的观察器
*   出自 DOM3 规范
*   实例对象拥有三个方法:
    *   `observe(element, options)` 开始观察节点, 发生变化时触发回调 (**先放到消息队列里**)
    *   `disconnect()` 停止观察器, 直到再次调用 `observe()`
    *   `takeRecords()` 将消息队列里未处理的变化通知全部删除并返回

> 详细参考 MDN: [MutationObserver](https://developer.mozilla.org/zh-CN/docs/Web/API/MutationObserver)

## 使用

```
<! DOCTYPE html>
<html>
<head>
	<title> Mutation Observer</title>
	<style type="text/css"> .box {
			width: 100px;
			height: 100px;
			background-color: red;
		}

		.child{
			width: 50px;
			height: 50px;
			background-color: blue;
		} </style>
</head>
<body>
	<div class="box">
		<div class="child"> </div>
	</div>

	<script type="text/javascript"> const { log } = console;

		let MutationObserver = window.MutationObserver | | window.WebKitMutationObserver | | window.MozMutationObserver;

		// box 观察器的回调
		let boxCallback = (mutationList) => {
			log('box callback is called');
			for (let mutation of mutationList) {
				log(mutation);
				log(mutation.target);
				log('oldValue: ' + mutation.oldValue);
				log('');
			}
		};

		// 用于 text 观察器的回调
		let textCallback = (mutationList) => {
			log('text callback is called');
			for (let mutation of mutationList) {
				log(mutation);
				log(mutation.target);
				log('oldValue: ' + mutation.oldValue);
				log('');
			}
		};

		// 用于处理 takeRecords() 接收到的变化
		let takeRecordsCallback = (mutationList) => {
			log('take records callback is called');
			for (let mutation of mutationList) {
				log(mutation);
				log(mutation.target);
				log('oldValue: ' + mutation.oldValue);
				log('');
			}
		};

		let observer_box = new MutationObserver(boxCallback); // 此回调是异步的

		let observer_text = new MutationObserver(textCallback);

		let box = document.querySelector('.box');
		observer_box.observe(box, {
			childList: true,
			attributes: true,
			subtree: true,
			attributeFilter: ['style'],
			attributeOldValue: true // 开启后 oldValue 会记录变化, 初始为 null
		});

		box.style.width = '200px'; // MutationRecord {type: "attributes", oldValue: null, ...}
		box.style.width = '400px'; // MutationRecord {type: "attributes", oldValue: "width: 200px;", ...}

		box.appendChild(document.createElement('div')); // MutationRecord {type: "childList", oldValue: null, ...}

		let child = document.querySelector('.child');

		child.style.width = '100px'; // MutationRecord {type: "attributes", oldValue: null, ...}
		child.style.width = '200px'; // MutationRecord {type: "attributes", oldValue: "width: 100px;", ...}

		var mutations = observer_box.takeRecords(); // 将上面的 MutationRecord 都获取了, 由于 callback 都是异步的, 故上面的的 callback 未执行

		if (mutations) {
		  	takeRecordsCallback(mutations);
		}

		// log('observer_box will disconnect')
		// observer_box.disconnect();

		// 放入消息队列
		setTimeout(() => {
			log('observer_box will disconnect')
			observer_box.disconnect();
		}, 0);

		let textNode = document.createTextNode('1');
		child.appendChild(textNode); // MutationRecord {type: "childList", target: div.child, ...}

		// characterData 需要用在 text 或 comment 元素才有效果
		observer_text.observe(textNode, {
			characterData: true,
			characterDataOldValue: true // oldValue 初始为 text 的值
		});

		textNode.appendData('2'); // MutationRecord {type: "characterData", target: text, oldValue: "1"}
		textNode.appendData('3'); // MutationRecord {type: "characterData", target: text, oldValue: "12"} </script>
</body>
</html>
复制代码
```

需要注意的是:

*   `MutationObserver` 接受的 callback 执行是异步的, 元素更变一般是同步代码, 更变后 callback 会放入消息 queue, 等待当前一轮事件循环的结束才会执行
*   `takeRecords()` 不是异步的, 故可以取到当前再消息 queue 的元素变化
*   `characterData` 参数只有在元素是 `text` 或 `comment` 类型时才有效果

# 参考

*   [js 监听 div 元素的宽高变化](https://segmentfault.com/a/1190000019599439)
*   [MutationObserver-MDN 文档](https://developer.mozilla.org/zh-CN/docs/Web/API/MutationObserver)

