---
title: js-mvvm
date: 2021-01-31 19:00:00
tags: 'JavaScript'
categories:
  - ['开发', 'JavaScript']
permalink: js-mvvm
mathjax: false
---

https://segmentfault.com/a/1190000015807808

# [实现一个 mvvm](/a/1190000015807808)

[! [](https://avatar-static.segmentfault.com/277/678/2776783912-59783adb36895_big64)**keller35**](/u/keller35) 发布于 2018-07-29

最近在团队内做了一次 vue 原理分享, 现场手写了一个乞丐版 mvvm, 这里记录一下这个 mvvm 实现的过程.

源码: [https://github.com/keller35/mvvm](https://github.com/keller35/mvvm)

这个 mvvm 是基于发布订阅模式实现 (也是 vue 本身的实现原理), 最终达到的效果如下:

使用方式也跟 vue 一样:

```
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title> mvvm</title>
</head>
<body>
    <div id="app">
        <input type="text" v-model="text">
        {{ text }}
        <button @click="reset" style="display: block;"> 重置 </button>
    </div>
    <script src="./index.js"> </script>
    <script>
        var vm = new Mvvm({
            el: 'app',
            data: {
                text: 'hello world'
            },
            methods: {
                reset() {
                    this.text = '';
                },
            },
        });
    </script>
</body>
</html>
```

实现很简单:

```
class Mvvm {
    constructor(options) {
        const { el, data, methods } = options;
        this.methods = methods;
        this.target = null;
        // 初始化 dispatcher
        this.observe(this, data);
        // 初始化 watcher
        this.compile(document.getElementById(el));
    }

    observe(root, data) {
        for (const key in data) {
            this.defineReactive(root, key, data[key]);
        }
    }

    defineReactive(root, key, value) {
        if (typeof value == 'object') {
            return this.observe(value, value);
        }
        const dep = new Dispatcher();
        Object.defineProperty(root, key, {
            set(newValue) {
                if (value == newValue) return;
                value = newValue;
                // 发布
                dep.notify(newValue);
            },
            get() {
                // 订阅
                dep.add(this.target);
                return value;
            }
        });
    }

    compile(dom) {
        const nodes = dom.childNodes;
        for (const node of nodes) {
            // 元素节点
            if (node.nodeType == 1) {
                const attrs = node.attributes;
                for (const attr of attrs) {
                    if (attr.name == 'v-model') {
                        const name = attr.value;
                        node.addEventListener('input', e => {
                            this[name] = e.target.value;
                        });
                        this.target = new Watcher(node, 'input');
                        this[name];
                    }
                    if (attr.name == '@click') {
                        const name = attr.value;
                        node.addEventListener('click', this.methods[name].bind(this));
                    }
                }
            }
            // text 节点
            if (node.nodeType == 3) {
                const reg = /\{\{(.*)\}\}/;
                const match = node.nodeValue.match(reg);
                if (match) {
                    const name = match[1].trim();
                    this.target = new Watcher(node, 'text');
                    this[name];
                }
            }
        }
    }
}

class Dispatcher {
    constructor() {
        this.watchers = [];
    }
    add(watcher) {
        this.watchers.push(watcher);
    }
    notify(value) {
        this.watchers.forEach(watcher => watcher.update(value));
    }
}

class Watcher {
    constructor(node, type) {
        this.node = node;
        this.type = type;
    }
    update(value) {
        if (this.type == 'input') {
            this.node.value = value;
        }
        if (this.type == 'text') {
            this.node.nodeValue = value;
        }
    }
}
```

原理:

1.  最根本的原理很简单, 无非是基于发布订阅的消息通知模式, 消息发出方来自 mvvm 中 modal 层的变法, 而订阅方来自 view 层.
2.  modal 层的变化, 是通过对 data 设置 setter 来实现响应式, 只要数据发生变化, 通知所有订阅者.
3.  view 层的订阅, 则是在 compile 阶段, compile 会对所有数据依赖进行收集, 然后在 getter 中注册监听.

[mvvm](/t/mvvm) [react.js](/t/react.js) [defineproperty](/t/defineproperty) [vue.js](/t/vue.js)

阅读 3.3k 更新于 2018-07-30

赞 45 收藏 27

[分享](#)

本作品系原创, [采用 <署名 - 非商业性使用 - 禁止演绎 4.0 国际> 许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0/)

* * *

! [](https://image-static.segmentfault.com/383/303/3833030929-5ffd15fc110ab)

[

##### 有赞美业前端团队

](/blog/fedbj)

关注专栏
