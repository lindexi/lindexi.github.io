# WPF 禁用实时触摸

微软想把 WPF 作为 win7 的触摸好用的框架，所以微软做了很多特殊的兼容。为了获得真实的触摸消息，微软提供了 OnStylusDown, OnStylusUp, 和 OnStylusMove 事件。

本文告诉大家如何使用代码禁用 WPF 的触摸消息，解决一些问题。

<!--more-->
<!-- CreateTime:2019/11/29 10:20:52 -->

<!-- csdn -->
<!-- 标签：WPF，触摸 -->

在 win7 还提供了多点触摸 windows 消息 WM_TOUCH ，通过这两个 API 一个是 OnStylusDown 这些事件，另一个就是 WM_TOUCH ，用户可以拿到触摸消息。

这两个 API 是相互独立，依靠相同的 HWND 。

那么为什么需要禁用 WPF 的 RealTimeStylus ，因为在 WPF 触摸平台会禁用 WM_TOUCH 消息。如果想要使用 WM_TOUCH ，在 WPF 需要禁用 WPF 的触摸事件。

如果没有禁用，就无法拿到 WM_TOUCH 消息，这个方法可以让自己定义自己的触摸。

禁用的方法使用下面代码

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

代码直接可以直接放在项目，代码是在微软文档复制。

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

详细请看 [通过 AppSwitch 禁用 WPF 内置的触摸让 WPF 程序可以处理 Windows 触摸消息 - walterlv](https://blog.walterlv.com/post/wpf-disable-stylus-and-touch-support.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
