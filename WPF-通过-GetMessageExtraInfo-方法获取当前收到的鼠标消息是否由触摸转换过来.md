
# WPF 通过 GetMessageExtraInfo 方法获取当前收到的鼠标消息是否由触摸转换过来

本文将告诉大家如何在 WPF 或者其他 Win32 应用里面，在收到鼠标消息时，通过 GetMessageExtraInfo 方法获取当前收到的鼠标消息是否由触摸消息提升而来

<!--more-->


<!-- CreateTime:2023/11/22 17:06:03 -->
<!-- 发布 -->
<!-- 博客 -->

大家都知道，在不开启 WM_Pointer 的情况下，无论是走 WM_Touch 或者是 RealTimeStylus 等方式，默认下触摸都会提升为鼠标消息从而更好兼容应用程序的逻辑

如果此时应用程序想要根据消息循环里面接收到的 Win32 消息判断一个鼠标消息的来源是否来自于触摸框触摸屏或者是 Pen 笔等，可以通过 GetMessageExtraInfo 方法获取更多的信息

根据 GetMessageExtraInfo 方法获取到的 LPARAM 进行 Mask 一下 0xFFFFFF80 值，即可通过返回的结果判断鼠标消息的来源，如返回的结果是 0xFF515780 则判断是 Touch 触摸消息过来的，通过返回结果是 0xFF515700 则判断是 Pen 笔过来的

演示的代码如下

```csharp
    private IntPtr Hook(IntPtr hwnd, int msg, IntPtr wparam, IntPtr lparam, ref bool handled)
    {
        const int WM_LBUTTONDOWN = 0x0201;
        if (msg == WM_LBUTTONDOWN)
        {
            var messageExtraInfo = PInvoke.GetMessageExtraInfo();
            var value = messageExtraInfo.Value.ToInt64();
            var mask = 0xFFFFFF80; // MOUSEEVENTF_FROMTOUCH
            var result = value & mask;

            if (result == 0xFF515780)
            {
                // 这是 Touch 过来
            }
            else if (result == 0xFF515700)
            {
                // 收到 Pen 的
            }
            else if (value == 0)
            {
                // 这是鼠标
            }
        }

        return IntPtr.Zero;
    }
```

通过以上代码即可了解当前收到的鼠标消息是否从触摸或笔消息提升的，还是由真正的鼠标创建

特别感谢 許煜坤-台灣微軟研究開發處 的大佬提供了这个方法

本文以上的可调试代码放在[github](https://github.com/lindexi/lindexi_gd/tree/3c2d3fc41f0bca74e1c15be5d732138e0b958497/WegairhokawhelnaHibairdercawwe) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/3c2d3fc41f0bca74e1c15be5d732138e0b958497/WegairhokawhelnaHibairdercawwe) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 3c2d3fc41f0bca74e1c15be5d732138e0b958497
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 3c2d3fc41f0bca74e1c15be5d732138e0b958497
```

获取代码之后，进入 WegairhokawhelnaHibairdercawwe 文件夹

更多请参阅 [应用程序疑难解答 - Win32 apps - Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/wintouch/troubleshooting-applications )

更多触摸和笔迹书写相关请看 [WPF 触摸相关](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E7%9B%B8%E5%85%B3.html)




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。