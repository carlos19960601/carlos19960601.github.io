---
title: 'Flutter学习笔记'
date: 2024-10-22T15:49:55+08:00
draft: true
---

### Row和Column在组件之间添加间距

```dart
Wrap(
  spacing: 100, // set spacing here
  children: <Widget>[
    Text("1"),
    Text("2"),
  ],
)
```

### AppBar滑动时背景透明


### 检测当前组件的滑动方向

```dart
final scrollableState = Scrollable.maybeOf(context);
final AxisDirection? axisDirection = scrollableState?.axisDirection;
```


### 参考资料

* [Set the space between Elements in Row Flutter](https://stackoverflow.com/questions/53141752/set-the-space-between-elements-in-row-flutter)