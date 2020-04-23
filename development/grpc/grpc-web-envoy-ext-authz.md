---
title: Grpc-web 使用 Envoy 外部认证简单例子
date: 2020-03-04 17:00:00
tags: 'Grpc'
categories:
  - ['开发', 'Grpc']
permalink: grpc-web-envoy-ext-authz
---

## 简介

使用 grpc-web 的情况下, 前端代码变成了 `grpcClient.doGrpcFunc`, 与之前的 fetch 请求的区别就是 grpc-web 使用 http2 metadata 的方式, 然后 envoy 又会帮忙设置成 http headers, 所以这里就简单实现一个设置认证并且添加一个 `http_filters` 来处理认证状态的例子

<!-- more -->

## 配置

这里主要的配置使用了 [envoy front-proxy 项目](https://github.com/envoyproxy/envoy/tree/master/examples/front-proxy)

### envoy 配置

首先配置了 `envoy.http_connection_manager` filter, 通过路由, 将所有的请求都转发到 `cluster: echo_service`, 当需要自定义 http header 的时候需要在 `cors` 配置上, 然后通过自定义一个 `http_filters` 来处理认证

```yaml
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address:
      address: 0.0.0.0
      port_value: 9901

static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address:
        address: 0.0.0.0
        port_value: 8080
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        config:
          codec_type: auto
          stat_prefix: ingress_http
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match: { prefix: "/" }
                route:
                  cluster: echo_service
                  max_grpc_timeout: 0s
              cors:
                allow_origin_string_match:
                - prefix: "*"
                allow_methods: GET, PUT, DELETE, POST, OPTIONS
                allow_headers: keep-alive,user-agent,cache-control,content-type,content-transfer-encoding,custom-header-1,x-accept-content-transfer-encoding,x-accept-response-streaming,x-user-agent,x-grpc-web,grpc-timeout
                max_age: "1728000"
                expose_headers: custom-header-1,grpc-status,grpc-message
          http_filters:
          - name: envoy.ext_authz
            config:
              http_service:
                server_uri:
                  uri: auth-service:6060
                  cluster: ext-authz
                  timeout: 0.25s
                authorization_request:
                  allow_headers:
                    - x-grpc-web
          - name: envoy.grpc_web
          - name: envoy.cors
          - name: envoy.router
  clusters:
  - name: echo_service
    connect_timeout: 0.25s
    type: logical_dns
    http2_protocol_options: {}
    lb_policy: round_robin
    hosts:
    - socket_address:
        address: grpc-service
        port_value: 9090
  - name: ext-authz
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    hosts:
    - socket_address:
        address: auth-service
        port_value: 6060
```

### helloworld.proto

```proto
syntax = "proto3";

package helloworld;

service Greeter {
  // unary call
  rpc SayHello(HelloRequest) returns (HelloReply);
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

### 客户端

通过调用 `sayHello` 并且设置 `{ Authorization: 'test' }` 模拟接口认证

```js
const { HelloRequest } = require('./helloworld_pb.js');
const { GreeterClient } = require('./helloworld_grpc_web_pb.js');

var client = new GreeterClient('https://xxx.club',
  null, null);

// simple unary call
var request = new HelloRequest();
request.setName('World');

client.sayHello(request, { Authorization: 'test' }, (err, response) => {
  console.log(response.getMessage());
});
```

### 认证服务端

```js
const express = require('express')
const app = express()
const port = 6060

app.use((req, res, next) => {
  console.log(req.headers)

  if(req.headers['Authorization']) {
    // 鉴权逻辑
    res.status(200)
    res.end()
  } else {
    res.status(401)
    res.end('Unauthorized')
  }
})

app.listen(port, () => console.log(`Example app listening on port ${port}!`))
```

### 应答服务端

```js
var PROTO_PATH = __dirname + '/helloworld.proto';

var grpc = require('grpc');
var protoLoader = require('@grpc/proto-loader');
var packageDefinition = protoLoader.loadSync(
  PROTO_PATH,
  {
    keepCase: true,
    longs: String,
    enums: String,
    defaults: true,
    oneofs: true
  });
var protoDescriptor = grpc.loadPackageDefinition(packageDefinition);
var helloworld = protoDescriptor.helloworld;

/**
 * @param {!Object} call
 * @param {function():?} callback
 */
function doSayHello (call, callback) {
  console.log('doSayHello', call)
  callback(null, { message: 'Hello! ' + call.request.name });
}

/**
 * @return {!Object} gRPC server
 */
function getServer () {
  var server = new grpc.Server();
  server.addService(helloworld.Greeter.service, {
    sayHello: doSayHello,
    sayRepeatHello: doSayRepeatHello,
    sayHelloAfterDelay: doSayHelloAfterDelay
  });
  return server;
}

if (require.main === module) {
  var server = getServer();
  server.bind('0.0.0.0:9090', grpc.ServerCredentials.createInsecure());
  server.start();
}

exports.getServer = getServer;
```

## 测试

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <title>gRPC-Web Example</title>
  <script src="./dist/main.js"></script>
</head>

<body>
  <p>Open up the developer console and see the logs for the output.</p>
</body>

</html>
```

通过简单的 http 服务器来访问页面, 可以看到 envoy 已经将 `{ Authorization: 'test' }` 转换为请求头去到认证服务器那边了, grpc 服务中可以通过 `call.metadata.Metadata._internal_repr.authorization` 来获取值

```js
{ host: 'xxx.club',
  'content-length': '0',
  authorization: 'test',
  'x-envoy-internal': 'true',
  'x-forwarded-for': '10.1.1.123',
  'x-envoy-expected-rq-timeout-ms': '250' }
```

```js
ServerUnaryCall {
  _events: [Object: null prototype] { error: [Function] },
  _eventsCount: 1,
  _maxListeners: undefined,
  call: Call {},
  cancelled: false,
  metadata:
   Metadata {
     _internal_repr:
      { 'x-request-id': [Array],
        'x-real-ip': [Array],
        'x-forwarded-for': [Array],
        'x-forwarded-host': [Array],
        'x-forwarded-port': [Array],
        'x-forwarded-proto': [Array],
        'x-original-uri': [Array],
        'x-scheme': [Array],
        pragma: [Array],
        'cache-control': [Array],
        'x-user-agent': [Array],
        authorization: [Array],
        accept: [Array],
        'user-agent': [Array],
        'x-grpc-web': [Array],
        'sec-fetch-dest': [Array],
        origin: [Array],
        'sec-fetch-site': [Array],
        'sec-fetch-mode': [Array],
        referer: [Array],
        'accept-language': [Array] },
     flags: 0 },
  request: { name: 'World' } }
```

## 小结

这只是一个简单的例子, 认证和授权一般都有很多成熟的服务, 后面需要继续探讨怎么进行无耦合的使用各种成熟的服务
