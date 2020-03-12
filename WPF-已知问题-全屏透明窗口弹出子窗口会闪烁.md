
# WPF 已知问题 全屏透明窗口弹出子窗口会闪烁

在 WPF 中通过设置 WindowStyle 为 None 以及 WindowState 为 Maximized 进入全屏，同时设置 AllowsTransparency 支持透明，此时弹出一个设置 WindowStyle 是 None 的子窗口，用 VisualStudio 2019 运行将会看到 子窗口 先显示出来，然后回到主窗口下面，然后再显示到主窗口上面

<!--more-->


<!-- 发布 -->

其实此问题我没有复现

此问题步骤十分简单，但是有要求是在 VisualStudio 2019 附加的基础上，如果单独运行预计没有此问题。此问题在 .NET Framework 4.5-4.8 以及 .NET Core 3.1 都复现

步骤：

1. 给 MainWindows 如下设置

```xml
WindowStyle="None" AllowsTransparency="True"
        WindowState="Maximized" 
```

2. 给主窗口添加一个按钮

```xml
        <Button Content="Show sub window" HorizontalAlignment="Center" VerticalAlignment="Center" Click="Button_OnClick"/>
```

3. 点击按钮显示一个子窗口

```csharp
        private void Button_OnClick(object sender, RoutedEventArgs e)
        {
            var window = new Window()
            {
            };

            window.Show();
        }
```

通过 VisualStudio 2019 运行项目，可以看到在点击按钮的时候，先显示了子窗口然后子窗口到主窗口下方，等一下又回到主窗口上方。也就是子窗口显示一下然后不显示，可以看到出现闪烁

注意，此时如果没有在 VisualStudio 2019 附加调试，那么不会看到子窗口闪烁

解决方法有两个

第一个方法是去掉主窗口的 AllowsTransparency 属性

第二个方法是设置子窗口的 Owner 为主窗口

```csharp
        private void Button_OnClick(object sender, RoutedEventArgs e)
        {
            var window = new Window()
            {
                Owner = this
            };

            window.Show();
        }
```

本文代码放在 [github](https://github.com/dotnet-campus/wpf-issues/tree/4ab8bafcb1ed67a479b988aefd2bbd6f11198ec5/ChildWindowDisplayedAndThenBackToMainWindow) 欢迎小伙伴访问





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。