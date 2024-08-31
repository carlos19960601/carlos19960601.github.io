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


### Don't listen for ref changes with useEffect

![](/React笔记/georgemoller_8216ce.gif)

> 本人曾经这么干过


### useEffect vs useLayoutEffect

![](/React笔记/georgemoller_d587ed.gif)

> 个人理解：useLayoutEffect在DOM更新后，浏览器绘制前执行。useEffect在DOM更新后，浏览器绘制后执行。

### Separation of Concerns in React

![](/React笔记/georgemoller_e8833e.gif)

> React中的关注点分离
> 这是一个重要的软件设计原则，在React开发中也同样适用。它指的是将代码按照不同的功能或责任进行划分和组织，使每个部分专注于特定的任务。在React中，这通常体现为:
> 1. 将UI和逻辑分离
> 2. 将状态管理与渲染逻辑分离
> 3. 将可复用的逻辑抽离成自定义Hook
> 4. 将大型组件拆分成更小的、功能单一的组件
> 
> 遵循这一原则可以提高代码的可维护性、可读性和可重用性。

### Easy to read abstraction over array state

![](/React笔记/georgemoller_ae4db6.gif)

> 个人理解：对数组状态进行抽象，简化逻辑

### How to import Server Components from Client Components

![](/React笔记/georgemoller_c72511.gif)

> 个人理解：服务端渲染框架中使用的小技巧

### Avoid Transforming Data in useEffect

![](/React笔记/georgemoller_bc91e5.gif)

> 个人理解：在useEffect中避免对数据进行转换，避免不必要的重新渲染
> 
> 原因：
>   * 性能问题：在 useEffect 中转换数据可能导致不必要的重渲染。
>   * 代码复杂性：可能使组件逻辑变得难以理解和维护。
> 
> 更好的做法：
>   * 在渲染过程中直接转换数据。
>   * 使用 useMemo 缓存计算结果。

### How and when to use flushSync

![](/React笔记/georgemoller_cf6586.gif)

### Defer creation of non-primitive values if you are using them within a dependency array

![](/React笔记/georgemoller_a5e9d8.gif)

> 个人理解: useEffect使用Tips

### Avoid premature optimization

![](/React笔记/georgemoller_622a12.gif)

### How to expose custom refs with useImperativeHandle hook

![](/React笔记/georgemoller_10ebf7.gif)


### Separate business logic from UI

![](/React笔记/georgemoller_e78ede.gif)

### How to virtualize large lists

![](/React笔记/georgemoller_2e4bec.gif)

### Avoid difficult to read conditionals in React

![](/React笔记/georgemoller_54b4e2.gif)


### Using inferred types in React

![](/React笔记/georgemoller_a5281a.gif)

### Avoid Provider Wrapping Hell

![](/React笔记/georgemoller_f90099.gif)


### useEffect cheatsheet

![](/React笔记/georgemoller_2aae18.gif)
