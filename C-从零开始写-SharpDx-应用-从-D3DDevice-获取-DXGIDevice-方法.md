
# C# 从零开始写 SharpDx 应用 从 D3DDevice 获取 DXGIDevice 方法

本文将告诉大家在拿到了 D3D11 的 Device 也就是 D3DDevice 之后，如何去获取 DXGI.Device 以及 DXGI.Factory 的方法

<!--more-->


<!-- CreateTime:2021/1/11 8:23:59 -->


<!-- 标签：C#,D2D,DirectX,SharpDX,Direct2D, -->
<!-- 发布 -->

本文是 SharpDX 系列博客，更多博客请点击[SharpDX 系列](https://blog.lindexi.com/post/sharpdx.html )

在 [C# 从零开始写 SharpDx 应用 初始化dx修改颜色](https://blog.csdn.net/lindexi_gd/article/details/82114907 ) 这篇博客完成了基础的初始化

而接下来如何根据之前的代码，也就是如何根据上一篇博客创建的 D3D11.Device 来创建对应的 DXGI.Device 以及 DXGI.Factory 对象

其实方法十分简单，我记录一下，因此我刚才在写一些逗比代码的时候，又忘记是如何创建的

其实在创建 D3D11.Device 就用到了 DxGI 的 SwapChainDescription 交换链的描述对象，而调用了 D3D11.Device.CreateWithSwapChain 可以创建对应的交换链

```csharp
  D3D11.Device.CreateWithSwapChain(/*忽略代码*/, out D3D11.Device _d3DDevice, out DXGI.SwapChain _swapChain);
  // 看过前面系列博客的小伙伴就知道，其实 _d3DDevice 和 _swapChain 都是我定义好的字段，可不是在这里定义的变量哦。只是为了方便，没有修改变量名而已
```

有两个创建的方法，一个是根据 `D3D11.Device _d3DDevice` 来创建，另一个就是根据 `DXGI.SwapChain _swapChain` 来创建

在 `_d3DDevice` 调用 QueryInterface 可以拿到 `DXGI.Device` 对象

```csharp
  var dxgiDevice = _d3DDevice.QueryInterface<DXGI.Device>();
```

而通过 DXGI.Device 获取 DXGI.Factory 对象还有一点坑，需要根据 DxGI 的显卡适配器 `DXGI.Adapter` 通过查询它的父级找到是哪个 DXGI.Factory 枚举出来的适配器

```csharp
  DXGI.Adapter dxgiDeviceAdapter = dxgiDevice.Adapter;
  var dxgiFactory = dxgiDeviceAdapter.GetParent<DXGI.Factory>();
```

其实这就是坑了我的地方，也是我对 DirectX 的概念理解不够熟的原因，我开始就通过 DXGI.Device 对象尝试 GetParent 获取 DXGI.Factory 但是不出意外炸了

```csharp
  var dxgiDevice = _d3DDevice.QueryInterface<DXGI.Device>();
  var dxgiFactory = dxgiDevice.GetParent<DXGI.Factory>();
```

提示内容如下

```
SharpDX.SharpDXException:“HRESULT: [0x80004002], Module: [General], ApiCode: [E_NOINTERFACE/No such interface supported], Message: 不支持此接口”

   at SharpDX.Result.CheckError()
   at SharpDX.DXGI.DXGIObject.GetParent(Guid riid, IntPtr& parentOut)
   at SharpDX.DXGI.DXGIObject.GetParent[T]()
```

而另一个方法获取是，在拿到 `DXGI.SwapChain _swapChain` 对象，就可以使用下面代码获取工厂

```csharp
  var dxgiFactory2 = _swapChain.GetParent<DXGI.Factory>();
  var dxgiDevice2 = _swapChain.GetDevice<DXGI.Device>();
```

可以看到这两个方法获取的对象是相同的，如下面代码

```csharp
            if (dxgiDevice.NativePointer == dxgiDevice2.NativePointer)
            {

            }

            if (dxgiFactory.NativePointer == dxgiFactory2.NativePointer)
            {

            }
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/1d9754c2/Dx ) 欢迎小伙伴访问

## 参考

[SharpDX Beginners Tutorial Part 3: Initializing DirectX - Johan Falk](http://www.johanfalk.eu/blog/sharpdx-beginners-tutorial-part-3-initializing-directx )

[SharpDX 系列](https://blog.lindexi.com/post/sharpdx.html )

[WPF 底层渲染](https://blog.csdn.net/lindexi_gd/category_9276313.html )

[Directx11入门之D3D程序初始化 - 九野的博客 - CSDN博客](https://blog.csdn.net/acmmmm/article/details/79369294 )

[Directx11入门之第五章 渲染管线 - 九野的博客 - CSDN博客](https://blog.csdn.net/acmmmm/article/details/79394416 )

[Direct3D 11入门级知识介绍](https://blog.csdn.net/pizi0475/article/details/7786348 )

[Direct3D设备](https://blog.csdn.net/nightelve/article/details/6460477 )

[D3D11_CREATE_DEVICE_FLAG enumeration](https://msdn.microsoft.com/en-us/library/windows/desktop/ff476107(v=vs.85).aspx )

[DirectX11 With Windows SDK--01 DirectX11初始化_X_Jun的博客-CSDN博客](https://blog.csdn.net/X_Jun96/article/details/80293708)

[Direct3D feature levels - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/direct3d11/overviews-direct3d-11-devices-downlevel-intro?WT.mc_id=DX-MVP-5003606)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。