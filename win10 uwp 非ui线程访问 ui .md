# win10 uwp 非ui线程访问 ui 

大家都知道，不可以在 其他线程访问 UI 线程，访问 UI 线程包括给 依赖属性设置值、读取依赖属性、调用方法（如果方法里面修改了依赖属性）等。一旦访问UI线程，那么就会报错，为了解决这个问题，需要使用本文的方法，让后台线程访问 UI 线程。

<!--more-->
<!-- CreateTime:2019/10/12 15:00:12 -->

<!-- csdn -->

本文提供三个方法可以让其他线程访问 UI 线程

第一个方法是比较不推荐使用的，可能出现 [win10 uwp Window.Current.Dispatcher中Current为null](https://blog.lindexi.com/post/win10-uwp-Window.Current.Dispatcher%E4%B8%ADCurrent%E4%B8%BAnull.html)

```csharp
           await Window.Current.Dispatcher.RunAsync(CoreDispatcherPriority.High,
                () =>
                {
                    //需要访问 ui 的代码
                });
```

请注意，这里使用了 CoreDispatcherPriority ，表示优先级，请不要在这里使用 High ，一般都是使用比较低的优先

为何不设置为 High ，参见
[CoreDispatcherPriority](https://docs.microsoft.com/en-us/uwp/api/Windows.UI.Core.CoreDispatcherPriority)

那么比较推荐的一个方法是在一个用户控件或者Page之类的，如果在里面使用了异步线程需要访问 ui 的属性，那么可以使用下面代码

```csharp
await Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal, () => 
{
            //UI code here
});
```

在 UWP 所有的继承依赖属性 [DependencyObject](https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.dependencyobject) 的类，都有 [Dispatcher](https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.dependencyobject.dispatcher) 属性

如果是写在其他类，没有 Dispatcher 属性，那么可以使用下面的代码

```csharp
await CoreApplication.MainView.CoreWindow.Dispatcher.RunAsync(CoreDispatcherPriority.Normal, () => 
{ 
     //代码
});
```

上面两种方法都写在堆栈网 [https://stackoverflow.com/a/38175976/6116637](https://stackoverflow.com/a/38175976/6116637)

[https://stackoverflow.com/questions/7401538/simple-example-of-dispatcherhelper](https://stackoverflow.com/questions/7401538/simple-example-of-dispatcherhelper)

[https://stackoverflow.com/questions/38149767/uwp-update-ui-from-task](https://stackoverflow.com/questions/38149767/uwp-update-ui-from-task)

参见：

[UWP 在非UI线程中更新UI - 星期八再娶你 - 博客园](https://www.cnblogs.com/hupo376787/p/11660732.html#4387513 )

[win10 uwp Window.Current.Dispatcher中Current为null](https://blog.lindexi.com/post/win10-uwp-window.current.dispatcher%E4%B8%ADcurrent%E4%B8%BAnull )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 