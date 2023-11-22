---
title: "Go: Options Pattern"
date: 2022-03-20T07:52:18Z
draft : false
isCJKLanguage : true
categories:
- Development
tags:
- pattern
- go
enableComment : true
---

**functional options** 提供友好的API，在不改变兼容性的情况下增长参数，且可以提供合理的默认值；

本质上是利用可变参数，保证函数的参数列表具有兼容性，也就是新调用函数时，不需要修改函数的之前调用时的参数；

```go
type Server struct {
	host string
}

type ServerOption func(o *Server)

// Host 修改host
func Host(host string) ServerOption {
	return func(s *Server) {
		s.host = host
	}
}

// 使用Options Pattern后，New函数签名在新增配置项的情况下也是不变的，是具备兼容性的。
func NewServer(opts ...ServerOption) *Server {
	var s Server
    // 可以在这里提供一些合理的默认值
	for _, opt := range opts {
		opt(&s)
	}
	return &s
}

server1 :=  NewServer(Host(1.1.1.1))

```

```go
type Server struct {
	host string
	port int // 新加的port配置项
}

// Port 修改post
func Port(port int) ServerOption {
	return func(o *Server) {
		o.port = port
	}
}

// NOTE: server1处是可以不改动的
server2 :=  NewServer(Host(1.1.1.2), Port(8080))
```

### 延伸阅读:

[options-pattern](https://www.sohamkamani.com/golang/options-pattern)

[functional-options-for-friendly-apis](https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis)

[golang-functional-options-pattern](https://golang.cafe/blog/golang-functional-options-pattern.html)

[functional_option_pattern_is_terrible_in_practice](https://www.reddit.com/r/golang/comments/pwjrjq/functional_option_pattern_is_terrible_in_practice)

