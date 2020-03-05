# win10 uwp 重启软件

在16299支持在软件自己重启，不需要让用户点击关闭然后启动，虽然我还不知道这个有什么用。本文告诉大家如何让软件关闭重新打开。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:51 -->


首先需要使用的版本是 16299 ，然后使用 RequestRestartAsync 方法就可以关闭软件重新打开。

下面就是简单的软件

```csharp

    <StackPanel>
        <TextBlock Margin="100" HorizontalAlignment="Center" Text="点击上面的按钮重启   林德熙"></TextBlock>
        <Button HorizontalAlignment="Center" Content="重启" Click="ButtonBase_OnClick"></Button>
    </StackPanel>

        private async void ButtonBase_OnClick(object sender, RoutedEventArgs e)
        {
            await CoreApplication.RequestRestartAsync("退出");
        }
```

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201712151723520171221193314.jpg)

这个方法在 CoreApplication ，使用的时候可以传入参数，在软件打开的时候可以拿到这个参数

请看代码

```csharp
        protected override void OnActivated(IActivatedEventArgs args)
        {
            base.OnActivated(args);
            Debugger.Launch();
            LaunchActivatedEventArgs launchArgs = args as LaunchActivatedEventArgs;
            string str = launchArgs?.Arguments;
          
        }
```

如果需要调试点击按钮重新打开，需要点击周期选择挂起，然后才会重新打开

如果需要重新打开的软件进入调试，可以在 OnLaunched 添加下面代码

```csharp
            Debugger.Launch();

```

添加了这句代码就可以在软件启动进入调试

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018228141535.jpg)

[How to Restart your App Programmatically - Building Apps for WindowsBuilding Apps for Windows](https://blogs.windows.com/buildingapps/2017/07/28/restart-app-programmatically/#WfIzq58ZwbzmwMMd.97 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 