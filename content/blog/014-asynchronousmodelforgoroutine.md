---
title: "一个goroutine异步处理任务的模型"
date: 2021-06-20T11:17:46+08:00
draft: false
weight: 14
---

一个只使用goroutine(不引入mq等第三方组件)进行异步任务处理的模型。

<!--more-->

业务场景中有时会遇到耗时比较长的接口，如果采用同步处理逻辑，会带来极度不友好的交互体验。通过将一个post接口拆分为post+get两个接口(post负责下达请求，get负责轮询作业状态)，以异步的方式来优化用户体验。当然，异步处理的方式也需要加入超时控制避免goroutine泄漏。

本文整理出一个简单的异步业务处理模型来达到此目的。

## 代码

```go
package main

import (
	"context"
	"log"
	"time"
)

type Result struct {
	Value  interface{}
	Err    error
	ErrMsg string
}

func longTimeTask(ctx context.Context, c chan Result) {
	// do some long time cost task
	time.Sleep(3 * time.Second)

	c <- Result{
		Value:  "no error occured",
		Err:    nil,
		ErrMsg: "",
	}
}

func doWithTimeout(ctx context.Context) {
	// extract values from old ctx
	// init a new timeout context
	ctxOut, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	c := make(chan Result)
	go func() {
		longTimeTask(ctxOut, c)
	}()

	select {
	case <-ctxOut.Done():
		// timeout happened
		log.Printf("timeout happened")
	case result := <-c:
		// task finished in time
		// handle result
		log.Printf("long time task finished, result is: %+v", result)
	}
}

func businessAPI(ctx context.Context) error {
	// do something

	go func(context.Context) {
		doWithTimeout(ctx)
	}(ctx)

	return nil
}

func main() {
	// api procedure
	_ = businessAPI(context.TODO())
	log.Printf("businessAPI done")

	// make sure all goroutines are exited, cuz this is a main goroutine
	time.Sleep(10 * time.Second)
}
```

## 测试结果
1. 将longTimeTask中的超时设置为8s,doWithTimeout中的超时设置为5s:
```bash
2021/06/20 14:59:31 businessAPI done
2021/06/20 14:59:36 timeout happened
```

2. 将longTimeTask中的超时设置为3s,doWithTimeout中的超时设置为5s:
```bash
2021/06/20 15:00:35 businessAPI done
2021/06/20 15:00:38 long time task finished, result is: {Value:no error occured Err:<nil> ErrMsg:}
```

## 简单说明
1. doWithTimeout函数开始切记需要用Background初始化新的context，否则如果父ctx设置的超时时间比子ctx设置的小，将会以父ctx设置的时间为准：
```go
// WithDeadline returns a copy of the parent context with the deadline adjusted
// to be no later than d. If the parent's deadline is already earlier than d,
// WithDeadline(parent, d) is semantically equivalent to parent. The returned
// context's Done channel is closed when the deadline expires, when the returned
// cancel function is called, or when the parent context's Done channel is
// closed, whichever happens first.
```
2. doWithTimeout使用select阻塞获取超时或者任务返回的结果
3. doWithTimeout和longTimeTask通过第二个channel参数进行结果传递
4. longTimeTask函数只负责产生结果，将结果处理逻辑放到doWithTimeout中
