# win10 UWP MessageDialog 和 ContentDialog

我之前开发一个软件 winMarkdown，这个软件在关闭需要提示用户还没有保存东西，需要保存，如果用户选择退出，那么把数据存放。
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

在Metro程序中，没有传统的窗口，当我们要用需要交互的消息提示时，在Win8时代，引入了一个MessageDialog来取代常用的MessageBox。

我在MainPage，挂起`App.Current.Suspending += suspend;`添加通知用户

```csharp
        private async void suspend(object sender, Windows.ApplicationModel.SuspendingEventArgs e)
        {
            SuspendingDeferral deferral = e.SuspendingOperation.GetDeferral();
            MessageDialog message_dialog = new MessageDialog("当前还在运行，确定退出", "退出");
            message_dialog.Commands.Add(new UICommand("确定", cmd => { }, "退出"));
            message_dialog.Commands.Add(new UICommand("取消", cmd => { }));
            message_dialog.DefaultCommandIndex = 0;
            message_dialog.CancelCommandIndex = 1;
            IUICommand result = await message_dialog.ShowAsync();
            if (result.Id as string == "退出")
            {
                
            }
            deferral.Complete();
        }
```

`SuspendingDeferral deferral = e.SuspendingOperation.GetDeferral();`挂起还要做，直到`deferral.Complete();`

```csharp
            MessageDialog message_dialog = new MessageDialog("当前还在运行，确定退出", "退出");
            message_dialog.Commands.Add(new UICommand("确定", cmd => { }, "退出"));
            message_dialog.Commands.Add(new UICommand("取消", cmd => { }));
```

两个按钮，一个确定，一个取消，可以UICommand ID作为点击后，是哪个按钮点击

```csharp
MessageDialog.DefaultCommandIndex按ESC选择按钮
MessageDialog.CancelCommandIndex按enter按钮
```

```csharp
            IUICommand result = await message_dialog.ShowAsync();
            if (result.Id as string == "退出")
            {
                
            }
```

程序要调试挂起，需要生命周期，点击挂起
![这里写图片描述](http://img.blog.csdn.net/20160307210947340)

我们按enter就会点击确定

而我们对于MessageDialog功能还是觉得不够，ContentDialog可以定义复杂的Xaml自定义

我们把MessageDialog换ContentDialog

```csharp
            ContentDialog content_dialog = new ContentDialog()
            {
                Title = "退出",
                Content = "当前还在运行，确定退出",
                PrimaryButtonText = "确定",
                SecondaryButtonText = "取消",
                FullSizeDesired = true,
            };

            content_dialog.PrimaryButtonClick += (_s, _e) => { };

            await content_dialog.ShowAsync();
```

![这里写图片描述](http://img.blog.csdn.net/20160307212834443)

```xml
<UserControl
    x:Class="produproperty.content"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:produproperty"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d"
    d:DesignHeight="300"
    d:DesignWidth="400">

    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition></RowDefinition>
            <RowDefinition></RowDefinition>
        </Grid.RowDefinitions>
        <TextBlock Grid.Row="0" Text="当前还在运行，确定退出"></TextBlock>
        <CheckBox Grid.Row="1" Content="保存"></CheckBox>
    </Grid>
</UserControl>

```

```csharp
            ContentDialog content_dialog = new ContentDialog()
            {
                Title = "退出",
                Content = new content(),
                PrimaryButtonText = "确定",
                SecondaryButtonText = "取消",
                FullSizeDesired = false,
            };

            content_dialog.PrimaryButtonClick += (_s, _e) => { };

            await content_dialog.ShowAsync();
```

![这里写图片描述](http://img.blog.csdn.net/20160307213038399)


参见：
http://www.cnblogs.com/TianFang/p/4857205.html


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。