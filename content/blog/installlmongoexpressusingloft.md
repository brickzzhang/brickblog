---
title: "使用loft在集群中安装mongo和mongo-express"
date: 2021-03-05T16:25:45+08:00
draft: false
weight: 4
---

## 使用loft安装mongo

1. 启动loft
```shell
loft start
```

2. 浏览器打开"https://localhost:9898"

3. 通过app安装mongo:
![img.png](/img/mongoapp.png)
![img.png](/img/mongoapp1.png)
   
- helm chart内容如下：
```yaml
# 创建mongo server

architecture: standalone
auth:
  database: test
  enabled: true
  password: password
  rootPassword: password
  username: mongouser
externalAccess:
  enabled: false
persistence:
  enabled: false
service:
  port: 27017
  type: ClusterIP
```

chart中，database需指定为admin，mongo-express需连接到管理员库才能进行写操作。

[mongodb chart配置文档](https://artifacthub.io/packages/helm/bitnami/mongodb/10.9.1)

4. 记录mongodb cluster ip，供mongo express连接使用
![img.png](/img/clusterip.png)

## 安装mongo express

与安装mongodb步骤相同，使用chart如下：

```yaml
# 创建mongo express

basicAuthPassword: password
basicAuthUsername: brickzzhang
mongodbAdminPassword: password
mongodbAdminUsername: mongouser
mongodbEnableAdmin: true
mongodbPort: 27017
mongodbServer: 172.16.254.65
service:
  port: 8081
  type: LoadBalancer
serviceAccount:
  create: false
```

`basicAuthPassword`和`basicAuthUsername`分别为登录express使用的账户名和密码。
`mongodbAdminPassword`, `mongodbAdminUsername`为mongodb chart中设置的`auth.password`和`auth.username`，需将`auth.enable`设置为`true`以使能管理员权限。
`mongodbPort`为mongodb chart中`service.port`。
`service.port`和`service.type`分别为express暴露的登录端口和service类型。

[mongo express chart配置文档](https://artifacthub.io/packages/helm/cowboysysop/mongo-express/2.3.1)
