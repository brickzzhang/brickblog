---
title: "构建Docker镜像"
date: 2021-05-31T11:21:07+08:00
draft: false
---

创建docker镜像过程

<!--more-->

## 创建命令

1. 获取基础镜像
```bash
docker pull csighub.tencentyun.com/pulse-line/alpine
```

2. 运行容器
```bash
docker run -it csighub.tencentyun.com/pulse-line/alpine
```

3. 查看容器ID
```bash
docker ps
```

4. 修改镜像
```bash
apk add zip
```

5. 提交修改后的镜像
```bash
docker commit -m "add zip" -a "brickzzhang" 033c92e8f55c csighub.tencentyun.com/pulse-line/alpine-zip:v0.0.1
```

6. 给提交后的镜像打标签
```bash
docker tag csighub.tencentyun.com/pulse-line/alpine-zip:v0.0.1 csighub.tencentyun.com/pulse-line/alpine-zip:latest
```

7. 登录image hub
```bash
docker login csighub.tencentyun.com
```

8. 推送镜像到hub
```bash
docker push csighub.tencentyun.com/pulse-line/alpine-zip:latest
```

## 说明
推荐的镜像制作方式是通过Dockerfile而非commit命令，因为无法通过commit生成的镜像得知镜像制作步骤。本文制作背景为减少项目ci过程中的超时概率，所以需要通过commit提前把镜像打好。

## 参考
[利用 commit 理解镜像构成](https://yeasy.gitbook.io/docker_practice/image/commit)
