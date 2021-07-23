---
title: js-fetch-formdata
date: 2021-01-31 17:00:00
tags: 'JavaScript'
categories:
  - ['开发', 'JavaScript']
permalink: js-fetch-formdata
mathjax: false
---

https://www.jianshu.com/p/5a33458e4a84

# react 之 fetch 登录请求 formData 提交参数

买了一包大鸡排也没心思吃, 解决完这个问题后, 狠狠地咬了一口大鸡排!

之前每次用 fetch 都是获取数据, 没有提交过参数, 但是现在做的登录功能, 是要将表单中输入的用户名和密码拿到后提交给后台服务器端, 并得到返回数据来判断用户名和密码是否正确.

1, react 表单

按照以往 js 获取表单数据的方法, 当然是获取到该 input 的 ID, 然后根据 id 定位后获取到其 value 值, 但是很可惜, react 不能这样做.

react 对表单元素做了优化处理, 对其进行抽象处理, 使其使用方式更统一和规范.

约束性组件和非约束性组件

约束性组件, 简单来说就是 React 管理了它的 value, 而非约束性组件的 value 则是由原生的 DOM 管理.
所以在写法上区别很大:

非约束性组件写法:

图片.png



defaultValue 中就是原生 DOM 中的 value 属性, 非约束性组件中的 value 值就是用户输入的内容, React 完全不管理输入的过程.

约束性组件写法:

图片.png



约束性组件中的 value 值不再是一个写死的值, 而是写在 state 中, 由 this.handleChange 负责管理.
在 handleChange 中可以重新渲染 state 的值, 同时也可以对输入的内容进行校验.

2, fetch 数据请求

当我们拿到用户名和密码时, 需要将数据提交给服务器端并得到返回值.fetch 传参数必须要是 formData, 就是这个折磨了我好久.

```
let url = ".................................";// 接口地址

let formData = new FormData();
formData.append('c','login');
formData.append('username', this.state.userName);
formData.append('password', this.state.passWord);
formData.append('client', 'android');

fetch(url, {
    method: 'post',
    body: formData,
}).then(function (res) {
    return res.json();
}).then(function (json) {
    if (json.code == "200") {
        console.log("232323233----- 正确 ")
    } else if (json.code == "400") {
        console.log("2323232323------ 错了~")
    }
})
```

终于完成简易的登录功能了.
