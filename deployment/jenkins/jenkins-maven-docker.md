---
title: 构建 maven 和 docker 的 Jenkins 镜像
date: 2020-04-14 15:00:00
tags: 'Jenkins'
categories:
  - ['部署', 'CI']
permalink: jenkins-maven-docker
photo:
---

使用官方的源镜像为基础镜像, 添加 maven 包和 docker 客户端, 这样主要方便共享宿主机的 maven 缓存和 docker 镜像层缓存

```Dockerfile
FROM jenkins/jenkins:2.220

# 系统为 Debian 9

USER root

ARG MAVEN_VERSION=3.6.3
ARG MAVEN_SHA=c35a1803a6e70a126e80b2b3ae33eed961f83ed74d18fcd16909b2d44d7dada3203f1ffe726c17ef8dcca2dcaa9fca676987befeadc9b9f759967a8cb77181c0
ARG MAVEN_BASE_URL=https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/${MAVEN_VERSION}/binaries

RUN echo "deb http://mirrors.aliyun.com/debian/ stretch main non-free contrib" > /etc/apt/sources.list \
  && echo "deb-src http://mirrors.aliyun.com/debian/ stretch main non-free contrib" >> /etc/apt/sources.list \
  && echo "deb http://mirrors.aliyun.com/debian-security stretch/updates main" >> /etc/apt/sources.list \
  && echo "deb-src http://mirrors.aliyun.com/debian-security stretch/updates main" >> /etc/apt/sources.list \
  && echo "deb http://mirrors.aliyun.com/debian/ stretch-updates main non-free contrib" >> /etc/apt/sources.list \
  && echo "deb-src http://mirrors.aliyun.com/debian/ stretch-updates main non-free contrib" >> /etc/apt/sources.list \
  && echo "deb http://mirrors.aliyun.com/debian/ stretch-backports main non-free contrib" >> /etc/apt/sources.list \
  && echo "deb-src http://mirrors.aliyun.com/debian/ stretch-backports main non-free contrib" >> /etc/apt/sources.list \
  # apt-get update Hash Sum mismatch
  # && rm -fR /var/lib/apt/lists/* && mkdir /var/lib/apt/lists/partial \
  && apt-get update \
  && apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common \
  # && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo Asia/Shanghai > /etc/timezone \
  # 下载
  && mkdir -p /usr/share/maven /usr/share/maven/ref/repository \
  && echo "${MAVEN_BASE_URL}/apache-maven-${MAVEN_VERSION}-bin.tar.gz" \
  && curl -fsSL -o /tmp/apache-maven.tar.gz ${MAVEN_BASE_URL}/apache-maven-${MAVEN_VERSION}-bin.tar.gz \
  # 校验文件
  # && echo "${MAVEN_SHA}  /tmp/apache-maven.tar.gz" | sha512sum -c - \
  # 解压
  && tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 \
  # clear
  && rm -f /tmp/apache-maven.tar.gz \
  && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn \
  # 使用 aliyun 源安装 docker, 这里安装后镜像比较大, 因为镜像基本系统与宿主机比较像可以使用宿主机的 docker 客户端
  # && curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun \
  # 将 jenkins 用户加入 docker 用户组
  && usermod -aG 999 jenkins \
  # 修改 maven 本地仓库为普通用户权限
  && chown 1000:1000 /usr/share/maven/ref/repository \
  # 清理
  && apt-get purge && apt-get autoremove&& apt-get clean \
  && apt-get remove apt-transport-https ca-certificates curl gnupg-agent software-properties-common -y \
  # 这里不能删除掉 /tmp 因为 Jenkins 会使用到
  && rm -rf /var/lib/apt/lists/* && rm -rf /tmp/*

ENV MAVEN_HOME /usr/share/maven
VOLUME /usr/share/maven/ref/repository
COPY settings.xml /usr/share/maven/conf/settings.xml

USER jenkins
```

```xml
<?xml version="1.0" encoding="UTF-8"?>

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <localRepository>/usr/share/maven/ref/repository</localRepository>
  <mirrors>
     <mirror>
        <id>aliyun-nexus</id>
        <mirrorOf>central</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      </mirror>
      <mirror>
        <id>CN</id>
        <name>OSChina Central</name>
        <url>http://maven.oschina.net/content/groups/public/</url>
        <mirrorOf>central</mirrorOf>
      </mirror>
  </mirrors>
</settings>
```

```sh
# 打包一下
docker build -t maven-jenkins:2.220 .
```

```yaml
version: '3'
services:
  jenkins:
    image: maven-jenkins:2.220
    container_name: jenkins
    # user: root
    volumes:
      - /home/jenkins/jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
      # dood
      - /usr/bin/docker:/usr/bin/docker
      # 使用统一 .m2 目录
      - /home/.m2/repository:/usr/share/maven/ref/repository
    environment:
      - JENKINS_HOST_HOME=/var/jenkins_home
      - JENKINS_ADMIN_ID=root
      - JENKINS_ADMIN_PW=root123456
      - JENKINS_MODE=master
      - JAVA_OPTS=-Djenkins.install.runSetupWizard=false
    ports:
      - "8080:8080"
      - "5000:5000"
      - "50000:50000"
    extra_hosts:
      # 转发
      - "mirrors.jenkins-ci.org:127.0.0.1"
    # network_mode: "host"
```

可以使用基于 alpine 镜像来实现较小打包

```Dockerfile
# =====================================================================
# Jenkins with DooD (Docker outside of Docker) and integration maven
# =====================================================================
FROM jenkins/jenkins:alpine

USER root

ARG MAVEN_VERSION=3.6.0
ARG MAVEN_SHA=fae9c12b570c3ba18116a4e26ea524b29f7279c17cbaadc3326ca72927368924d9131d11b9e851b8dc9162228b6fdea955446be41207a5cfc61283dd8a561d2f
ARG MAVEN_BASE_URL=https://apache.osuosl.org/maven/maven-3/${MAVEN_VERSION}/binaries

RUN echo "https://mirrors.aliyun.com/alpine/v3.8/main/" > /etc/apk/repositories \
  && echo "https://mirrors.aliyun.com/alpine/v3.8/community/" >> /etc/apk/repositories \
  && apk add --no-cache tar procps tzdata shadow docker \
  && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo Asia/Shanghai > /etc/timezone \
  && mkdir -p /usr/share/maven /usr/share/maven/ref/repository \
  && curl -fsSL -o /tmp/apache-maven.tar.gz ${MAVEN_BASE_URL}/apache-maven-${MAVEN_VERSION}-bin.tar.gz \
  && echo "${MAVEN_SHA}  /tmp/apache-maven.tar.gz" | sha512sum -c - \
  && tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 \
  && rm -f /tmp/apache-maven.tar.gz \
  && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn \
  && usermod -aG 999 jenkins \
  && chown 1000:1000 /usr/share/maven/ref/repository \
  && apk del tzdata shadow tar

ENV MAVEN_HOME /usr/share/maven
VOLUME /usr/share/maven/ref/repository
COPY settings.xml /usr/share/maven/conf/settings.xml

USER jenkins
```

参考

- [制作自己的 Jenkins 镜像](https://www.jianshu.com/p/fb7b31a25358)
