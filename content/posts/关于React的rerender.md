+++
title = '关于React的rerender'
date = 2024-12-04T16:06:31+08:00
draft = true
+++

作为一个React初学者，在写React程序的时候经常遇到rerender的问题。由于写的都比较简单，也没有关注rerender带来的问题。但是遇到问题了又到处Google找问题，还是比较浪费时间的。所以这次下定决心学习一下React的Rerender的机制。

当Component第一次出现在屏幕上，我们称之为`mounting`。React会初始化Component的state, 运行hooks，添加element到dom中。

当React检测到Component不再需要时，我们称之为`unmounting`。React会执行clean-up，销毁组件及其相关的state，最后从dom中移除element。

`rerender`是在已经存在的Component的基础上，一些state发生了变化，导致Component重新渲染。相比于`mounting`会更加轻量。

每次rerender都始于state的改变。你可以把React App想象成1棵树。state变化的那个节点会导致这个节点下面的所有分支节点都重新渲染。

![](/关于React的rerender/rerender-state.png)

你可能听说过props改变了也会导致Component rerender。但这是个误解，出现这个根本原因还是因为state发生了变化。

### 参考资料

* [React re-render 完全指南](https://juejin.cn/post/7254443448562974775)
* [Advanced React deep dives, investigations, performance patterns and techniques (Nadia Makarevich).pdf](/books/Advanced%20React%20deep%20dives,%20investigations,%20performance%20patterns%20and%20techniques%20(Nadia%20Makarevich)%20(Z-Library).pdf)