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

### `didChangeDependencies`什么时候触发

1. 在`initState`后立刻调用
2. 依赖发生变化。一般是指`updateShouldNotify`返回为true的时候。比如使用`Provider`机制。
3. `didChangeDependencies`可能会触发多次，`initState`只会有1次。

### 颜色

* The primary key color will be used for the source color, much like the source color in the dynamic setting and will override all other key colors, so set this one first. This means, there is no need to add additional colors.

* Secondary roles are used for less prominent components in the UI, while expanding the opportunity for color expression.

* Tertiary roles are used for contrasting accents that can be used to balance primary and secondary colors or bring heightened attention to an element. 

* Neutral roles are used for surfaces and backgrounds, as well as high emphasis text and icons.Your brand colors will now be included in the core color scheme adjusted to match the M3 color space, fully accessible, and able to export and implement within code as a theme.

### 参考资料

* [Set the space between Elements in Row Flutter](https://stackoverflow.com/questions/53141752/set-the-space-between-elements-in-row-flutter)


### 禁用平台创建

当使用flutter create project_name`命令启动新的Flutter项目时，您可能会发现自己需要手动删除某些平台文件夹。

```bash
flutter config --no-enable-linux-desktop  --no-enable-macos-desktop --no-enable-web  --no-enable-windows-desktop
```