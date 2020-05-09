---
title: 新建 hexo Jenkins 任务
date: 2020-01-02 18:00:00
tags: 'Jenkins'
categories:
  - ['部署', 'CI']
permalink: hexo-blog-git-jenkins
photo:
---

## 起因

自己整 VPS 搭的一套 CI 还是需要用的，所以有任何项目还是得想方法用用。虽然每次维护都比较麻烦，毕竟国内的 VPS 要访问 GitHub 还是挺慢的，访问 Jenkins 服务器也超级慢，升级插件需要漫长的等待，就更不用说 docker hub 了，只能加多些跳板。

本来挺方便的一个 Jenkins，被我开启了 slave 模式，美其名曰不占用资源，按需启用，但是中间的配置过程也是一种 🦐 折腾，这也是最近几天都是手动去触发构建的真实原因。

之前 Jenkins slave 模式使用了一个封装好 docker 客户端，kubectl 等工具的镜像，不过镜像太旧了，启动出错，继续封装也太大了，所以重新打包了一个包含 nodejs 和 git 等常用的镜像。问题是还得去摸索清楚 Jenkins k8s 插件的镜像使用规则，是否对于每个 `node('slave-agent')` 都可以用自定义的模版来处理，这样就可以不同类型项目使用不同的 slave 镜像了 ：heart_eyes：。

<!-- more -->

## 尝试打包镜像

最简单的当然就是使用官方的镜像作为来源咯 :joy:，简单实惠人人喜欢，特别是都有基于 `alpine` 的镜像那就是非常容易下载了。

```dockerfile
FROM jenkins/jnlp-slave:alpine
ENV NODE_VERSION 10.18.0

USER root
RUN apk add --no-cache \
        libstdc++ \
    && apk add --no-cache --virtual .build-deps \
        curl \
    && ARCH= && alpineArch="$(apk --print-arch)" \
      && case "${alpineArch##*-}" in \
        x86_64) \
          ARCH='x64' \
          CHECKSUM="043f9e1c412a391f42a9667373b851590a9a77c08cf6fde6828a3cdb3fb8f316" \
          ;; \
        *) ;; \
      esac \
  && if [ -n "${CHECKSUM}" ]; then \
    set -eu; \
    curl -fsSLO --compressed "https://unofficial-builds.nodejs.org/download/release/v$NODE_VERSION/node-v$NODE_VERSION-linux-$ARCH-musl.tar.xz"; \
    echo "$CHECKSUM  node-v$NODE_VERSION-linux-$ARCH-musl.tar.xz" | sha256sum -c - \
      && tar -xJf "node-v$NODE_VERSION-linux-$ARCH-musl.tar.xz" -C /usr/local --strip-components=1 --no-same-owner \
      && ln -s /usr/local/bin/node /usr/local/bin/nodejs; \
  else \
    echo "Building from source" \
    # backup build
    && apk add --no-cache --virtual .build-deps-full \
        binutils-gold \
        g++ \
        gcc \
        gnupg \
        libgcc \
        linux-headers \
        make \
        python \
    # gpg keys listed at https://github.com/nodejs/node#release-keys
    && for key in \
      94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
      FD3A5288F042B6850C66B31F09FE44734EB7990E \
      71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
      DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
      C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
      B9AE9905FFD7803F25714661B63B535A4C206CA9 \
      77984A986EBC2AA786BC0F66B01FBB92821C587A \
      8FCCA13FEF1D0C2E91008E09770F7A9A5AE15600 \
      4ED778F539E3634C779C87C6D7062848A1AB005C \
      A48C2BEE680E841632CD4E44F07496B3EB3C1762 \
      B9E2F5981AA6E0CD28160D9FF13993A75599653C \
    ; do \
      gpg --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" || \
      gpg --batch --keyserver hkp://ipv4.pool.sks-keyservers.net --recv-keys "$key" || \
      gpg --batch --keyserver hkp://pgp.mit.edu:80 --recv-keys "$key" ; \
    done \
    && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION.tar.xz" \
    && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
    && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
    && grep " node-v$NODE_VERSION.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
    && tar -xf "node-v$NODE_VERSION.tar.xz" \
    && cd "node-v$NODE_VERSION" \
    && ./configure \
    && make -j$(getconf _NPROCESSORS_ONLN) V= \
    && make install \
    && apk del .build-deps-full \
    && cd .. \
    && rm -Rf "node-v$NODE_VERSION" \
    && rm "node-v$NODE_VERSION.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt; \
  fi \
  && rm -f "node-v$NODE_VERSION-linux-$ARCH-musl.tar.xz" \
  && apk del .build-deps

ENV YARN_VERSION 1.21.1

RUN apk add --no-cache --virtual .build-deps-yarn curl gnupg tar \
  && for key in \
    6A010C5166006599AA17F08146C2130DFD2497F5 \
  ; do \
    gpg --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" || \
    gpg --batch --keyserver hkp://ipv4.pool.sks-keyservers.net --recv-keys "$key" || \
    gpg --batch --keyserver hkp://pgp.mit.edu:80 --recv-keys "$key" ; \
  done \
  && curl -fsSLO --compressed "https://yarnpkg.com/downloads/$YARN_VERSION/yarn-v$YARN_VERSION.tar.gz" \
  && curl -fsSLO --compressed "https://yarnpkg.com/downloads/$YARN_VERSION/yarn-v$YARN_VERSION.tar.gz.asc" \
  && gpg --batch --verify yarn-v$YARN_VERSION.tar.gz.asc yarn-v$YARN_VERSION.tar.gz \
  && mkdir -p /opt \
  && tar -xzf yarn-v$YARN_VERSION.tar.gz -C /opt/ \
  && ln -s /opt/yarn-v$YARN_VERSION/bin/yarn /usr/local/bin/yarn \
  && ln -s /opt/yarn-v$YARN_VERSION/bin/yarnpkg /usr/local/bin/yarnpkg \
  && rm yarn-v$YARN_VERSION.tar.gz.asc yarn-v$YARN_VERSION.tar.gz \
  && apk del .build-deps-yarn
```

这里就只是简单的通过向官方镜像里安装 nodejs 环境而已，去掉了 `RUN addgroup ...`, `ENTRYPOINT ...`, `CMD ...` 等等会改变镜像的启动方式的命令即可，然后再安装 git 环境，下载好基本的内容就可以了，剩下的可变的因素都通过 Jenkins 项目配置中设置。

```dockerfile
FROM 127.0.0.1:32000/yanjinzh6/jnlp-slave-nodejs:1

RUN apk --update add git less openssh && \
    rm -rf /var/lib/apt/lists/* && \
    rm /var/cache/apk/* && \
    npm install -g hexo --registry=https://registry.npm.taobao.org && \
    git clone https://github.com/yanjinzh6/hexo-notes.git hexo-notes && \
    cd hexo-notes/themes && \
    git clone https://github.com/yanjinzh6/hexo-theme-next.git next
```

通过 `docker tag yanjinzh6/jnlp-slave-hexo:1 127.0.0.1:32000/yanjinzh6/jnlp-slave-hexo:1`, `docker push 127.0.0.1:32000/yanjinzh6/jnlp-slave-hexo:1` 将镜像推送到本地的注册中心去供 Jenkins 使用。

## 配置

个人比较喜欢工作流的项目，感觉就是使用了 Jenkins 包装好的 API 兼容性更好了，当实际上学习成本也比较高，例如当前，需要 clone 多个 git 仓库就碰上不方便的地方了，由于 Jenkins 有 workspace 的概念，所以很多插件的工作模式都是基于当前 workspace 的。

比如 git 插件，可以使用 `git credentialsId: 'xxx', url: 'https://github.com/yanjinzh6/notes.git'` 来 clone 项目，但是没办法设置路径，也许是缺失相应参数了。

还有每次命令都是以 workspace 为当前目录的，比如 `sh "cd a"`，然后再执行 `sh "cd b"`, 结果并不是在 `"${workspace}/a/b"` 中，只是在 `"${workspace}/b"` 中，这样会导致工作流脚本不好使用。

```
node('slave-agent') {
    def home = '/home/jenkins'
    def workspace = 'hexo-notes'
    def next = 'next'
    def cos = 'hexo-cos-tool'
    def source = 'source/_posts'

    stage('clone notes') {
        sh "mkdir ${home}/${workspace}/${source} && cd ${home}/${workspace}/${source} && git clone https://github.com/yanjinzh6/notes.git"
    }
    stage('init workspace') {
        sh "cd ${home}/${workspace} && npm install --registry=https://registry.npm.taobao.org && hexo g"
    }
    stage('init cos tool') {
        sh "cd ${home}/${workspace} && git clone https://github.com/yanjinzh6/hexo-cos-tool.git && cd ${home}/${workspace}/${cos} && npm install --registry=https://registry.npm.taobao.org"
    }
    stage('deploy') {
        withCredentials([string(credentialsId: 'zzz', variable: 'cosAppId'), string(credentialsId: 'yyy', variable: 'cosSecret')]) {
            sh "node ${home}/${workspace}/${cos}/index.js ${cosAppId} ${cosSecret} zzz ap-guangzhou ${home}/${workspace}/public"
        }
    }
}
```

## 总结

现在修改了 Jenkins k8s 插件的配置，使用新的自定义镜像肯定会导致其他项目失效，所以需要解决。还有需要处理站内连接的问题。
