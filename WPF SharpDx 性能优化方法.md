# WPF SharpDx 性能优化方法

本文记录一些通用的 SharpDx 的性能优化方法

<!--more-->
<!-- CreateTime:2020/7/30 14:59:13 -->



本文属于 [SharpDx 系列](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-SharpDx-%E6%B8%B2%E6%9F%93%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html) 博客，建议从头开始读

## 不要监听 ContentRendered 事件

监听了 Window 的 ContentRendered 事件将会降低整个 WPF 的渲染性能

## 关注硬件渲染

注意是否在 WPF 开启了硬件渲染，详细请看 [WPF 渲染级别](https://blog.lindexi.com/post/WPF-%E6%B8%B2%E6%9F%93%E7%BA%A7%E5%88%AB.html) 和 [WPF 设置纯软件渲染](https://blog.csdn.net/lindexi_gd/article/details/102831135 )

## 执行业务代码的耗时

在执行 SharpDx 的指导渲染逻辑，也就是告诉 SharpDx 应该画点还是画线的逻辑，在这个逻辑里面耦合了业务逻辑，业务逻辑本身的耗时将会让 SharpDx 的收集绘制指令的性能降低

```csharp
_renderTarget.BeginDraw();

// 执行耗时的业务代码

_renderTarget.DrawEllipse(ellipse, brush, 1);

_renderTarget.EndDraw();
```

因此优先解决这部分业务代码，可选方案是让这部分代码先执行，执行完成之后再执行 SharpDx 的绘制逻辑

另一个方法就是让业务代码在另一个线程执行

这部分和具体业务相关

## 减少绘制数量

尽管使用 SharpDx 的绘制效率很高，但是假定需要执行的绘制命令特别多，此时也会降低性能，因此我的一个性能比较强的应用就预先计算出某些命令不会在界面可见，这部分就不参与渲染

如以下代码，这里的代码将会很多次的获取椭圆渲染

```csharp
_renderTarget.BeginDraw();

for (int i = 0; i < 1000000; i++)
{
	var ellipse = GetEllipse(i);
	_renderTarget.DrawEllipse(ellipse, brush, 1);
}

_renderTarget.EndDraw();
```

而此时如果有一个方法可以判断某些矩形在界面是不可见的，如超过画面，那么过滤掉这部分命令，可以提升很多性能

```csharp
   _renderTarget.BeginDraw();

   for (int i = 0; i < 1000000; i++)
   {
       var ellipse = GetEllipse(i);
       if (IsVisible(ellipse))
       {
           _renderTarget.DrawEllipse(ellipse, brush, 1);
       }
   }

   _renderTarget.EndDraw();
```

减少渲染命令是优化最强的方法

## 减少 Geometry 的绘制

在所有基础绘制命令，绘制 Geometry 是最吃显卡的，因此如果能使用基础图形，如线条或矩形等代替就不要使用 Geometry 绘制

## 图片优先 jpg 图片

大部分的显卡对于绘制 jpg 图片都有优化，可以认为 jpg 图片的渲染性能比较好。但是这不是说 jpg 的比 png 的好，因为影响图片的渲染性能有很多，如图片 dpi 和图片大小等

假设某些图片不关注透明等，同时这些图片是可以预先制作的，那么优先选 jpg 格式


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
