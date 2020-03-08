# WPF 解决弹出模态窗口关闭后，主窗口不在最前

本文告诉大家如何解决这个问题，在 WPF 的软件，弹出一个模态窗口。使用另一个窗口在模态窗口前面。从任务栏打开模态窗口。关闭模态窗口。这时发现，主窗口会在刚才使用的另一个窗口下面。

<!--more-->
<!-- CreateTime:2019/6/23 11:48:38 -->


这是 Windows 的bug ，不过从上面的描述，也许大家还不知道这个问题是什么。不过我把他放在 github ，所以大家可以看到这个问题具体是什么。

可以运行代码：[wpf-issues/ChildWindows ](https://github.com/dotnet-campus/wpf-issues/tree/master/ChildWindows )

在上面的网站有详细的视频告诉大家是如何做的就可以看到这个问题。

因为模态窗口会在关闭的时候，让主窗口不在最前，所以团队不敢使用模态窗口。

但是这个问题在看了 Windows 历史之后，才发现这个问题是 Windows 的问题。在窗口关闭的时候，Windows 会找一个在这个窗口下方的第一个可用的窗口，激活他。因为弹出模态窗口的主窗口是被禁用的。所以在模态窗口关闭的时候，就忽略了主窗口可以激活，于是找到主窗口下方的一个可以被激活的窗口，这时激活他，于是这个被找到的窗口就在主窗口的上面。

那么这个问题可以如何解决？实际上只需要在模态窗口关闭之前，激活主窗口就可以。请看下面的代码

```csharp
     private void ButtonBase_OnClick(object sender, RoutedEventArgs e)
        {
            var w = new Window
            {
                Title = "ChildWindow",
                Owner = this,
                Width = 600,
                Height = 500,
                Content = "Step 1: Move a window of another appliation to over the MainWindow\r\n" +
                          "Step 2: Active this window from the Windows taskbar\r\n" +
                          "Step 3: Close this window (ChildWindow)\r\n" +
                          "\r\nNow you can see that the MainWindow drops down and coverd by another window.\r\n" +
                          "If you don't understand the description above, please refer to the video demo in this project."
            };
            w.Closing += W_Closed;
            w.Show();
        }

        private void W_Closed(object sender, CancelEventArgs e)
        {
            Activate();
        }
```

解决的项目我传到 [CSDN](http://download.csdn.net/download/lindexi_gd/10243218 )，可以直接运行。

参见 [关闭模态窗口后，父窗口居然跑到了其他窗口的后面](https://walterlv.gitee.io/post/fix-owner-window-dropping-down-when-close-a-modal-child-window.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
