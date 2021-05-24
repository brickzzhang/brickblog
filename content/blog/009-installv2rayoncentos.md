---
title: "centos系统上安装v2ray"
date: 2021-05-24T10:41:47+08:00
draft: false
weight: 9
---

在centos上安装v2ray教程

<!--more-->

## 安装步骤

1. 下载安装脚本

```bash
curl -s -L https://git.io/v2ray.sh
```

2. 根据教程安装v2ray软件及其依赖包

[教程链接](https://github.com/233boy/v2ray/wiki/V2Ray%E6%90%AD%E5%BB%BA%E8%AF%A6%E7%BB%86%E5%9B%BE%E6%96%87%E6%95%99%E7%A8%8B)

3. 停止v2ray进程

`v2ray stop`

4. 用以下内容替换`/etc/v2ray/config.json`文件

```json
{
    "inbounds":[
        {
            "port":*****,
            "protocol":"vmess",
            "settings":{
                "clients":[
                    {
                        "id":"*******",
                        "level":1,
                        "alterId":64
                    }
                ]
            },
            "streamSettings":{
                "network":"tcp"
            }
        }
    ],
    "outbounds":[
        {
            "protocol":"freedom",
            "settings":{

            }
        },
        {
            "protocol":"blackhole",
            "settings":{

            },
            "tag":"blocked"
        }
    ],
    "routing":{
        "rules":[
            {
                "type":"field",
                "ip":[
                    "geoip:private"
                ],
                "outboundTag":"blocked"
            }
        ]
    }
}
```

`******`部分需替换为实际内容。

4. 使用后台命令启动v2ray后台进程

```bash
nohup /usr/bin/v2ray/v2ray -config /etc/v2ray/config.json > /dev/null 2>&1 &
```

## 说明

使用[教程链接](https://github.com/233boy/v2ray/wiki/V2Ray%E6%90%AD%E5%BB%BA%E8%AF%A6%E7%BB%86%E5%9B%BE%E6%96%87%E6%95%99%E7%A8%8B)中封装的v2ray命令也可管理v2ray后台进程的生命周期。因为没有对教程中封装的v2ray命令行工具进行代码分析，不清楚配置数据的安全性，所以本文重新使用后台进程启动。

## 参考

[V2Ray搭建详细图文教程](https://github.com/233boy/v2ray/wiki/V2Ray%E6%90%AD%E5%BB%BA%E8%AF%A6%E7%BB%86%E5%9B%BE%E6%96%87%E6%95%99%E7%A8%8B)
