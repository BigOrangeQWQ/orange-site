---
title: "浅谈 Docker in Docker 的两种方法"
date: 2024-08-29T20:55:24+08:00
slug: 2024-08-29-docker-in-docker
type: posts
draft: false
categories:
  - computer
tags:
  - 容器
  - Docker
  - DinD
---

# DinD (Docker in Docker) 简介

在运行某一个容器时，该容器若存在运行其它容器的需求时，需要在需要运行其它容器的容器安装一个 Docker 环境

但是我们会发现安装一个 Docker 环境来启动其它容器的代价是巨大的，这样的容器会十分笨重且难以管理

于是可以自然的想到一个很重要的点，**需要在容器里面复用容器外的 Docker 环境**，这样既能启动容器，也很轻便快捷。

怎么复用呢？有什么方法能够支持复用？

当我们每次启动 Docker 容器时，是不是可以提供或者同步什么东西过去？环境变量，挂载文件还是网络呢？

是不是可以通过网络的形式做通信？藉此来连接到外部的 Docker 环境，我们是不是只需要一个 Client 而不需要启动一个 Docker 了。

有两种技术可言，一个是 [DinD](https://hub.docker.com/_/docker/) 容器，启动一个 DinD 容器，让其它容器通过 Network 形式连接到该容器后来创建新的容器，但这种方式因为要使用到 [--privilege](https://docs.docker.com/engine/containers/run/#runtime-privilege-and-linux-capabilities) 选项而容易有[安全风险](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/)问题。

第二个 DooD(Docker outside of Docker) 使用套接字(Socket)的解决方案，我们只需要将 `/var/run/docker.sock` 文件挂载到容器内部即可：

`docker run -v /var/run/docker.sock:/var/run/docker.sock ...`

只需要在需要启动其它的容器内安装 Docker Cli 或者是其它能连接的 Client 即可解决此事。

