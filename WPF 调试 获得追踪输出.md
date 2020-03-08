# WPF 调试 获得追踪输出

在 WPF 开发中，如果把一个程序发布出去，但是发现有些地方诡异，除了看日志外，如果需要拿到程序实时的输出，可以使用跟踪输出 ，本文告诉大家如何拿到 WPF 的跟踪输出。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:51 -->

<!-- csdn -->

<!-- 标签： WPF，调试 -->

如果有关注项目的宏，就会发现默认在 DEBUG 和 RELEASE 都有一个宏，Trace 这个宏就是用来程序信息跟踪。

因为来讲原理很无聊，还是用一个例子告诉大家这个调试方法是如何使用。

首先需要下载一个软件，通过这个软件可以用来输出。

官网：[DebugView](https://docs.microsoft.com/en-us/sysinternals/downloads/debugview )

[调试日志大师(DebugView的增强版)-CSDN下载](https://download.csdn.net/download/wg_duan/943900 )

下载完成直接打开就好，然后开始创建一个空白的 WPF 程序，在按下按钮的时候添加信息，说按下按钮。

![](http://image.acmx.xyz/lindexi%2F2018516145883346.jpg)

创建的 WPF 使用简单的界面，直接放一个按钮

```csharp
     <Button HorizontalAlignment="Center" Content="点击"
             Click="ButtonBase_OnClick"
             VerticalAlignment="Center"></Button>
```

在后台代码，添加输出

```csharp
        private void ButtonBase_OnClick(object sender, RoutedEventArgs e)
        {
            Trace.WriteLine("德熙点击按钮");
        }
```

这时尝试运行一下代码，点击一下按钮，可以看到 VisualStudio 的输出显示了。

但是如果把这个程序发布出去，千万不要问我这么诡异的程序会有用户，我自己的图床做的那么漂亮都没有用户。用户没有 VisualStudio 那么如何获得刚才的输出？

刚才是不是打开了一个软件，尝试在 VisualStudio 找到刚才 WPF 输出的文件夹，双击打开刚刚的程序，这时点击一下，看看 DebugView 显示什么

![](http://image.acmx.xyz/lindexi%2F201851615428160.jpg)

所以在程序多写一些 Trace ，这样用户说程序很诡异就可以快速使用 DebugView 在用户那里看到程序的输出。

如果发现自己的程序没有输出，那么右击项目属性，看看下面是否取消

![](http://image.acmx.xyz/lindexi%2F201851615734398.jpg)

参见：

[DebugView 调试入门 - CSDN博客](https://blog.csdn.net/jiankunking/article/details/44984487 )

[C# Logging using Trace and DebugView](http://dickvdbrink.github.io/c%23/2015/01/09/CSharp-Logging-using-Trace-and-DebugView.html )

[Viewing WPF Trace Output Outside of Visual Studio](https://wpf.2000things.com/2017/06/29/1212-viewing-wpf-trace-output-outside-of-visual-studio/#comment-61387 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
