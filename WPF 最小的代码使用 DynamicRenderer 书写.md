# WPF 最小的代码使用 DynamicRenderer 书写

在 WPF 中有 DynamicRenderer 提供高性能的书写，这个类在 WPF 只有 InkCanvas 使用，如果想要在自己的 UIElement 使用，需要写一些代码

<!--more-->
<!-- CreateTime:2019/1/9 14:07:26 -->

<!-- csdn -->
<!-- 标签：WPF，笔迹 -->

先创建一个 UIElement 自定义一个，因为 DynamicRenderer 继承于 StylusPlugIn 需要使用 UIElement 的 StylusPlugIns 属性才能添加他

下面创建一个 MeexikelelHaiwurbe 的 UIElement 现在里面什么都没写

```csharp
    public class MeexikelelHaiwurbe : UIElement
    {
    }
```

为了使用 DynamicRenderer 需要支持他的输入层和显示层

## 输入层

对于 StylusPlugIn 需要加入到 UIElement 的 StylusPlugIns 才能收到触摸的消息

这部分的原理比较复杂，请看 [WPF 高速书写 StylusPlugIn 原理](https://lindexi.gitee.io/post/WPF-%E9%AB%98%E9%80%9F%E4%B9%A6%E5%86%99-StylusPlugIn-%E5%8E%9F%E7%90%86.html )

在构造函数添加代码将 DynamicRenderer 添加到 UIElement 的 StylusPlugIns 方法

```csharp
        public MeexikelelHaiwurbe()
        {
            var dynamicRenderer = new DynamicRenderer();
            StylusPlugIns.Add(dynamicRenderer);
        }
```

现在输入层就做好了，如果现在将 MeexikelelHaiwurbe 添加到界面，可以看到没有任何的显示，因为现在还没有将 DynamicRenderer 的显示层添加到视觉树

如果此时可以看到 DynamicRenderer 的 Down 和 Move 函数，可以看到这两个函数几乎没有触发，原因在于附加的元素没被声明自己的宽度和高度，也就是附加的 MeexikelelHaiwurbe 是不可见的

从 [WPF 高速书写 StylusPlugIn 原理](https://lindexi.gitee.io/post/WPF-%E9%AB%98%E9%80%9F%E4%B9%A6%E5%86%99-StylusPlugIn-%E5%8E%9F%E7%90%86.html ) 可以知道，在 StylusPlugIn 要收到触摸的消息，需要附加的元素可以收到消息才可以

所以下面需要设置 MeexikelelHaiwurbe 的宽高

## 设置宽高

在 UIElement 有一个方法是 HitTestCore 设置命中测试，通过这个方法可以判断一个点是否点到了元素上，于是重新这个方法，无论什么点都返回这个元素，于是这个元素就可以做到命中测试，宽度和高度都是最大

当然有层级的关系，不会点到任何的地方都命中这个元素，关于层级请看 WPF 的原理 [WPF 源代码 从零开始写一个 UI 框架](https://lindexi.gitee.io/post/WPF-%E6%BA%90%E4%BB%A3%E7%A0%81-%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E5%86%99%E4%B8%80%E4%B8%AA-UI-%E6%A1%86%E6%9E%B6.html ) 这里面介绍了一个 WPF 框架是如何做的，同时命中测试的原理是什么

```csharp
        protected override HitTestResult HitTestCore(PointHitTestParameters hitTestParameters)
        {
            return new PointHitTestResult(this, hitTestParameters.HitPoint);
        }
```

如果想要一个元素命中测试不可见，就是返回 null 就可以


但是现在还无法显示笔迹，因为没有放在视觉树

## 视觉树

现在一个元素显示在界面需要添加到视觉树，请看代码

```csharp
        private Visual _visual;

        /// <inheritdoc />
        public MeexikelelHaiwurbe()
        {
            var dynamicRenderer = new DynamicRenderer();
            StylusPlugIns.Add(dynamicRenderer);

            _visual = dynamicRenderer.RootVisual;

            AddVisualChild(_visual);
        }

        protected override HitTestResult HitTestCore(PointHitTestParameters hitTestParameters)
        {
            return new PointHitTestResult(this, hitTestParameters.HitPoint);
        }

        /// <inheritdoc />
        protected override Visual GetVisualChild(int index)
        {
            return _visual;
        }

        /// <inheritdoc />
        protected override int VisualChildrenCount => 1;
```

下面是使用 DynamicRenderer 的最小代码

```csharp
    public class MeexikelelHaiwurbe : UIElement
    {
        private Visual _visual;

        /// <inheritdoc />
        public MeexikelelHaiwurbe()
        {
            var dynamicRenderer = new DynamicRenderer();
            StylusPlugIns.Add(dynamicRenderer);

            _visual = dynamicRenderer.RootVisual;

            AddVisualChild(_visual);
        }

        protected override HitTestResult HitTestCore(PointHitTestParameters hitTestParameters)
        {
            return new PointHitTestResult(this, hitTestParameters.HitPoint);
        }

        /// <inheritdoc />
        protected override Visual GetVisualChild(int index)
        {
            return _visual;
        }

        /// <inheritdoc />
        protected override int VisualChildrenCount => 1;
    }
```

[WPF 高性能笔](https://lindexi.gitee.io/post/WPF-%E9%AB%98%E6%80%A7%E8%83%BD%E7%AC%94.html )

[WPF 高速书写 StylusPlugIn 原理](https://lindexi.gitee.io/post/WPF-%E9%AB%98%E9%80%9F%E4%B9%A6%E5%86%99-StylusPlugIn-%E5%8E%9F%E7%90%86.html ) 

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
