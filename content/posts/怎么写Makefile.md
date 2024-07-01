---
title: '怎么写Makefile'
date: 2024-07-01T17:11:40+08:00
draft: false
---

## 基本语法

```makefile
targets: prerequisites
	command
	command
	command
```

## 变量

**声明变量**

```makefile
NAME=ClashV
BINDIR=bin
```

**使用`$()`来引用变量**

```makefile
PLATFORM_LIST = \
	darwin-amd64 \
	darwin-amd64-compatible \

all-arch: $(PLATFORM_LIST)
```

### 自动变量

`$@`: 表示target

```makefile
darwin-amd64:
	GOARCH=amd64 GOOS=darwin $(GOBUILD) -o $(BINDIR)/$(NAME)-$@
```

## 条件判断

```makefile
ifeq ($(BRANCH),Alpha)
VERSION=alpha-$(shell git rev-parse --short HEAD)
else ifeq ($(BRANCH),Beta)
VERSION=beta-$(shell git rev-parse --short HEAD)
else ifeq ($(BRANCH),)
VERSION=$(shell git describe --tags)
else
VERSION=$(shell git rev-parse --short HEAD)
endif
```


## 函数

### shell

```makefile
BRANCH=$(shell git branch --show-current)
```


## 相关链接

* [Makefile Tutorial](https://makefiletutorial.com/)