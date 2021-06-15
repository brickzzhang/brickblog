---
title: "基于alpine镜像制作go和goimports的Docker镜像"
date: 2021-06-01T16:58:04+08:00
draft: false
weight: 11
---

基于alpine制作go docker镜像

<!--more-->

## 步骤

1. 下载go源码包
[For Linux](https://golang.org/doc/install?download=go1.16.4.linux-amd64.tar.gz)

2. 解压压缩包至当前文件夹

3. 编辑Dockerfile
```Dockerfile
FROM csighub.tencentyun.com/pulse-line/alpine

COPY go /usr/local/go

ENV PATH=$PATH:/usr/local/go/bin:/root/go/bin \
	GOROOT=/usr/local/go

WORKDIR /usr/local/test/

RUN ["go", "install", "golang.org/x/tools/cmd/goimports@latest"]
```

4. 制作Docker镜像
```bash
docker build -t alpine-go-goimports:v0.0.1 .
```

5. 运行容器
```bash
docker run -it alpine-go-goimports:v0.0.1
```
