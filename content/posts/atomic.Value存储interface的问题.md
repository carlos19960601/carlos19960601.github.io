---
title: 'atomic.Value存储interface的问题'
date: 2024-06-26T17:39:28+08:00
draft: false
tags: 
  - Golang
  - 问题
categories:
  - Golang
---

先看这段代码

```go
import (
	"io"
	"net/http"
	"sync/atomic"
)

func main() {
	var v atomic.Value
	var err error

	err = &http.ProtocolError{}
	v.Store(err)
	err = io.EOF
	v.Store(err)
}
```

运行后会报错 `panic: sync/atomic: store of inconsistently typed value into Value`。

原因是`atomic.Value.Store`需要类型是一致的。在这里`err`类型发生了变化，虽然他们都是`error`接口类型。具体参考[Issues#22550](https://github.com/golang/go/issues/22550)


怎么解决？包装一层就能运行了。

```go
type tValue[T any] struct {
	value T
}

func main() {
	var v atomic.Value
	var err error

	err = &http.ProtocolError{}
	v.Store(tValue[error]{err})
	err = io.EOF
	v.Store(tValue[error]{err})
}
```