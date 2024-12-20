---
title: 'Css学习笔记'
date: 2024-11-30T22:08:35+08:00
draft: false
categories:
  - Css
---


### tailwindcss

#### Arbitrary values

在使用tailwind的时候，如果遇到没有预定值的样式，就可以使用arbitrary values。

比如：

* `bg-[#ff8906]` 就可以设置背景色为橙色。
* `w-[300px]` 就可以设置宽度为300px。

还有一些，就是没有定义类型的，最常见的就是svg的属性。`stroke-dasharray`在tailwind中就没有定义类型，所以就可以使用arbitrary values。

* `[stroke-dasharray:450,0]`: 设置stroke-dasharray为450,0。


#### [styling-based-on-parent-state](https://tailwindcss.com/docs/hover-focus-and-other-states#styling-based-on-parent-state)

这种是在子元素依赖于父元素hover等状态时使用的。

```html
<a href="#" class="group block max-w-xs mx-auto rounded-lg p-6 bg-white ring-1 ring-slate-900/5 shadow-lg space-y-3 hover:bg-sky-500 hover:ring-sky-500">
  <div class="flex items-center space-x-3">
    <svg class="h-6 w-6 stroke-sky-500 group-hover:stroke-white" fill="none" viewBox="0 0 24 24"><!-- ... --></svg>
    <h3 class="text-slate-900 group-hover:text-white text-sm font-semibold">New project</h3>
  </div>
  <p class="text-slate-500 group-hover:text-white text-sm">Create a new project from a variety of starting templates.</p>
</a>
```

设置`group`，然后在子元素上使用`group-hover`。


### [stroke-dasharray属性](https://css-tricks.com/almanac/properties/s/stroke-dasharray/)

首先这个是给svg使用的属性。控制stroke的dash长度和间隔长度。

* stroke-dasharray: 2; 表示dash的长度为2，间隔长度为2。
* stroke-dasharray: 2 4; 表示dash的长度为2，间隔长度为4。
* stroke-dasharray: 5 10 15; 表示dash的长度为5，间隔长度为10，dash的长度为15，间隔长度为5。


