# win10 uwp 在笔迹开始书写拿到书写移动事件

在使用 InkCanvas 的过程，无法直接通过 Pointer 消息拿到书写移动，需要使用 StrokeInput 才能获取到

<!--more-->
<!-- CreateTime:2019/7/15 8:58:05 -->

<!-- csdn -->

在 InkCanvas.InkPresenter.StrokeInput 提供了 [StrokeStarted](https://docs.microsoft.com/en-us/uwp/api/windows.ui.input.inking.inkstrokeinput.strokestarted ) 、[StrokeContinued](https://docs.microsoft.com/en-us/uwp/api/windows.ui.input.inking.inkstrokeinput.strokecontinued) 、[StrokeEnded](https://docs.microsoft.com/en-us/uwp/api/windows.ui.input.inking.inkstrokeinput.strokeended) 这些看名字就知道是干什么用的事件，通过这些事件可以在书写的过程拿到触摸事件或鼠标事件

使用 StrokeContinued 作为例子，先在 XAML 界面放一个 InkCanvas 控件，同时修改他的属性名是 InkCanvas 在后台代码可以这样写

```csharp
            InkCanvas.InkPresenter.StrokeInput.StrokeContinued += StrokeContinued;

```

拿到的StrokeContinued是PointerEventArgs可以拿到很多信息

```csharp
        public void StrokeContinued(InkStrokeInput sender, PointerEventArgs args)
        {
           
        }
```

在 Win10 的 [Anniversary Update](https://blogs.windows.com/windowsexperience/2016/08/02/new-video-series-this-week-on-windows-highlights-windows-10-anniversary-update/) 1607 提供了 [CoreWetStrokeUpdateSource](https://docs.microsoft.com/en-us/uwp/api/windows.ui.input.inking.core.corewetstrokeupdatesource ) 可以在湿笔迹的时候触发事件

在 [CoreWetStrokeUpdateSource](https://docs.microsoft.com/en-us/uwp/api/windows.ui.input.inking.core.corewetstrokeupdatesource ) 同样提供了开始书写和书写过程中的事件

先通过 Create 方法创建 CoreWetStrokeUpdateSource 请看代码

```csharp
 var coreWetStrokeUpdateSource = CoreWetStrokeUpdateSource.Create(inkCanvas.InkPresenter);
```

这个事件有一点坑的是有加入的时机问题，请确保在所有的 InkCanvas 包括他的容器都 Loaded 完成之后才可以使用这个事件，不然是不会有触发的

通过 coreWetStrokeUpdateSource 可以拿到触摸按下和移动等事件

```csharp
coreWetStrokeUpdateSource.WetStrokeStarting += CoreWetStrokeUpdateSource_WetStrokeStarting;
coreWetStrokeUpdateSource.WetStrokeContinuing += CoreWetStrokeUpdateSource_WetStrokeContinuing;
coreWetStrokeUpdateSource.WetStrokeStopping += CoreWetStrokeUpdateSource_WetStrokeStopping;
coreWetStrokeUpdateSource.WetStrokeCompleted += CoreWetStrokeUpdateSource_WetStrokeCompleted;
```

我在这些事件里面做输出，这样可以知道在多指触摸下的事件是如何触发的

```csharp
        private void CoreWetStrokeUpdateSource_WetStrokeCompleted(CoreWetStrokeUpdateSource sender,
            CoreWetStrokeUpdateEventArgs args)
        {
            Debug.WriteLine($"CoreWetStrokeUpdateSource_WetStrokeCompleted{args.PointerId}");
        }

        private void CoreWetStrokeUpdateSource_WetStrokeStopping(CoreWetStrokeUpdateSource sender,
            CoreWetStrokeUpdateEventArgs args)
        {
            Debug.WriteLine($"WetStrokeStopping{args.PointerId}");
        }

        private void CoreWetStrokeUpdateSource_WetStrokeContinuing(CoreWetStrokeUpdateSource sender,
            CoreWetStrokeUpdateEventArgs args)
        {
            Debug.WriteLine($"CoreWetStrokeUpdateSource_WetStrokeContinuing {args.PointerId}");
        }

        private void CoreWetStrokeUpdateSource_WetStrokeStarting(CoreWetStrokeUpdateSource sender,
            CoreWetStrokeUpdateEventArgs args)
        {
            Debug.WriteLine($"CoreWetStrokeUpdateSource_WetStrokeStarting{args.PointerId}");
        }
```

运行代码可以看到下面的输出

```csharp
CoreWetStrokeUpdateSource_WetStrokeStarting 1
CoreWetStrokeUpdateSource_WetStrokeContinuing 1
CoreWetStrokeUpdateSource_WetStrokeContinuing 1
……
CoreWetStrokeUpdateSource_WetStrokeStarting 2
CoreWetStrokeUpdateSource_WetStrokeStarting 3
CoreWetStrokeUpdateSource_WetStrokeContinuing 1
CoreWetStrokeUpdateSource_WetStrokeContinuing 2
CoreWetStrokeUpdateSource_WetStrokeContinuing 3
CoreWetStrokeUpdateSource_WetStrokeContinuing 1
CoreWetStrokeUpdateSource_WetStrokeContinuing 2
CoreWetStrokeUpdateSource_WetStrokeContinuing 3
CoreWetStrokeUpdateSource_WetStrokeContinuing 1
CoreWetStrokeUpdateSource_WetStrokeContinuing 2
CoreWetStrokeUpdateSource_WetStrokeContinuing 3
CoreWetStrokeUpdateSource_WetStrokeContinuing 1
CoreWetStrokeUpdateSource_WetStrokeContinuing 2
CoreWetStrokeUpdateSource_WetStrokeContinuing 3
……

CoreWetStrokeUpdateSource_WetStrokeCompleted  1
CoreWetStrokeUpdateSource_WetStrokeContinuing 2
CoreWetStrokeUpdateSource_WetStrokeContinuing 3
CoreWetStrokeUpdateSource_WetStrokeContinuing 2
CoreWetStrokeUpdateSource_WetStrokeContinuing 3
CoreWetStrokeUpdateSource_WetStrokeCompleted2
CoreWetStrokeUpdateSource_WetStrokeContinuing 3
CoreWetStrokeUpdateSource_WetStrokeContinuing 3
CoreWetStrokeUpdateSource_WetStrokeCompleted  3
```

可以从输出看到在多指触摸的时候，对每个手指都会触发一次完整的 Starting-Continuing-Completed 事件

那么 CoreWetStrokeUpdateSource 和 StrokeContinued 的区别在哪？就是在于触发的先后问题，在笔迹开始书写之前先触发 CoreWetStrokeUpdateSource 事件，然后在书写完成之后触发 StrokeContinued 事件，我同时输出 WetStrokeContinuing 和 StrokeContinued 事件，请看下面

```csharp
CoreWetStrokeUpdateSource_WetStrokeContinuing 9
StrokeContinued
CoreWetStrokeUpdateSource_WetStrokeContinuing 9
CoreWetStrokeUpdateSource_WetStrokeContinuing 9
CoreWetStrokeUpdateSource_WetStrokeContinuing 9
StrokeContinued
StrokeContinued
CoreWetStrokeUpdateSource_WetStrokeContinuing 9
CoreWetStrokeUpdateSource_WetStrokeContinuing 9
CoreWetStrokeUpdateSource_WetStrokeContinuing 9
CoreWetStrokeUpdateSource_WetStrokeContinuing 9
StrokeContinued
StrokeContinued
CoreWetStrokeUpdateSource_WetStrokeContinuing 9
CoreWetStrokeUpdateSource_WetStrokeContinuing 9
CoreWetStrokeUpdateSource_WetStrokeContinuing 9
StrokeContinued
StrokeContinued
CoreWetStrokeUpdateSource_WetStrokeContinuing 9
CoreWetStrokeUpdateSource_WetStrokeContinuing 9
CoreWetStrokeUpdateSource_WetStrokeContinuing 9
CoreWetStrokeUpdateSource_WetStrokeContinuing 9
CoreWetStrokeUpdateSource_WetStrokeContinuing 9
CoreWetStrokeUpdateSource_WetStrokeContinuing 9
StrokeContinued
StrokeContinued
```

可以看到 WetStrokeContinuing 的输出时机和频率都会比 StrokeContinued 更高

另外推荐开放的是 StrokeContinued 的事件，因为在使用 UWP 笔迹就是为了做高性能的笔，在 WetStrokeContinuing 的事件里面如果添加了业务代码，那么将会影响笔迹的书写速度

在 UWP 的笔迹书写过程，需要等待 WetStrokeContinuing 事件完成之后才能进行笔迹渲染，而调用 StrokeContinued 事件是在笔迹书写完成之后调用同时不对这个事件做任何的等待

在框架里面建议开放的是 StrokeContinued 触发的事件，减少有逗比开发者在事件里面写了不清真代码降低了笔迹的性能

[背水一战 Windows 10 (61) - 控件（媒体类）: InkCanvas 涂鸦编辑 - webabcd - 博客园](https://www.cnblogs.com/webabcd/p/7242825.html )

[InkStrokeInput Class (Windows.UI.Input.Inking) - Windows UWP applications](https://docs.microsoft.com/en-us/uwp/api/windows.ui.input.inking.inkstrokeinput )

[win10 uwp 通过 win2d 画出笔迹](https://blog.lindexi.com/post/win10-uwp-%E9%80%9A%E8%BF%87-win2d-%E7%94%BB%E5%87%BA%E7%AC%94%E8%BF%B9.html )

[mtaulty/GraphPaperControl: Simple user control that uses Win2D to display a tiled grid design.](https://github.com/mtaulty/GraphPaperControl )

[Creating a Custom Ruler with DirectInk - Windows Developer Blog](https://blogs.windows.com/buildingapps/2016/08/16/creating-a-custom-ruler-with-directink/ )

[Windows-universal-samples/Samples/SimpleInk at master · microsoft/Windows-universal-samples](https://github.com/microsoft/Windows-universal-samples/tree/master/Samples/SimpleInk )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
