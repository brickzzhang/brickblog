---
title: "使用protoc生成代码时报错"
date: 2021-04-24T12:41:54+08:00
draft: false
weight: 7
---

两个项目使用了grpc，其中一个项目在使用protoc生成代码时报错:
``` text
--go_out: protoc-gen-go: plugins are not supported; use 'protoc --go-grpc_out=...' to generate gRPC`
```
另一个项目则可正确生成代码。

<!--more-->

## 问题描述

两个项目使用了grpc，其中一个项目在使用protoc生成代码时报错:
``` text
--go_out: protoc-gen-go: plugins are not supported; use 'protoc --go-grpc_out=...' to generate gRPC`
```
另一个项目则可正确生成代码。

报错protoc命令：

`"protoc -I. -I api -I ${GOOGLE_PB_PKG_PATH}/googleapis -I api/third_party --go_out=plugins=grpc:. api/*.proto"`

不报错protoc命令：

`"protoc -I. -I api -I ${GOOGLE_PB_PKG_PATH}/googleapis -I api/third_party --go_out=. --go-grpc_out=. ${version}/*.proto"`

## 问题分析

1. 报错原因

参考[issue](https://github.com/golang/protobuf/issues/1070), `protoc-gen-go`库迁移到google组织维护后，暂未提供对grpc的支持。尝试使用目前可下到的`protoc-gen-go-grpc`版本也无法解决问题。但issue的comment也提到，`protoc-gen-go`原仓库`github.com/golang/protobuf/protoc-gen-go`会一直提供对grpc的支持，所以解决问题需将`protoc-gen-go`仓库的指向有goole改为github的旧版本。

## 解决方法

1. 若保留现有的`protoc-gen-go`版本，[issue](https://github.com/golang/protobuf/issues/1070)底下的一个comment已经提供了解决方法如下：
``` text
The practical effect is that when using this generator you will need to specify two flags on the protoc command line (or run protoc twice):

protoc --go_out=. --go-grpc_out=. foo.proto
```

这种方法与不报错项目使用的命令一致。

2. 另一种方法为替换`protoc-gen-go`为github维护的版本，使用命令如下：

``` bash
go install github.com/golang/protobuf/protoc-gen-go@v1.4.0
```

可workaround掉这个报错。

## 参考

1. [issue](https://github.com/golang/protobuf/issues/1070)
2. [掘金](https://juejin.cn/post/6844904134332645383)
