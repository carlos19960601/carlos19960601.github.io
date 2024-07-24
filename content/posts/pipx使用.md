---
title: 'pipx使用'
date: 2024-07-24T16:43:50+08:00
draft: false
---

# pipx是什么？

可以参加[pipx](https://github.com/pypa/pipx)的介绍。

作为一个python小白，看了几遍，搜索了多篇博客了解`pipx`是什么，仍然不懂是个啥。

我在这仅说明一下我自己的理解。`pipx`相当于一个安装工具。`python`有自己的安装包的工具`pip`。那么`pip`和`pipx`有什么区别呢？

如果直接`pip`安装包会安装在`python`目录下的`site-packages`下。由于这是个全局目录。如果在不同的项目里面，可能使用的各个包版本不同，可能会产生冲突。所以我们偏向使用一个干净的`site-package`。其实`python`提供了解决方案--虚拟环境。`pipx`只是基于这个概念，让使用者更好的使用。

> 注意 pipx 安装的是一些可执行的python应用。比如 poetry, jupyter等

# 简单使用

安装pipx

```shells
brew install pipx
pipx ensurepath
```

安装jupyter

可以使用`--python`参数指定虚拟环境python`版本。如果不指定，使用的是你电脑上安装的最新的`python`版本。

```shell
pipx install notebook --python=python3.11 --force
```

执行后, jupyter就会被安装在 `~/.local/pipx/venvs/notebook/bin`。可以看到在`pipx`的`venvs`目录下面。也就说`pipx`安装的每个应用都会有自己的虚拟环境。

在写jupyter notebook的时候，我们可能需要`numpy`，`panda`等这些`python`库。所以我们需要将这些安装到notebook的虚拟环境中，才能使用

```shells
pipx inject numpy
```
