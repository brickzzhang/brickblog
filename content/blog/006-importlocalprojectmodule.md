---
title: "非开源项目中go mod导入本地包"
date: 2021-04-18T13:30:13+08:00
draft: false
weight: 6
---

非开源go项目，如何导入本项目的另一个module。

<!--more-->

## 问题描述

项目结构如下：

``` text
├── demo
├── go.mod
├── go.sum
├── main
│   └── main.go
├── tmpl
│   ├── service.go.extension.gotmpl
│   ├── service.go.gotmpl
│   └── service.go.tmpl
└── tmplgen
    ├── go.mod
    ├── go.sum
    ├── service.go.extension.gotmpl.go
    └── service.go.gotmpl.go
```

上述项目中，main.go中导入tmplgen下template包。使用`import "tmplgen/template"`的方式导入，执行`go mod tidy`报错：

``` text
template-demo go mod tidy
template-demo/main imports
	tmplgen/template: package tmplgen/template is not in GOROOT (/usr/local/Cellar/go/1.16.1/libexec/src/tmplgen/template)
```

## 问题分析

观察报错报错信息，可见go mod无法在$GOROOT中找到template包。template包定义在当前目录下，所以需指定template包的导入路径。指定go mod包的导入路径，就需要使用`go mod replace`。在main包的`go.mod`中添加：

``` text
replace tmplgen/template => ./tmplgen
```

## 补充

1. template包在被导入之前，需通过`go mod init template`命令初始化，否则会报错：

``` text
➜  template-demo go mod tidy
go: tmplgen/template@v0.0.0-00010101000000-000000000000 (replaced by ./tmplgen): reading tmplgen/go.mod: open /Users/brick/workspace/pl-pro/template-demo/tmplgen/go.mod: no such file or directory
```

go 官方blog说明：

``` text
As of Go 1.11, the go command enables the use of modules when the current directory or any parent directory has a go.mod, provided the directory is outside $GOPATH/src.
```

[import blog link](https://blog.golang.org/using-go-modules)

2. go module路径的第一部分应当为域名或路径：

[stack overflow link](https://stackoverflow.com/questions/58473656/malformed-module-path-xxxx-xxxx-uuid-missing-dot-in-first-path-element-when-mi)

## 参考

1. [go mod如何导入本地包](https://www.cnblogs.com/wind-zhou/p/12824857.html#_labelTop)
