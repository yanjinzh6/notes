---
title: Maven Snapshot 快照部署
date: 2020-06-26 09:00:00
tags: 'Maven'
categories:
  - ['开发', 'Java', '工具']
permalink: maven-snapshot-deploy
---

## 简介

Snapshot 快照仓库主要用于保存开发过程中的不稳定的发布, 在发布的过程中, Maven 会自动为构件打上时间戳, 例如: `0.0.1-20191214.101010-15` 就表示 2019 年 12 月 14 日 10 点 10 分 10 秒的第 15 次快照, 有了该时间戳, Maven 就能随时找到仓库中该构件 0.0.1-SNAPSHOT 版本最新的文件

默认情况下, Maven 每天检查一次更新 (由仓库配置的 updatePolicy 控制) , 用户也可以使用命令行 -U 参数强制让 Maven 检查更新, 如：`mvn clean install -U`

<!-- more -->

## 配置

修改 `setting.xml`

```xml
<snapshots>
  <enabled>true</enabled>
  <updatePolicy>always</updatePolicy>
  <checksumPolicy>warn</checksumPolicy>
</snapshots>
```

- `enabled`: 默认为 `true`
- `updatePolicy`: 检查 Snapshot 更新频率, 可以配置为 always (实时更新), daily (每天更新), interval:xxx (隔xxx分钟更新一次), never (从不更新), 默认为 daily
- `checksumPolicy`: 校验策略

## 问题

碰到本地测试一直使用远程的版本, 本地 `mvn clean install` 后的版本无效, 这时可以使用 `mvn clean install -nsu` 命令来忽略 Snapshot 快照的更新
