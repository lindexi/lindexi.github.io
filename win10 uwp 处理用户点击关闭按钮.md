# win10 uwp 处理用户点击关闭按钮

在 UWP 开发的时候，我做的文档软件需要在文档还没有保存的时候，用户点击关闭按钮的时候告诉用户需要保存。如何在 UWP 阻止用户点击关闭按钮退出软件，如何知道用户点击了关闭按钮

<!--more-->
<!-- CreateTime:2019/7/4 9:28:57 -->

<!-- csdn -->

在 UWP 中有限制的功能，需要在 Package.appxmanifest 中开启，关于限制的功能请看[App capability declarations](https://docs.microsoft.com/en-us/windows/uwp/packaging/app-capability-declarations#restricted-capabilities )
 
拿到用户点击事件需要在 Package.appxmanifest 添加 confirmAppClose 功能

添加方法是点击 Package.appxmanifest 右击点查看代码

找到现有的 Capabilities 元素，默认一个空的 UWP 应用里面会添加网络功能，可以看到下面代码

```csharp
  <Capabilities>
    <Capability Name="internetClient" />
  </Capabilities>
```

在这个元素里面再添加一项，请看代码

```csharp
    <rescap:Capability Name="confirmAppClose" xmlns:rescap="http://schemas.microsoft.com/appx/manifest/foundation/windows10/restrictedcapabilities"/>
```

这里的 rescap 就是限制的意思，需要添加命名空间，添加之后的代码请看下面

```csharp
  <Capabilities>
    <Capability Name="internetClient" />
    <rescap:Capability Name="confirmAppClose" xmlns:rescap="http://schemas.microsoft.com/appx/manifest/foundation/windows10/restrictedcapabilities"/>
  </Capabilities>
```

添加完成之后，就可以在代码里面使用，我在主页面的构造函数监听关闭事件，请看代码

```csharp
            Windows.UI.Core.Preview.SystemNavigationManagerPreview.GetForCurrentView().CloseRequested += MainPage_CloseRequested;
```

在 `MainPage_CloseRequested` 触发的时候就是用户点击关闭按钮，或者用户在任务栏右击关闭应用

在 `MainPage_CloseRequested` 可以让应用延迟关闭或阻止应用关闭，如下面代码，我就不让用户关闭应用

```csharp
        private async void MainPage_CloseRequested(object sender, SystemNavigationCloseRequestedPreviewEventArgs e)
        {
            // 让用户无法关闭
            e.Handled = true;

            var messageDialog = new MessageDialog("欢迎访问 blog.lindexi.com 大量 UWP 博客", "骚年你确定关闭");
            await messageDialog.ShowAsync();
        }
```

只需要通过 `e.Handled = true` 就可以阻止用户关闭应用，就这样写出了一个用户无法快速退出的应用，基本上只能通过任务管理器才能退出

在我的软件里面作为文档软件需要在用户退出的时候发现有没有保存的文档就提示用户保存，也就是需要和上面代码差不多，给出提示。但是在 UWP 中的提示是异步的，也就是异步的提示无法阻止事件的继续，可以看到的是在用户点击关闭的时候，软件的提示页面还没显示，软件就关闭了

在 SystemNavigationCloseRequestedPreviewEventArgs 提供了延迟关闭的方法

在开始显示提示页面之前，调用 `e.GetDeferral()` 方法拿到返回值，在执行完成方法之后调用完成方法，请看代码

```csharp
            var deferral = e.GetDeferral();

            var messageDialog = new MessageDialog("欢迎访问 blog.lindexi.com 大量 UWP 博客", "骚年你确定关闭");
            await messageDialog.ShowAsync();

            deferral.Complete();
```

上面代码的 deferral 可以作为字段保存，这样可以在执行完成更多代码之后才调用完成

本文使用的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/fe9c6c91efbd6a01594a27e7fa2055ba3f8c170c/KemkajardeabalDifeewabaylacurcear ) 欢迎小伙伴访问，如有问题请提 [issus](https://github.com/lindexi/lindexi_gd/issues/new) 或评论

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
