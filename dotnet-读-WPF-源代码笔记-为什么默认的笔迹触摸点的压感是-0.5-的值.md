
# dotnet 读 WPF 源代码笔记 为什么默认的笔迹触摸点的压感是 0.5 的值

本文是我在读 WPF 源代码做的笔记

<!--more-->



<!-- 标签：WPF，WPF源代码 -->
<!-- 发布 -->

在 WPF 中，如 [WPF 触摸到事件](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%88%B0%E4%BA%8B%E4%BB%B6.html ) 博客内容，在 WPF 中将会通过 StylusPointCollection 传入原始的触摸数据，也就是 `int[] rawPacketData` 来创建触摸点

在 StylusPointCollection 的构造函数里面，将会读取 `int[] rawPacketData` 的内容，拿到触摸点

```csharp
        internal StylusPointCollection(StylusPointDescription stylusPointDescription, int[] rawPacketData, GeneralTransform tabletToView, Matrix tabletToViewMatrix)
```

在创建 StylusPoint 时，使用下面代码

```csharp
    StylusPoint newPoint = new StylusPoint(p.X, p.Y, StylusPoint.DefaultPressure, _stylusPointDescription, data, false, false);
```

这里的 StylusPoint.DefaultPressure 如下代码

```csharp
    public struct StylusPoint
    {
        internal static readonly float DefaultPressure = 0.5f;
    }
```

这就是为什么触摸的默认压感是 0.5 的原因

而如果是是通过带压感的笔的触摸点，那么在 StylusPointDescription 描述里面，将会 ContainsTruePressure 属性表示当前触摸点是否存在压感，此时在创建完成触摸点之后，会使用如下逻辑，读取压感设置给触摸点

```csharp
 bool containsTruePressure = stylusPointDescription.ContainsTruePressure;

 StylusPoint newPoint = new StylusPoint(p.X, p.Y, StylusPoint.DefaultPressure, _stylusPointDescription, data, false, false);
 if (containsTruePressure)
 {
     //use the algoritm to set pressure in StylusPoint
     int pressure = rawPacketData[i + 2];
     newPoint.SetPropertyValue(StylusPointProperties.NormalPressure, pressure);
 }
```

当前的 WPF 在 [https://github.com/dotnet/wpf](https://github.com/dotnet/wpf) 完全开源，使用友好的 MIT 协议，意味着允许任何人任何组织和企业任意处置，包括使用，复制，修改，合并，发表，分发，再授权，或者销售。在仓库里面包含了完全的构建逻辑，只需要本地的网络足够好（因为需要下载一堆构建工具），即可进行本地构建





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。