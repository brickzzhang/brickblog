---
title: "docker启动prometheus和grafana"
date: 2021-03-18T16:24:58+08:00
draft: false
weight: 5
---

本地安装prometheus和grafana。

<!--more-->

## 启动prometheus

需提前安装好docker

1. 下载prometheus的docker镜像

```shell
docker pull prom/prometheus:latest
```

2. 编辑prometheus配置文件

```yaml
# A scrape configuration scraping a Node Exporter and the Prometheus server
# itself.
scrape_configs:
  # Scrape Prometheus itself every 5 seconds.
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']

  # Scrape the Node Exporter every 5 seconds.
  - job_name: 'brickzzhang'
    scrape_interval: 5s
    static_configs:
      - targets: ['10.95.22.58:2112']
```

配置文件中, `targets`字段需通过`ifconfig`命令获取本机内网ip地址, 而非`localhost`, 否则将会监听docker内网地址。

3. docker启动prometheus

```shell
docker run -p 9090:9090  -v /yourpath/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

4. 打开浏览器localhost:9090端口，可看到上报信息

![img.png](/img/prometheus.png)


## 启动grafana

1. 下载grafana镜像

```shell
docker pull grafana/grafana
```

2. docker启动grafana

```shell
docker run -d --name=grafana -p 3000:3000 grafana/grafana
```

3. 浏览器登录grafana(localhost:3000)

- 编辑grafana数据源

![img.png](/img/grafanadatasource.png)
![img.png](/img/grafana2.png)

图中url若填写`localhost`，grafana会当作docker内部地址处理，需填写本机内网IP。

- 导入prometheus模板

![img.png](/img/grafana3.png)

即可自行编辑panel观察监控数据。
