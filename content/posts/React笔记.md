---
title: 'React笔记'
date: 2024-08-28T14:43:33+08:00
draft: true
---

这篇博客主要记录Twitter博主[@_georgemoller](https://x.com/_georgemoller)分享的React技巧。对于我这种React小白很使用。于是搬运过来，并附上个人的一些理解。如有问题请邮箱本人，欢迎各位指正。


### Solve re-render issues with composition

![](/React笔记/georgemoller_1c2c3b.gif)

> 个人理解：功能单一，利用组合模式减少耦合，带来不必要的重新渲染


### Utility Types For Enhanced Reusablity

![](/React笔记/georgemoller_9378eb.gif)

> 个人理解：Typescript Pick的使用


### Bad use case for useCallback

![](/React笔记/georgemoller_ed1930.gif)

> useCallback 使用在复杂逻辑的组件，简单组件不会消耗造成性能问题时不必使用

### Intersection types to extends native props in a component


![](/React笔记/georgemoller_29194f.gif)