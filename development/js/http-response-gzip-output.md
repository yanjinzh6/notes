---
title: HTTP 请求响应 gzip 头处理
date: 2020-03-25 11:00:00
tags: 'JavaScript'
categories:
  - ['开发', 'JavaScript']
permalink: http-response-gzip-output
---

处理 HTTP 响应通常如下

```js
res.setEncoding('utf8')
let rawData = ''
res.on('data', chunk => {
  rawData += chunk
}).end()
```

有时候会出现 `rawData` 中的数据是乱码的, 这时候检查一下 `res.headers` 后发现 `Content-Encoding: gzip`, 这是服务器响应使用了 `gzip` 压缩, 所以这里需要简单判断一下

```js
let output
if (res.headers['content-encoding'] == 'gzip') {
  var gzip = zlib.createGunzip()
  res.pipe(gzip)
  output = gzip
} else {
  output = res
}
output.setEncoding('utf8')
let rawData = ''
output.on('data', chunk => {
  rawData += chunk
}).end()
```
