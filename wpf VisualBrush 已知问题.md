# wpf VisualBrush 已知问题

本文告诉大家，visualBrush 已知 bug ，希望大家使用 VisualBrush 时可以知道

1. 如果把 VisualBrush 绑定的是在元素加入到视觉树前，那么在元素加入到视觉树之后移除视觉树，VisualBrush 就不会自动刷新

1. 如果把没有加入视觉树的元素加入到 VisualBrush 绑定，之后把元素加入视觉树，再移除，再加入，这时可能 VisualBrush 不再刷新。

1. 如果元素绑定 VisualBrush 然后对元素使用 RenderTargetBitmap 就会让 VisualBrush 无法使用。

解决方法，设置 VisualBrush 的 Visual 为空再设置元素

```csharp
var visual = visualBrush.Visual;
visualBrush.Visual = null;
visualBrush.Visual = visual;
```

参见：https://stackoverflow.com/a/3073378/6116637

https://stackoverflow.com/a/13182210/6116637