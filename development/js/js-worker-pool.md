---
title: Node JS 多线程线程池
date: 2021-01-23 11:00:00
tags: 'JavaScript'
categories:
  - ['开发', 'JavaScript']
permalink: js-worker-pool
mathjax: false
---

## 简介

JavaScript 是一个具有 `单线程` 特性的 `简单` 语言, 适合完成一些 `简单的任务`, 比如验证表单, 当访客离开页面时改变页面标题和 favicon, 或者渲染整个页面, 双向绑定与视图更新.

支持多线程的后端语言常常具有专门的机制在线程之间同步数据, Node.js 在 `12` 版本后正式添加多线程支持.

<!-- more -->

## Node.js 的单线程

通过终端输入 `node` 然后回车, 进入 Node.js 的 REPL 模式. 接着打开 `任务管理器` 或者 `活动监视器` 或者 `top` (取决于使用的系统), 会发现 `node` 的进程数可不是 1.

Node.js 启动后的进程数不是 1 甚至远大于 1, 不是因为进程池, 而是因为 V8 实例是多线程的:

* 主线程: 获取, 编译, 执行代码
* 编译线程: 当主线程在执行时, 编译线程可以优化代码
* Profiler 线程: 记录方法耗时的线程
* 其它线程: 比如支持并行 GC 的多线程

## Node.js: Event Loop 和 Worker Pool

众所周知, JavaScript 使用 Event Loop 处理主线程. 对于 Node.js, 耗时操作如 `crypto` 和 `fs`(前者 CPU 密集, 后者 I/O 密集) 在 Worker Pool 中进行.Worker Pool 是 [libuv](https://github.com/libuv/libuv) 实现的, 其执行模型是单独创建一个进程, 在这个进程中同步执行任务, 然后将结果返回到 Event Loop 中, Event Loop 可以通过 callback 获取并使用结果. 基于这个执行模式可以写出这样的代码:

```js
const { readFile } = require('fs');

fs.readFile('path/to/something', (err, content) => {
  if (err) console.error(err);
  console.log(content);
});
```

以上是一个非阻塞代码的示例, 不必同步等待某事的发生. 只需告诉 Worker Pool 去读取文件, 并用结果去调用提供的函数即可. 由于 Worker Pool 也有自己的线程, 因此 Event Loop 不会被阻塞.

在需要对数据进行复杂计算的时候 (比如 AI, 机器学习, 大数据, 或者 [OI-Wiki](https://oi-wiki.org/) 的 MathJax 公式渲染), 主线程 (同时也是唯一的线程) 会被阻塞以至于不能执行其他任务, 因此 Node.js 一般不适合用于执行耗时函数.

> 如果想要了解更多, 可以看看 Node.js 官网上的这篇文章: {[Don't Block the Event Loop (or the Worker Pool)](https://nodejs.org/en/docs/guides/dont-block-the-event-loop/)}

## Worker Threads

从 Node.js 10.5.0 开始, Node.js 提出了真正意义的多线程支持.

```js
const { Worker } = require('worker_threads');

const worker = new Worker(path, { data });
worker.on('message', msg => { /* ... */ });
worker.on('error', err => { /* ... */ });
worker.on('exit', exitcode => { /* ... */ });
worker.on('online', () => { /* ... */ });
```

通过创建一个 `Worker` 类的实例, 实例的第一个参数 `path` 是 Workers 的路径, 第二个参数 `data` 是这个 Worker 在启动时可以读到的数据. Worker 之间的通信基于事件, 因此为事件设置了 Listener 和回调. 使用回调是因为 Worker 可以发送不止一个 `message`. 当然, 如果 Worker 只会发送一个 message, 那么用 `Promise.resolve` 也是可以的.

Worker 支持监听四种事件:

* message: Worker 将数据发送到主线程时就会触发 `message` 事件
* error: Worker 中任何没有被 catch 的 Error 都会触发这一事件, 同时 Worker 会被中止
* exit: Worker 退出时触发. 在 Worker 中 `process.exit()` 得到的 exitCode 是 0,`worker.terminate()` 得到的 exitCode 是 1
* online: 当 Worker 解析完毕开始执行时触发

## 线程间通信

如果需要将数据发送到另一个线程, 可以使用 `postMessage` 方法:

```js
port.postMessage(data[, transferList]);
```

第一个参数是一个被复制到另一个线程的对象. 一般的, 这部分内存会被主线程和 Worker 共用. 参考 [The structured clone algorithm](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm)

一般最常见的用法是将 Worker 的数据发送到主线程:

```js
// worker.js
const { parentPort } = require('worker_threads');
parentPort.postMessage(data);
```

同样, 通过 `parentPort` 也可以监听从主线程发过来的信息:

```js
// worker.js
const { parentPort } = require('worker_threads');
parentPort.on('message', data => { /* ... */ });
```

如果要在 Worker 中使用主线程中 `new Worker` 时第二个参数传入的数据, 应该使用 `workerData` 属性:

```js
// worker.js
const { workerData } = require('worker_threads');
```

> 关于 `worker_threads` 的其它属性, 如何修改线程间的共用内存, 以及如何在线程之间使用 MessageChanel 通信, 请参考 [Node.js Worker Threads 的相关文档](https://nodejs.org/dist/latest/docs/api/worker_threads.html).

## 实现一个 Worker Pool

需要注意的是, 创建, 执行, 销毁一个 Worker 的开销是很大的. 频繁创建 Worker 消耗的 CPU 算力很快就会抵消多线程带来的好处, 越来越多的监听器甚至可能会导致 OOM. 所以为每一个任务创建一个 Workers 是很不现实的, 在实践中应该实现一个 Worker Pool, 在初始化时将会创建 **有限数量** 的 Worker 并加载单一的 `worker.js`, 主线程通过线程间通信的方法将要执行的任务传给 Worker, 而 Worker 也通过线程间通信的方法将任务的结果回传给主线程, 当所有任务完成后, 这些 Worker 将会被统一销毁.

一个简单的线程池逻辑

* 创建指定数量的 Workers
* 创建一个任务队列
* 创建一个 Workers 索引, 以及用于追踪他们是否处于激活状态的索引

```js
// 获取当前设备的 CPU 线程数目, 作为 numberOfThreads 的默认值.
const { length: cpusLength } = require('os').cpus();

class WorkerPool = {
  constructor(workerPath, numberOfThreads = cpusLength) {
    if (numberOfThreads < 1) {
      throw new Error('Number of threads should be greater or equal than 1!');
    }

    this.workerPath = workerPath;
    this.numberOfThreads = numberOfThreads;

    // 任务队列
    this._queue = [];
    // Worker 索引
    this._workersById = {};
    // Worker 激活状态索引
    this._activeWorkersById = {};

    // 创建 Workers
    for (let i = 0; i < this.numberOfThreads; i++) {
      const worker = new Worker(workerPath);

      this._workersById[i] = worker;
      // 将这些 Worker 设置为未激活状态
      this._activeWorkersById[i] = false;
    }
  }
}
```

在添加 Worker 执行之前, 还需要做两件事情, 检查是否有空闲的 Worker 用于执行任务, 以及 Worker 本身的执行.

首先是检查空闲的 Worker, 这个并不难:

```js
getInactiveWorkerId() {
  for (let i = 0; i < this.numberOfThreads; i++) {
    if (! this._activeWorkersById[i]) return i;
  }
  return -1;
}
```

接下来是调用 Worker 执行, 目的是在指定的 Worker 里执行指定的任务:

```js
runWorker(workerId, taskObj) {
  const worker = this._workersById[workerId];

  // 当任务执行完毕后执行
  const doAfterTaskIsFinished = () => {
    // 去除所有的 Listener, 不然一次次添加不同的 Listener 会 OOM 的
    worker.removeAllListeners('message');
    worker.removeAllListeners('error');
    // 将这个 Worker 设为未激活状态
    this._activeWorkersById[workerId] = false;

    if (this._queue.length) {
      // 任务队列非空, 使用该 Worker 执行任务队列中第一个任务
      this.runWorker(workerId, this._queue.shift());
    }
  };

  // 将这个 Worker 设置为激活状态
  this._activeWorkersById[workerId] = true;
  // 设置两个回调, 用于 Worker 的监听器
  const messageCallback = result => {
    taskObj.cb(null, result);
    doAfterTaskIsFinished();
  };
  const errorCallback = error => {
    taskObj.cb(error);
    doAfterTaskIsFinished();
  };

  // 为 Worker 添加 'message' 和 'error' 两个 Listener
  worker.once('message', messageCallback);
  worker.once('error', errorCallback);
  // 将数据传给 Worker 供其获取和执行
  worker.postMessage(taskObj.data);
}
```

实现外部调用的 `run()` 方法.

```js
run(data) {
  // Promise 是个好东西
  return new Promise((resolve, reject) => {
    // 调用 getInactiveWorkerId() 获取一个空闲的 Worker
    const availableWorkerId = this.getInactiveWorkerId();

    const taskObj = {
      data,
      cb: (error, result) => {
        // 虽然 Workers 需要使用 Listener 和 Callback, 但这不能阻止使用 Promise, 对吧?
        // 不, 你不能 util.promisify(taskObj) . 人不能, 至少不应该.
        if (error) reject(error);
        return resolve(result);
      }
    };

    if (availableWorkerId === -1) {
      // 当前没有空闲的 Workers 了, 把任务丢进队列里, 这样一旦有 Workers 空闲时就会开始执行.
      this._queue.push(taskObj);
      return null;
    }

    // 有一个空闲的 Worker, 用它执行任务
    this.runWorker(availableWorkerId, taskObj);
  })
}
```

实现 `destroy` 方法, 调用 `worker.terminate()` 销毁所有创建的 Worker:

```js
destroy(force = false) {
    for (let i = 0; i < this.numberOfThreads; i++) {
      if (this._activeWorkersById[i] && ! force) {
        // 通常情况下, 不应该在还有 Worker 在执行的时候就销毁它, 这一定是什么地方出了问题, 所以还是抛个 Error 比较好
        // 不过保留一个 force 参数, 总有人用得到的
        throw new Error(`The worker ${i} is still runing!`);
      }

      // 销毁这个 Worker
      this._workersById[i].terminate();
    }
  }
}
```

## 使用线程池拉取网页数据并保存

通过简单的提交一个 URL 并由后台进行处理后返回, 如果一直在主线程中等待会导致接口响应时间变慢, 这时可以使用 `worker_threads` 来处理结果并进行保存

```js
// 接收主线程获取到的 url
parentPort.on('message', async endpoint => {
  logger.debug(`开始查询 endpoint: ${endpoint}`)
  const data = await query({
    endpoint
  })
  if (data.status !== 200 || !data.data) {
    logger.error('查询 endpoint 出现错误')
    logger.error(data)
    return
  }
  // 处理数据
  // 处理过程省略 ...
  const res = await hmsetAsync(getKey(endpoint), data.data)
  logger.debug(`hmsetAsync 结果: ${res}`)
  // 返回数据到主线程
  parentPort.postMessage('OK')
})
```

再通过 `websocket` 和 `redis` 的订阅的方式进行通知即可
