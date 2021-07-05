---
title: "web服务对接swagger ui进行api调试"
date: 2021-07-02T17:14:47+08:00
draft: false
weight: 15
---

web服务进行api开发时，可在后端配置启用swagger，通过swagger ui进行接口调试。本文将使用grpc框架，配置一个支持swagger debug的后端服务。

<!--more-->

##  需要的Route
swagger ui增加调试需要对接三个接口：

1. 自定义首页

接口使用自定义swagger首页路径，e.g. `/sw/`。接口返回两个文件：`index.html`首页和接口列表文件`swagger-config.json`。`swagger-config.json`格式为如下:
```json
{
    "urls":[
        {
            "name":"swagger/hello_world",
            "url":"/swagger/swagger/hello_world.swagger.json"
        }
    ]
}
```
`swagger-config.json`文件每个url由`/swagger`+`swagger file path`两部分组成。`swagger file path`为对应swagger文件的存放路径。第二个接口将使用此路径请求具体的文件内容进行展示。

请求url格式如下：
`http://localhost:8082/sw`

2. 获取swagger.json文件内容

接口使用`/swagger/`作为固定路径，对传入的url进行解析，在去掉`/swagger/`头部后，将余下部分拼接为server端swagger.json文件的存储路径，将对应路径上的文件返回。

请求url格式如下：
`http://localhost:8082/swagger/swagger/application/v1/application_service.swagger.json`

3. 业务接口

接口使用service中定义每个接口url作为请求路径，通过反向代理将swagger调试的业务流量代理到server端grpc gateway端口上，就可以获取业务逻辑的真是返回。

请求url格式如下：
`http://localhost:8092/v1/helloworld`


## 关键实现

1. 使用`http.FileServer`返回静态文件：

接口1和2都需要返回静态文件，可使用http提供的FileServer方法实现。方法说明如下：
```text
// FileServer returns a handler that serves HTTP requests
// with the contents of the file system rooted at root.
//
// As a special case, the returned file server redirects any request
// ending in "/index.html" to the same path, without the final
// "index.html".
//
// To use the operating system's file system implementation,
// use http.Dir:
//
//     http.Handle("/", http.FileServer(http.Dir("/tmp")))
//
// To use an fs.FS implementation, use http.FS to convert it:
//
//	http.Handle("/", http.FileServer(http.FS(fsys)))
```

2. 使用`httputil.NewSingleHostReverseProxy`进行反向代理

接口3使用`httputil.NewSingleHostReverseProxy`将swagger端口接收的流量转发到grpc gateway使用的业务端口上，方法说明如下：
```text
// NewSingleHostReverseProxy returns a new ReverseProxy that routes
// URLs to the scheme, host, and base path provided in target. If the
// target's path is "/base" and the incoming request was for "/dir",
// the target request will be for /base/dir.
// NewSingleHostReverseProxy does not rewrite the Host header.
// To rewrite Host headers, use ReverseProxy directly with a custom
// Director policy.
```

## 说明

go的http包，如果匹配路径中带有`/`，会自动增加一个匹配规则不带`/`后缀的路由。所以接口1的路由定义为`/sw/`会更加通用。

## 代码

相关代码可参考[grpc-helloworld](https://github.com/brickzzhang/grpc-helloworld)。

## 参考

1. [利用go反向代理转发api](https://www.cnblogs.com/mind-water/p/11061034.html)
2. [swagger-ui](https://github.com/swagger-api/swagger-ui/tree/master/dist)
3. [http下FileServer使用](https://blog.csdn.net/Edu_enth/article/details/89519564)
4. [http.FileServer实现静态文件服务](https://shockerli.net/post/golang-pkg-http-file-server/)
5. [go和http包默认路由匹配规则](https://bbs.huaweicloud.com/blogs/143838)