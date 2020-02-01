---
title: Jupyter 添加 pip 包
date: 2020-02-01 20:20:00
tags: 'Jupyter'
categories:
  - ['使用说明', '软件']
permalink: jupyter-add-package
photo:
---

# 简介

jupyter notebook 官方镜像中已经包含了 miniconda 环境了, 可以通过自定义 Dockerfile 来添加自定义包, 当然这样需要提前将所有需要的第三方包, 也可以通过 jupyter 直接安装, 其实这些都也存在不便, 如果能直接挂载宿主机的文件夹来实现加载第三方库就比较完美

<!-- more -->

# 安装

直接在 jupyter 中新建 python3 文件, 输入 `!pip install [package]` 命令即可, 当然类似的系统命令也是可以使用 `!` 开头命令, 只是使用的是自定义的用户, 所以需要注意权限

# 国内源

## 临时使用

```sh
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple some-package
```

## 设置默认

```sh
pip install pip -U
# pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pip -U
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

# 其他方法

[Python: Best way to add to sys.path relative to the current running script](https://stackoverflow.com/questions/8663076/python-best-way-to-add-to-sys-path-relative-to-the-current-running-script)

可以使用引用文件夹方式

# 引用

- [jupyter/docker-stacks](https://github.com/jupyter/docker-stacks)
