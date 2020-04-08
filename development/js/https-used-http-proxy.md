---
title: HTTPS 请求使用 HTTP 代理
date: 2020-03-25 10:00:00
tags: 'JavaScript'
categories:
  - ['开发', 'JavaScript']
permalink: https-used-http-proxy
---

Nodejs HTTP 包使用代理并不是很方便, 主要记录下一种平时试验用的方法, 原理就是通过 HTTP 连接代理后, HTTPS 使用上面代理的隧道进行通信

```js
const req = http.get({
  host: '127.0.0.1',
  port: 8081,
  method: 'CONNECT',
  timeout: 10000,
  path: 'baidu.com:443'
}).on('connect', (response, socket) => {
  log.log(response.headers, response.statusCode, response.statusMessage)
  if (response.statusCode === 200) {
    https.request({
      method: 'GET',
      host: 'baidu.com',
      socket,
      agent: false,
      timeout: 10000,
      headers: {
        'User-Agent': 'Mozilla/5.0'
      },
      rejectUnauthorized: false
    }, (res) => {
      log.log(res.headers, res.statusCode, res.statusMessage)
      let chunks = []
      res.on('data', chunk => chunks.push(chunk))
      res.on('end', () => {
        log.log('DONE', Buffer.concat(chunks).toString('utf8'))
        resolve(Buffer.concat(chunks).toString('utf8'))
      })
    }).on('error', err => {
      log.error(err)
    }).end()
  }
}).on('timeout', error => {
  log.error(error)
  reject(error)
}).on('error', error => {
  log.error(error)
  reject(error)
}).end()
```

```sh
client:log {} 200 Connection established +0ms
client:log { server: 'bfe/1.0.8.18',
client:log   date: 'Wed, 25 Mar 2020 02:17:21 GMT',
client:log   'content-type': 'text/html',
client:log   'content-length': '161',
client:log   connection: 'close',
client:log   location: 'http://www.baidu.com/' } 302 Moved Temporarily +238ms
client:log DONE <html>
<head><title>302 Found</title></head>
<body bgcolor="white">
<center><h1>302 Found</h1></center>
<hr><center>bfe/1.0.8.18</center>
</body>
</html>
 +6ms
```

注意: `rejectUnauthorized: false` 是不规范的行为, 正常是需要验证 CA 证书的, 可以使用 `ssl-root-cas/latest` 模块

最简单的还是使用现成的模块

```js
const HttpsProxyAgent = require('https-proxy-agent')
const options = url.parse(endpoint)
options.timeout = 10000
options.headers = {
  'User-Agent': 'Mozilla/5.0'
}
options.agent = new HttpsProxyAgent({
  host: '172.17.0.1',
  port: 8081,
  headers: {
    'User-Agent': 'Mozilla/5.0'
  },
  timeout: 10000
})
https.get(options, (res) => {}).end()
```