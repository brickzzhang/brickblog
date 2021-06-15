---
title: "对Go Context WithTimeout超时后清理资源的误解"
date: 2021-06-15T20:02:13+08:00
draft: false
weight: 12
---

关于go context.WithTimeout理解的误区

<!--more-->

## 问题现象

### 先贴代码:

```go
package main

import (
	"context"
	"log"
	"time"
)

func test(ctx context.Context) {
	defer func() {
		log.Printf("in defer\n")
	}()
	for {
		select {
		case <-ctx.Done():
			log.Printf("done\n")
			return
		default:
			log.Printf("in test\n")
			time.Sleep(10 * time.Second)
			return
		}
	}
}

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
	defer cancel()

	go test(ctx)

	time.Sleep(12 * time.Second)
}
```

### 运行结果

```bash
2021/06/15 20:15:58 in test
2021/06/15 20:16:08 in defer
```

代码中使用协程运行test函数，之后父协程sleep 12s。context使用WithTimeout函数设置的超时时间为3s。按照WithTimeout函数的注释：

```go
// WithTimeout returns WithDeadline(parent, time.Now().Add(timeout)).
//
// Canceling this context releases resources associated with it, so code should
// call cancel as soon as the operations running in this Context complete:
//
// 	func slowOperationWithTimeout(ctx context.Context) (Result, error) {
// 		ctx, cancel := context.WithTimeout(ctx, 100*time.Millisecond)
// 		defer cancel()  // releases resources if slowOperation completes before timeout elapses
// 		return slowOperation(ctx)
// 	}
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {
	return WithDeadline(parent, time.Now().Add(timeout))
}
```

调用cancel(或者context超时)后，会清理context相关的资源。很容易误解为goroutine会在context超时后自动清理退出。但输出结果与预期相反，test协程仍然会跑完default分支的sleep流程，之后经过defer阶段退出。

修改test函数：
```go
func test(ctx context.Context) {
	defer func() {
		log.Printf("in defer\n")
	}()
	for {
		select {
		case <-ctx.Done():
			log.Printf("done\n")
			return
		default:
			log.Printf("in test\n")
			time.Sleep(1 * time.Second)
		}
	}
}
```

得到输出如下：
```bash
2021/06/15 20:38:05 in test
2021/06/15 20:38:06 in test
2021/06/15 20:38:07 in test
2021/06/15 20:38:08 done
2021/06/15 20:38:08 in defer
```

test函数在3s之后，接收到context发送的done信号，之后经过defer阶段退出。

### 结论

对比发现WithTimeout(或WithDeadline)在超时(或调用cancel)后，并不会如comment描述那样清理相关资源。comment中`Canceling this context releases resources associated with it`中的`it`应该理解为`context`而非`goroutine`。:( sad...

所以一旦遇到需要关注goroutine退出操作时，for select套餐一般来说是无法避免的。
