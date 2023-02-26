# WPF 禁用实时触摸

本文告诉大家如何使用代码禁用 WPF 的触摸消息，解决一些问题，或者准确来说挖出更大的坑

<!--more-->
<!-- CreateTime:2019/11/29 10:20:52 -->

<!-- csdn -->
<!-- 标签：WPF，触摸 -->

在 Win7 时，系统层提供了 WM_TOUCH 这个 Windows 消息，用来通知应用窗口收到触摸消息。而在 Win7 之前，当时还是触摸的蛮荒时代，市面上没有非常好的触摸标准，而且当时的设备性能也没有现在的那么好，触摸又要求实时性。于是在 XP 等时代，就需要使用到 RealTimeStylus 实时触摸技术，用来超快速获取到触摸数据，从而实时响应触摸

在 WPF 里面，默认的触摸走的是 RealTimeStylus 实时触摸。即使在 Win7 到 Win10 或 Win11 都是 RealTimeStylus 实时触摸用来实时响应触摸。如果大家有做高性能笔书写的需求，想要降低系统的触摸层的延迟，那就推荐走 RealTimeStylus 实时触摸获取触摸数据

由于触摸在 Win7 时代差不多刚刚形成标准不久，由于 Win7 的触摸架构没有像 Win10 那样考虑的全面，导致了 RealTimeStylus 实时触摸可能在一些情况下丢失触摸。这时也许可以通过切换为 WM_TOUCH 触摸消息来解决触摸失效问题。当然，这不是万能的，你也可以说，可能 WM_TOUCH 触摸消息偏移了，换成 RealTimeStylus 实时触摸来减少触摸延迟和修复触摸偏移等问题

如果是想切换到 WM_TOUCH 触摸消息，那就需要禁用 WPF 的 RealTimeStylus 实时触摸之后，调用 Win32 的 RegisterTouchWindow 方法对窗口开启触摸消息支持

如果没有禁用 WPF 的 RealTimeStylus 实时触摸，就无法拿到 WM_TOUCH 消息，这是因为两套触摸机制将会打架。在 Windows 系统层发现开启了实时触摸之后，将不会调度 WM_TOUCH 消息给到应用窗口。当然，触摸消息的概念是给窗口级的，只不过在 WPF 里面，一旦开启 RealTimeStylus 实时触摸，在 WPF 框架层将会给每个窗口都开启 RealTimeStylus 实时触摸。如果想要不禁用实时触摸又想收到 WM_TOUCH 消息，可选开一个 WinForms 窗口，详细请看 [WPF 不禁用实时触摸而收到 WM_Touch 触摸消息方法](https://blog.lindexi.com/post/WPF-%E4%B8%8D%E7%A6%81%E7%94%A8%E5%AE%9E%E6%97%B6%E8%A7%A6%E6%91%B8%E8%80%8C%E6%94%B6%E5%88%B0-WM_Touch-%E8%A7%A6%E6%91%B8%E6%B6%88%E6%81%AF%E6%96%B9%E6%B3%95.html )

值得一说的是，如果你使用的是 Win10 或更高版本的系统，我更推荐你开启 WM_Pointer 消息，因为这是 Win10 触摸架构（准确来说 Win8 就有，但是坑多）提供的机制，可以同时用来表示鼠标、触摸等，且几乎没有遇到触摸失效问题，开启之后也不影响 RealTimeStylus 实时触摸，开启方法请看 [win10 支持默认把触摸提升 Pointer 消息](https://blog.lindexi.com/post/win10-%E6%94%AF%E6%8C%81%E9%BB%98%E8%AE%A4%E6%8A%8A%E8%A7%A6%E6%91%B8%E6%8F%90%E5%8D%87-Pointer-%E6%B6%88%E6%81%AF.html )

禁用 WPF 的 RealTimeStylus 实时触摸有多个方法，一个是以下的代码，可以在任意时候在主 UI 线程调用以下代码的 DisableWPFTabletSupport 方法进行禁用 WPF 的 RealTimeStylus 实时触摸。禁用之后无法恢复哦

```csharp
public static void DisableWPFTabletSupport()
{
    // Get a collection of the tablet devices for this window.  
    TabletDeviceCollection devices = System.Windows.Input.Tablet.TabletDevices;

    if (devices.Count > 0)
    {   
        // Get the Type of InputManager.
        Type inputManagerType = typeof(System.Windows.Input.InputManager);
        
        // Call the StylusLogic method on the InputManager.Current instance.
        object stylusLogic = inputManagerType.InvokeMember("StylusLogic",
                    BindingFlags.GetProperty | BindingFlags.Instance | BindingFlags.NonPublic,
                    null, InputManager.Current, null);

        if (stylusLogic != null)
        {
            //  Get the type of the stylusLogic returned from the call to StylusLogic.
            Type stylusLogicType = stylusLogic.GetType();
            
            // Loop until there are no more devices to remove.
            while (devices.Count > 0)
            {
                // Remove the first tablet device in the devices collection.
                stylusLogicType.InvokeMember("OnTabletRemoved",
                        BindingFlags.InvokeMethod | BindingFlags.Instance | BindingFlags.NonPublic,
                        null, stylusLogic, new object[] { (uint)0 });
            }                
        }
               
    }
}
```

以上的代码可以直接在你的项目上使用，可以直接拷贝到你的产品项目上，以上的代码是从微软文档里面复制的

虽然禁用微软提供的触摸事件，可以修复很多坑，但是禁用了也是有很多新的坑，不过我就不在这里告诉大家。自己尝试运行下面代码，然后试试程序。

为什么这样就可以禁用触摸，请看[WPF 触摸到事件](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%88%B0%E4%BA%8B%E4%BB%B6.html )

[Disable the RealTimeStylus for WPF Applications](https://msdn.microsoft.com/en-us/library/dd901337(v=vs.90).aspx )

如果想要 WPF 一启动就禁用触摸，请使用 App.config 添加如下代码

```xml
<configuration>
  <runtime>
    <AppContextSwitchOverrides value="Switch.System.Windows.Input.Stylus.DisableStylusAndTouchSupport=true" />
  </runtime>
</configuration>
```

或者是在 App 构造函数使用 AppContext 设置开关，如以下代码

```csharp
public partial class App : Application
{
    public App()
    {
        AppContext.SetSwitch("Switch.System.Windows.Input.Stylus.DisableStylusAndTouchSupport", true);
    }
}
```

详细请看 [通过 AppSwitch 禁用 WPF 内置的触摸让 WPF 程序可以处理 Windows 触摸消息 - walterlv](https://blog.walterlv.com/post/wpf-disable-stylus-and-touch-support.html )

更多触摸请看 [WPF 触摸相关](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E7%9B%B8%E5%85%B3.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
