# C# 从零开始写 SharpDx 应用 禁止 Alt + Enter 进入全屏

有小伙伴给我报了一个坑，那就是使用我的博客写的方法，在按下 ALT+ENTER 键时，将会让屏幕黑屏。其实原因就是默认的 DX 关联了 ALT+ENTER 快捷键，进入了全屏。本文来告诉大家如何解决

<!--more-->
<!-- CreateTime:2021/1/11 8:23:59 -->


<!-- 标签：C#,D2D,DirectX,SharpDX,Direct2D, -->
<!-- 发布 -->

这里使用 Alt + Enter 键进入全屏，其实进入的是独占的全屏，和我上次告诉大家的使用以下几篇博客的方法不同

- [C# 纯控制台创建一个全屏窗口](https://blog.lindexi.com/post/C-%E7%BA%AF%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E5%85%A8%E5%B1%8F%E7%AA%97%E5%8F%A3.html)
- [WPF 全屏透明窗口](https://blog.lindexi.com/post/WPF-%E5%85%A8%E5%B1%8F%E9%80%8F%E6%98%8E%E7%AA%97%E5%8F%A3.html)

但是和 [SharpDx 进入全屏模式](https://blog.lindexi.com/post/SharpDx-%E8%BF%9B%E5%85%A5%E5%85%A8%E5%B1%8F%E6%A8%A1%E5%BC%8F.html) 这篇博客说到的方法相同，都是采用独占的方式。其实在 Win10 是否独占的优势不大，因为系统有优化，请看 [Windows 对全屏应用的优化](https://blog.lindexi.com/post/Windows-%E5%AF%B9%E5%85%A8%E5%B1%8F%E5%BA%94%E7%94%A8%E7%9A%84%E4%BC%98%E5%8C%96.html)

而为什么在按下 ALT+ENTER 键时，将会让屏幕黑屏，是因为进入全屏之后，没有任何的更新，因此看起来黑屏

解决方法就是在 DXGI.Factory 设置窗口关联，忽略 Alt + Enter 键进入全屏

```csharp
            dxgiFactory.MakeWindowAssociation(_renderForm.Handle, WindowAssociationFlags.IgnoreAltEnter);
```

而 DXGI.Factory 可以通过 D3D11.Device 获取，如下面代码

```csharp
            var dxgiDevice = _d3DDevice.QueryInterface<DXGI.Device>();
            DXGI.Adapter dxgiDeviceAdapter = dxgiDevice.Adapter;
            var dxgiFactory = dxgiDeviceAdapter.GetParent<DXGI.Factory>();
```

本文是 SharpDX 系列博客，更多博客请点击[SharpDX 系列](https://blog.lindexi.com/post/sharpdx.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

