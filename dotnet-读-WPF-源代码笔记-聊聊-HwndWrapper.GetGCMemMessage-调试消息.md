
# dotnet 读 WPF 源代码笔记 聊聊 HwndWrapper.GetGCMemMessage 调试消息

我在阅读 WPF 源代码，在 HwndWrapper 的静态构造函数看到了申请了 HwndWrapper.GetGCMemMessage 这个 Windows 消息，好奇这个消息是什么功能的。通过阅读 WPF 源代码和写测试应用，了解到这是一个完全用来内部测试或调试的消息，没有任何业务上的功能

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

在 WPF 的 HwndWrapper 的静态构造函数有以下代码

```csharp
    internal class HwndWrapper : DispatcherObject, IDisposable
    {
        static HwndWrapper()
        {
            s_msgGCMemory = UnsafeNativeMethods.RegisterWindowMessage("HwndWrapper.GetGCMemMessage");
        }

        private static WindowMessage s_msgGCMemory;
    }
```

这个 Windows 消息包含了 GC 字符串，让我以为这是一个和 GC 内存相关的消息。阅读代码才了解到这是一个完全用来调试的消息

唯一的使用是在 HwndWrapper 以下 WndProc 的函数里面。如方法的命名，这个方法就是用来接收所有的 Windows 消息的。而在 WPF 里面，每个 WPF 的 Window 对象都会创建 HwndWrapper 对象，也就是说每个 Window 对象能收到的消息，都会进入此 WndProc 方法里面

```csharp
    private IntPtr WndProc(IntPtr hwnd, int msg, IntPtr wParam, IntPtr lParam, ref bool handled)
    {
            // The default result for messages we handle is 0.
            IntPtr result = IntPtr.Zero;
            WindowMessage message = (WindowMessage)msg;

            // 忽略其他代码

            if (message == s_msgGCMemory)
            {
                // This is a special message we respond to by forcing a GC Collect.  This
                // is used by test apps and such.
                IntPtr lHeap = (IntPtr)GC.GetTotalMemory((wParam == new IntPtr(1) )? true : false);
                result =  lHeap;
                handled = true;
            }

            CheckForCreateWindowFailure(result, true);

            // return our result
            return result;
    }
```

通过以上有删减的代码可以了解到，在收到这个消息时，仅仅只是调用 GC.GetTotalMemory 获取到一个值，用于传入 CheckForCreateWindowFailure 方法里面

如 CheckForCreateWindowFailure 方法的命名，可以看到这是一个判断是否创建窗口失败的方法，方法里面的代码如下

```csharp
        private void CheckForCreateWindowFailure( IntPtr result, bool handled )
        {
            if( ! _isInCreateWindow )
                return;
            
            if( IntPtr.Zero != result )
            {
                System.Diagnostics.Debug.WriteLine("Non-zero WndProc result=" + result);
                if( handled )
                {
                    if( System.Diagnostics.Debugger.IsAttached )
                        System.Diagnostics.Debugger.Break();
                    else
                        throw new InvalidOperationException();
                }
            }
        }
```

先判断 `_isInCreateWindow` 字段。接着判断 result 如果非零且 handled 是 true 将尝试触发断点或炸一下。而显然，调用 GC.GetTotalMemory 一定会返回一个非 0 的值，且 handled 一定被设置为 true 的值。也就是说，如果 `_isInCreateWindow` 被设置为 true 的话，那在调试下将进入断点

继续看看 `_isInCreateWindow` 字段是在哪里使用的

只有在 HwndWrapper 构造时，才会给 `_isInCreateWindow` 字段设置值，有删减的代码如下

```csharp
        public HwndWrapper(...)
        {
        	// 忽略其他代码
            _isInCreateWindow = true;

           try 
           {
                _handle = new SecurityCriticalDataClass<IntPtr>(UnsafeNativeMethods.CreateWindowEx(...));
            }
            finally
            {
                _isInCreateWindow = false;
            }
        	// 忽略其他代码
        }
```

也就是只有在调用 Win32 的 CreateWindowEx 方法过程才会设置 `_isInCreateWindow` 字段。换句话说，基本上能够接收到 `HwndWrapper.GetGCMemMessage` 消息时，此 `_isInCreateWindow` 字段不是 true 的值，也就是说断点是不会进入的

因为一旦`_isInCreateWindow` 字段是 false 的值，那么在 CheckForCreateWindowFailure 方法的以下判断代码里面，将会返回

```csharp
        private void CheckForCreateWindowFailure( IntPtr result, bool handled )
        {
            if( ! _isInCreateWindow )
                return;
            
        	// 忽略其他代码
        }
```

也就是说即使应用程序接收到了 `HwndWrapper.GetGCMemMessage` 消息，也是不会炸掉或进入调试断点。但是会调用 GC.GetTotalMemory 方法，也就是可能可以强行触发一次回收

例如新建一个 WPF 应用，给 MainWindow 不断发送 `HwndWrapper.GetGCMemMessage` 消息，可以在 VS 调试看到不断触发 GC 回收

```csharp
        var windowMessage = PInvoke.RegisterWindowMessage("HwndWrapper.GetGCMemMessage");

        var windowInteropHelper = new WindowInteropHelper(this);
        for (int i = 0; i < 100; i++)
        {
            PInvoke.SendMessage(new HWND(windowInteropHelper.Handle), windowMessage, new WPARAM(1), new LPARAM(0));

            await Task.Delay(100);
        }
```

以上的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/36b22cef80ec382a7467b2b3cbb77f21d68073aa/LaiwurhiroJaqadihawaho) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/36b22cef80ec382a7467b2b3cbb77f21d68073aa/LaiwurhiroJaqadihawaho) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 36b22cef80ec382a7467b2b3cbb77f21d68073aa
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 36b22cef80ec382a7467b2b3cbb77f21d68073aa
```

获取代码之后，进入 LaiwurhiroJaqadihawaho 文件夹




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。