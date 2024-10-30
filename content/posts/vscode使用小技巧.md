+++
title = 'Vscode使用小技巧'
date = 2024-10-30T18:00:36+08:00
draft = false
+++

### 标记重要的代码点

在阅读代码的时候，有一些关键代码(比如核心逻辑、代码入口等)需要经常找，所以我想要够标记一下，这样就能快速定位。

可以借助`TODO Tree`插件标记todo一样。自定义一个`IMPORTANT`

```json
"todo-tree.highlights.customHighlight": {
    ...
    "IMPORTANT": {
      "background": "#ff000080"
    }
  },
```

使用的时候像这样就能够在`TODO tree`里面看到了。

```go
// IMPORTANT: 本地流量入口
listener.ReCreateHTTP(general.Port, tunnel.Tunnel)
```

### 光标移动更加顺滑

```json
{
  "editor.smoothScrolling": true,
  "editor.cursorBlinking": "smooth",
}
```