---
title: Docker latest 标签
date: 2020-01-17 11:00:00
tags: 'Docker'
categories:
  - ['部署', '容器化']
permalink: docker-latest-tag
photo:
---

在使用 `docker pull` 的时候, 可能会经常性没有指定标签, 这样在 `docker images` 中通常显示为 `xxx:latest`

```sh
$ docker pull hello-world
Using default tag: latest
# ...
```

可以看到, 当没有指定 tag 的时候, docker 会提示拉取 `default tag: latest`

也就是说, latest 标签在容器系统中的意思是默认标签, 而不是最新标签.

这里可能很多开发者都会有误解, 以为拉取了最新的镜像了, 以后只要更新版本了, 都会跟 docker hub 保持一致

这种想法也是半对半错的. latest 只是一个标签而已, 具体是不是最新的, 需要的是镜像维护者进行处理的

也就是说, 每次更新版本后, 都将 latest 标签重新指向到最新镜像, 这时 latest 才是最新的, 如果没有重新指向, 则 latest 是之前指向的版本.

我们区分 latest 是否最新标签, 应该只能通过镜像的 ID 进行区分而已.

<!-- more -->

举例说明

构建 `my-hello-world:v1`

```sh
$ docker build -t my-hello-world:v1 .
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM hello-world
 ---> fce289e99eb9
Step 2/2 : LABEL name="my-hello-world"
 ---> Running in 21713c20a5eb
Removing intermediate container 21713c20a5eb
 ---> 08c3e3cf00f6
Successfully built 08c3e3cf00f6
Successfully tagged my-hello-world:v1
```

```sh
$ docker images | grep hello
my-hello-world                                  v1                                 08c3e3cf00f6        13 minutes ago      1.84kB
hello-world                                     latest                             fce289e99eb9        12 months ago       1.84kB
hello-world                                     v1                                 fce289e99eb9        12 months ago       1.84kB
```

上面例子中, 通过自定义打包, 得到了 `my-hello-world:v1` 镜像, 这时我们再重新更新 Dockerfile 文件打包默认标签

构建 `my-hello-world:laster`

```sh
$ docker build -t my-hello-world .
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM hello-world
 ---> fce289e99eb9
Step 2/2 : LABEL name="my-hello-world:laster"
 ---> Running in f847353d7ee2
Removing intermediate container f847353d7ee2
 ---> eb7a7ddb43f8
Successfully built eb7a7ddb43f8
Successfully tagged my-hello-world:latest
```

```sh
$ docker images | grep hello
my-hello-world                                  latest                             eb7a7ddb43f8        3 seconds ago       1.84kB
my-hello-world                                  v1                                 08c3e3cf00f6        15 minutes ago      1.84kB
```

可以看到这时候的 latest 是最新的

构建 `my-hello-world:v2`

```sh
$ docker build -t my-hello-world:v2 .
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM hello-world
 ---> fce289e99eb9
Step 2/2 : LABEL name="my-hello-world:v2"
 ---> Running in 9e4e0ef81e6b
Removing intermediate container 9e4e0ef81e6b
 ---> 498b879cae19
Successfully built 498b879cae19
Successfully tagged my-hello-world:v2
```

```sh
$ docker images | grep hello
my-hello-world                                  v2                                 498b879cae19        29 seconds ago      1.84kB
my-hello-world                                  latest                             eb7a7ddb43f8        3 minutes ago       1.84kB
my-hello-world                                  v1                                 08c3e3cf00f6        18 minutes ago      1.84kB
```

这时候 `my-hello-world:v2` 才是最新的, 而且 latest 没有自动更新为最新的版本

手动执行 `docker tag`

```sh
$ docker tag my-hello-world:v2 my-hello-world:latest
md:test/ $ docker images | grep hello
my-hello-world                                  latest                             498b879cae19        2 minutes ago       1.84kB
my-hello-world                                  v2                                 498b879cae19        2 minutes ago       1.84kB
my-hello-world                                  v1                                 08c3e3cf00f6        20 minutes ago      1.84kB
```

这时候可以发现 latest 和 v2 的版本一致了.

注意: **需要了解 latest 只是一个普通的标签, 当没有指定时系统自动设置了, 就像个默认值**
