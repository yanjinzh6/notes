---
title: Hexo Nunjucks Error
date: 2020-08-01 07:00:00
tags: 'Hexo'
categories:
  - ['使用说明', '软件']
permalink: hexo-nunjucks-error
---

## 简介

今天看了下 Hexo 的日志发现最近都没部署成功, 看来这个自动化日程还很漫长

错误如下

```sh
FATAL Something's wrong. Maybe you can find the solution here: https://hexo.io/docs/troubleshooting.html
Nunjucks Error:  [Line 4, Column 27] expected variable end
    =====               Context Dump               =====
    === (line number probably different from source) ===
  1 | <h2 id="简介"><a href="#简介" class="headerlink" title="简介"></a>简介</h2><p>单调栈是一种特殊的栈, 栈内的元素都保存一个单调性, 递增或递减, 利用这个特性可以解决一些特殊的问题</p>
  2 | <h2 id="问题"><a href="#问题" class="headerlink" title="问题"></a>问题</h2><p>给定一个数组 arr, 找出每一个元素左边和右边离当前位置最近且比当前值小的位置, 返回所有位置相关信息</p>
  3 | <p>例如 <code>arr = {3, 4, 1, 5, 6, 2, 7}</code></p>
  4 | <p>返回二维数组 <code>res = {{-1, 2}, {0, 2}, {-1, -1}, {2, 5}, {3, 5}, {2, -1}, {5, -1}}</code>, -1 表示不存在</p>
  5 | <!-- more -->
  6 |
  7 | <h2 id="分析"><a href="#分析" class="headerlink" title="分析"></a>分析</h2><p>通过对每个位置向左右方向分别遍历一下可以解决问题, 这样时间复杂度就是 $O(N^2)$</p>
  8 | <p>使用单调栈结构可以使得时间复杂度为 $O(N)$, 保证栈里的值都是递增的, 分析情况如下</p>
  9 | <ul>
    =====             Context Dump Ends            =====
    at formatNunjucksError (/home/jenkins/agent/workspace/hexo-notes/hexo-notes/node_modules/hexo/lib/extend/tag.js:99:13)
    at Promise.fromCallback.catch.err (/home/jenkins/agent/workspace/hexo-notes/hexo-notes/node_modules/hexo/lib/extend/tag.js:121:34)
    at tryCatcher (/home/jenkins/agent/workspace/hexo-notes/hexo-notes/node_modules/bluebird/js/release/util.js:16:23)
    at Promise._settlePromiseFromHandler (/home/jenkins/agent/workspace/hexo-notes/hexo-notes/node_modules/bluebird/js/release/promise.js:547:31)
    at Promise._settlePromise (/home/jenkins/agent/workspace/hexo-notes/hexo-notes/node_modules/bluebird/js/release/promise.js:604:18)
    at Promise._settlePromise0 (/home/jenkins/agent/workspace/hexo-notes/hexo-notes/node_modules/bluebird/js/release/promise.js:649:10)
    at Promise._settlePromises (/home/jenkins/agent/workspace/hexo-notes/hexo-notes/node_modules/bluebird/js/release/promise.js:725:18)
    at _drainQueueStep (/home/jenkins/agent/workspace/hexo-notes/hexo-notes/node_modules/bluebird/js/release/async.js:93:12)
    at _drainQueue (/home/jenkins/agent/workspace/hexo-notes/hexo-notes/node_modules/bluebird/js/release/async.js:86:9)
    at Async._drainQueues (/home/jenkins/agent/workspace/hexo-notes/hexo-notes/node_modules/bluebird/js/release/async.js:102:5)
    at Immediate.Async.drainQueues [as _onImmediate] (/home/jenkins/agent/workspace/hexo-notes/hexo-notes/node_modules/bluebird/js/release/async.js:15:14)
    at runCallback (timers.js:705:18)
    at tryOnImmediate (timers.js:676:5)
    at processImmediate (timers.js:658:5)
error Command failed with exit code 2.
info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
```

初看一下没发现什么问题, 但是这个错误第一次碰见, 看描述只知道 `format` 就是格式有问题

查了一下发现是因为双大括号的问题, 转义成 \{\{\}\}
