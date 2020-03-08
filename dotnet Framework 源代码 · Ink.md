# dotnet Framework 源代码 · Ink

本文是分析 .NET Framework 源代码的系列，主要告诉大家微软做笔迹用的思路，怎么做的笔迹才是高性能的，用户体验比较好的。我会告诉大家源代码的思想，当然这个文章会比较无聊。如果你是想做笔迹的，即使不是 WPF 开发，不是 C# 开发的，也可以看看，因为这个思想是微软的，相对还是比较好的。

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


<!-- 标签：.net Framework，源代码分析，wpf，ink，笔迹,dotnet -->

<div id="toc"></div>

本文开始先让大家简单使用微软的 Ink 试试他是如何做的。

## 使用

通过源代码的方式使用，在 WPF 、UWP 是很简单的，因为现在我不知道怎么去拿 UWP 的源代码，只会使用，所以本文分析的源代码都是 .net Framework 4.7 的，不会说道 UWP 的笔迹。因为 UWP 的笔迹做的比 WPF 好很多，而且下面讲的源代码是在 2011 年写的到现在微软都没有修改。

## 思想

国际惯例，系统调度的单位是线程，如果一个线程做了很多事情，那么这个线程对每个事情做的时间将会比较少。为了做到在用户触摸的时候就显示用户触摸的点，就需要使用一个线程在检查是否有用户输入，画出来。

在 Ink 也是这样，Ink实际上分为两层，一个是动态笔迹，一个是 static 的。那么什么是动态笔迹？实际上在用户触摸的时候，为了立刻画出来，所以用的是一个新的 UI 线程。看到这里是不是觉得有黑科技，是的，UI是可以使用多线程的，请看[WPF 同一窗口内的多线程 UI（VisualTarget） - walterlv](https://walterlv.github.io/post/multi-thread-ui-using-visualtarget-in-wpf.html )

为什么需要在一个新的 UI 线程画出？原因是核心的线程可能需要画很多其它的元素，在用户可以画的时候，如果这时有计时器，他控制界面的元素，那么UI线程就需要处理计时器的内容，而且有很多开发者会在核心线程写一些代码，这些代码都需要时间。微软的笔是给所有开发者用，所以他不能告诉用户，在用的时候不能在核心线程做其他的功能，不然就没法很快画。为了让用户在核心线程做的不会影响到画的，大法就创建了一个新线程，这个线程就只绘制用户输入的点。这一个线程画出的在用户抬手就消失，所以叫动态笔迹。

大家觉得上面这个解释还不对，实际上大法画出的笔如果使用用户画到哪就显示，那么看到来的笔一点也不顺，很粗糙。需要收集很多点做优化，现在我使用的是自己修改的贝塞尔算法，这个算法可以画出很好的笔。但是上面说了动态笔迹是用户检测到摸到屏幕就画，但是收集很多点才可以算出用户的线，可以看到动态笔迹说的就是在显示的时候还支持不停修改，也就是画出的线不是最后显示的线，在画的时候就可以不停修改。

那么static笔迹是什么，实际上我找不到一个比较好的翻译，所以直接使用了部分英文。在用户抬手时，就从收集到的点计算出最后画出的线，而且画出来的线就不会修改了。

## 收集点

在 WPF 可以通过 Stylus 收集按下和移动这些，但是大家也知道，路由事件是需要时间比较长的，可能在 Ink 收到之前，就有其他元素收到，他在收到的做了很多其他的业务，这时就会影响笔的画。为了在用户一按下就开始画，需要用到黑科技。所有的 UIElement 都有 Pulgin ，这个属性可以从 UIElement 拿到原始的触摸，这样可以比路由事件更快拿到用户按下。从这里拿到的触摸可能是在其他线程。

## 如何画出 Stroke

从上面收集到点，从点转换为 StylusPoint 加入 StylusPointCollection 然后创建 Stroke ，把 Stroke 转换为 Geometry ，通过 DrawingVisual 画出来。

```csharp
                var collection = new StylusPointCollection(point.Description) {point};
                var stroke = new Stroke(collection) {DrawingAttributes = _drawingAttributes};

                var brush = new SolidColorBrush(_drawingAttributes.Color);
                dc.DrawGeometry(brush, null, stroke.GetGeometry(_drawingAttributes));
```

上面的代码需要添加 `_drawingAttributes` 字段，同时使用 DrawingVisual 的方式画出笔迹

通过 stroke 这个类可以添加点的方式，可以自己模拟调试笔迹

在底层的 GetGeometry 函数是用到 StrokeRenderer 的 CalcGeometryAndBounds 函数

源代码：[StrokeRenderer.cs](https://referencesource.microsoft.com/#PresentationCore/Core/CSharp/MS/Internal/Ink/StrokeRenderer.cs,1670af750bda3057 )

## StylusPlugIns 

如果需要做出[高性能的笔](https://lindexi.gitee.io/post/WPF-%E9%AB%98%E6%80%A7%E8%83%BD%E7%AC%94.html) 那么就需要了解 [StylusPlugIns](https://docs.microsoft.com/en-us/dotnet/api/system.windows.input.stylusplugins?view=netframework-4.7.1)

在 Ink 底层的动态笔迹就是使用这个技术。

那么从 StylusPlugIns 可以拿到什么？实际上在[高性能的笔](https://lindexi.gitee.io/post/WPF-%E9%AB%98%E6%80%A7%E8%83%BD%E7%AC%94.html)已经有告诉大家了，不过这里对比一下两者的不同。

下面来对比从StylusPlugIns拿到的按下和抬起与从元素直接拿到的对比，前面的数字是开机到现在的毫秒数，可以从下面的图知道 StylusPlugIns 是比 从元素拿到数据要快，而且在路由事件比较复杂的时候，这个值会更大。

在收到抬起的时候，可以看到 StylusPlugIns 拿到的时间比元素收到的快 15 毫秒，在笔迹里，如果能快 15 毫秒是给用户体验很好。

![](http://image.acmx.xyz/65fb6078-c169-4ce3-cdd9-e35752d07be0%2F201832281435.jpg)

而且在移动的时候，可以看到 StylusPlugIns 收到移动的次数比从元素拿到的多，所以可以画出更加好看的线。

实际上上面的测试代码很简单，大家也可以自己写出来。

## 动态笔迹

在 WPF 的 Ink 的源代码可以看到 InkCanvas 使用 DynamicRenderer 作为动态笔迹层。动态笔迹层是什么？动态笔迹是相对静态笔迹的，动态笔迹就是用户在书写的过程，需要按照用户动态加入的点，动态渲染出来笔迹。这一层对笔迹性能要求最高，所以需要通过最快的方式拿到触摸，通过最快的方式画出来。

在 DynamicRenderer 这个类，继承 StylusPlugIns 可以很快拿到触摸，同时在
[另一个线程创建 UI 线程](https://lindexi.gitee.io/post/WPF-%E8%B7%A8%E7%BA%BF%E7%A8%8B-UI-%E7%9A%84%E6%96%B9%E6%B3%95.html )，这样就可以在主线程卡住的时候继续。

在动态笔迹层这个另一个UI线程叫 DynamicRenderer 线程，这部分的代码是在 DynamicRendererThreadManager 管理创建

在动态笔迹层收集到触摸的事件的时候，通过 rawStylusInput.GetStylusPoints() 方法解析底层报告的触摸点。

这里拿到的触摸点已经是被系统优化的，可以拿来直接用，而不需要做贝塞尔。

在 WPF 的代码，在动态笔迹层拿到了点之后，不是直接传送到另一个 UI 线程。

将收集到的点直接调用 StrokeRenderer 的方法然后通过 DrawingVisual 画出，再将 DrawingVisual 添加到 ContainerVisual 里面加入视觉树的过程，其中通过 StrokeRenderer 的方法可以拿到可以在 DrawingVisual 画出的元素。

在底层返回的是一个 StreamGeometry 通过 dispatcher 调到另一个 UI 线程，在另一个 UI 线程画出来，同时加入到视觉树。

这样做可以做到在收集到点的时候快速画出来，但是不足在于会让 Visual 的数量太多

在底层可以看到这个动态笔迹类只能支持一个手指，如果想要支持多个触摸就需要添加多个动态笔迹层。

## 转静态

在书写完成之后，可以通过路由事件在主线程收到 Up 的消息，判断当前已经有一个笔迹可以收集

在动态笔迹书写的时候，主线程也通过路由事件收集到触摸的信息，于是在判断有一个笔迹可以转静态的时候，主线程就创建一个 Stroke 将主线程收集到的触摸转换。

这时动态笔迹就会执行 NotifyAppOfDRThreadRenderComplete 方法，清理视觉树上动态笔迹的内容。

因为动态笔迹不知道静态笔迹什么时候才渲染完，在动态笔迹使用了 MediaContext 的 RenderComplete 事件判断是否渲染完成 

看到这里会问一下为什么动态笔迹的模块会不知道主线程的笔迹绘制完成？这里有两个原因，第一个是模块的耦合的原因。动态笔迹是笔迹绘制模块，这个模块处理的是从触摸线程拿到触摸数据，然后快速绘制在屏幕。而主线程做的有很多业务逻辑，在用户触摸到屏幕的时候收到事件，按照路由事件判断用户触摸的是那个元素做不同的业务，此时就不能让动态笔迹模块监听主线程的事件和回调，因为完全在不知道主线程会使用哪个模块进行转换笔迹。在触摸的时候，主线程也可以收到触摸事件，但此时主线程不做笔迹绘制，而是让动态笔迹层绘制，只有在抬手的时候才在主线程创建笔迹元素。这里关键的在于什么时候绘制在动态笔迹层的笔迹消失，什么时候绘制在主线程的笔迹显示的问题，这里就是 WPF 笔迹模块的一个核心。因为这就是第二个原因，在 WPF 主线程和渲染线程分开，而笔迹线程和渲染线程分开，也就是笔迹线程告诉渲染线程动态笔迹层的笔迹要消息，到渲染线程从屏幕移除动态笔迹层是有延迟，而主线程从告诉渲染线程绘制笔迹到渲染线程将笔迹绘制到屏幕也是有延迟，所以动态笔迹模块是不知道他在什么时候告诉渲染线程移除动态笔迹就刚好渲染线程将动态笔迹层的笔迹移除然后将主线程的笔迹画在屏幕上。为了解决这样的问题，在 WPF 里面用了很多黑科技，详细请大家看源代码，关注动态笔迹层的 NotifyAppOfDRThreadRenderComplete 方法

参见：[Viewports and content (Windows)](https://msdn.microsoft.com/en-us/library/windows/desktop/dn423906(v=vs.85).aspx )

[Ink input (Windows)](https://msdn.microsoft.com/en-us/library/windows/desktop/mt592874(v=vs.85).aspx )

[Ink input (Windows)](https://msdn.microsoft.com/en-us/library/windows/desktop/mt592874(v=vs.85).aspx )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
