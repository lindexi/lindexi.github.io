# dotnet 读 WPF 源代码笔记 Stroke 类可能存在的内存泄露

在 WPF 中，使用 Stroke 类时，可能会出现内存泄露，原因是 DrawingAttributes 的事件被监听没有释放。本文将从源代码的角度告诉大家这个内存泄露问题和如何解决

<!--more-->
<!-- CreateTime:2021/1/28 19:34:45 -->


<!-- 标签：WPF，WPF源代码 -->
<!-- 发布 -->

在满足如下条件的时候，将会让 Stroke 类出现内存泄露

1. 存在一个 Stroke 被强引用，将这个 Stroke 记为 A 对象
2. 取 A 对象的 DrawingAttributes 属性，创建出另一个新的 Stroke 对象，将这个对象记为 B 对象

此时将会发现 B 对象不会被释放，如 [demo](https://github.com/lindexi/lindexi_gd/tree/3214ecc7/KemjawyecawDurbahelal ) 所示，点击按钮可以看到内存不会释放

实现上面条件的代码很简单，请看代码

```csharp
      var stroke = new Stroke(new StylusPointCollection(stylusPointList), StrokeVisual.Stroke.DrawingAttributes);
```

上面代码的 StrokeVisual 是一个窗口的属性，也就是说 StrokeVisual.Stroke 是存在强引用。此时创建出来的 stroke 对象将不会被回收

按钮点击的代码如下

```csharp
        public StrokeVisual StrokeVisual { get; set; }

        private void Button_OnClick(object sender, RoutedEventArgs e)
        {
            for (int i = 0; i < 100; i++)
            {
                var stylusPointList = new List<StylusPoint>();
                for (int j = 0; j < 1000; j++)
                {
                    stylusPointList.Add(new StylusPoint(i, i));
                }

                var stroke = new Stroke(new StylusPointCollection(stylusPointList), StrokeVisual.Stroke.DrawingAttributes);
            }
        }
```

关于 StrokeVisual 的定义，请看 [WPF 最简逻辑实现多指顺滑的笔迹书写](https://blog.lindexi.com/post/WPF-%E6%9C%80%E7%AE%80%E9%80%BB%E8%BE%91%E5%AE%9E%E7%8E%B0%E5%A4%9A%E6%8C%87%E9%A1%BA%E6%BB%91%E7%9A%84%E7%AC%94%E8%BF%B9%E4%B9%A6%E5%86%99.html )

那为什么使用一个被强引用的 Stroke 的 DrawingAttributes 去创建另一个 Stroke 对象，会让另一个 Stroke 不会被释放

通过 WPF 的源代码可以看到，在 Stroke 里面是将 DrawingAttributes 作为属性存放，因此 Stroke 强引用 DrawingAttributes 对象。如果使用被强引用的 Stroke 的 DrawingAttributes 去创建另一个 Stroke 对象，因为在 Stroke 对象的构造函数里面有如下代码

```csharp
internal Stroke(StylusPointCollection stylusPoints, DrawingAttributes drawingAttributes, ExtendedPropertyCollection extendedProperties)
{
    _drawingAttributes = drawingAttributes;
    _drawingAttributes.AttributeChanged += new PropertyDataChangedEventHandler(DrawingAttributes_Changed);
}
```

也就说在 Stroke 里面添加了 `_drawingAttributes` 的事件，也就说 Stroke 也被 DrawingAttributes 强引用。而如上面描述，这里的 DrawingAttributes 被另一个 Stroke 强引用。因此如果多个 Stroke 使用相同的一个 DrawingAttributes 对象，有一个 Stroke 被强引用，那么其他的所有的 Stroke 对象也不会被释放

本文的测试代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/3214ecc7/KemjawyecawDurbahelal ) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/3214ecc7/KemjawyecawDurbahelal ) 欢迎下来进行调试

那如何解决此问题？在 DrawingAttributes 对象里面提供了 Clone 方法，在使用某个 Stroke 的 DrawingAttributes 对象创建一个新的 Stroke 的时候，如果要解决本文提到的的坑，可以调用 DrawingAttributes 的 Clone 方法创建一个新的 DrawingAttributes 对象，此时这个新的 DrawingAttributes 对象将不会被原有的 Stroke 强引用，因此也就不会让新创建的 Stroke 因为被 DrawingAttributes 强引用的原因内存泄露

```csharp
  var stroke = new Stroke(new StylusPointCollection(stylusPointList), StrokeVisual.Stroke.DrawingAttributes.Clone());
```

如上面代码，通过在新建 Stroke 的时候，传入的 DrawingAttributes 是调用 Clone 方法创建的

这个问题报告给了 WPF 官方，请看 [WPF Stroke may memory leak](https://github.com/dotnet/wpf/issues/4100 )

当然了，这个也不算是坑，通过 VisualStudio 进行内存调试，是可以找到这个坑的

其实通过阅读源代码，如果给多个 Stroke 对象使用相同的一个 StylusPointCollection 对象，那么也有相同的坑。此时只要有一个 Stroke 被强引用了，那么所有的 Stroke 都不会释放

```csharp
internal Stroke(StylusPointCollection stylusPoints, DrawingAttributes drawingAttributes, ExtendedPropertyCollection extendedProperties)
{
    _drawingAttributes = drawingAttributes;
    _drawingAttributes.AttributeChanged += new PropertyDataChangedEventHandler(DrawingAttributes_Changed);

    _stylusPoints = stylusPoints;
    _stylusPoints.Changed += new EventHandler(StylusPoints_Changed);
    _stylusPoints.CountGoingToZero += new CancelEventHandler(StylusPoints_CountGoingToZero);
}
```

当前的 WPF 在 [https://github.com/dotnet/wpf](https://github.com/dotnet/wpf) 完全开源，使用友好的 MIT 协议，意味着允许任何人任何组织和企业任意处置，包括使用，复制，修改，合并，发表，分发，再授权，或者销售。在仓库里面包含了完全的构建逻辑，只需要本地的网络足够好（因为需要下载一堆构建工具），即可进行本地构建

更多笔迹相关请看

- [WPF 渲染原理](https://lindexi.gitee.io/post/WPF-%E6%B8%B2%E6%9F%93%E5%8E%9F%E7%90%86.html )
- [高性能笔迹原理](https://blog.lindexi.com/post/%E9%AB%98%E6%80%A7%E8%83%BD%E7%AC%94%E8%BF%B9%E5%8E%9F%E7%90%86.html)
- [WPF 高性能笔](https://blog.lindexi.com/post/WPF-%E9%AB%98%E6%80%A7%E8%83%BD%E7%AC%94.html ) 
- [WPF 高速书写 StylusPlugIn 原理](https://blog.lindexi.com/post/WPF-%E9%AB%98%E9%80%9F%E4%B9%A6%E5%86%99-StylusPlugIn-%E5%8E%9F%E7%90%86.html )
- [WPF 最小的代码使用 DynamicRenderer 书写](https://blog.lindexi.com/post/WPF-%E6%9C%80%E5%B0%8F%E7%9A%84%E4%BB%A3%E7%A0%81%E4%BD%BF%E7%94%A8-DynamicRenderer-%E4%B9%A6%E5%86%99.html )
- [WPF 使用 Composition API 做高性能渲染](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Composition-API-%E5%81%9A%E9%AB%98%E6%80%A7%E8%83%BD%E6%B8%B2%E6%9F%93.html )
- [WPF 使用 Win2d 渲染](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Win2d-%E6%B8%B2%E6%9F%93.html )
- [win10 uwp win2d CanvasVirtualControl 与 CanvasAnimatedControl](https://blog.lindexi.com/post/win10-uwp-win2d-CanvasVirtualControl-%E4%B8%8E-CanvasAnimatedControl.html )
- [WPF 最简逻辑实现多指顺滑的笔迹书写](https://blog.lindexi.com/post/WPF-%E6%9C%80%E7%AE%80%E9%80%BB%E8%BE%91%E5%AE%9E%E7%8E%B0%E5%A4%9A%E6%8C%87%E9%A1%BA%E6%BB%91%E7%9A%84%E7%AC%94%E8%BF%B9%E4%B9%A6%E5%86%99.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
