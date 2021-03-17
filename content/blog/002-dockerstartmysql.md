---
title: "Start MySQL from Docker"
date: 2021-02-19T16:39:33+08:00
draft: false
weight: 2
---

从docker启动MySQL命令。

<!--more-->

## 从docker启动MySQL命令

```shell
docker run --name my-mysql -e MYSQL_ROOT_PASSWORD=mysql@123 -d mysql

docker network create mysql-net

docker network connect mysql-net my-mysql

docker run -it --network mysql-net --rm mysql mysql -hmy-mysql -uroot -p
```
