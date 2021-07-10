
# C# 从零开始写 SharpDx 应用 聊聊功能等级

本文将和大家聊聊 DirectX 里面的功能等级在 SharpDx 的使用方法

<!--more-->


<!-- CreateTime:2021/1/11 8:23:59 -->


<!-- 标签：C#,D2D,DirectX,SharpDX,Direct2D, -->
<!-- 发布 -->

本文是 SharpDX 系列博客，更多博客请点击[SharpDX 系列](https://blog.lindexi.com/post/sharpdx.html )

在[C# 控制台创建 Sharpdx 窗口](https://blog.lindexi.com/post/C-%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E5%86%99-SharpDx-%E5%BA%94%E7%94%A8-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%88%9B%E5%BB%BA-Sharpdx-%E7%AA%97%E5%8F%A3.html )已经创建了一个窗口，现在需要在这个窗口初始化。因为是从零开始写，所以需要非常多细节，我觉得一篇文章是很难全部告诉大家，所以分为了系列的文章。从零开始写有利于大家了解一个渲染框架是如何做出来，并且从底层优化渲染，当然这个方法就是学习的时间会比较长。我会在文章去掉很多细节放在后面的博客讲，让大家先知道总体是如何做的

## 创建交换链

在 [C# 从零开始写 SharpDx 应用 初始化dx修改颜色](https://blog.csdn.net/lindexi_gd/article/details/82114907 ) 这篇博客里面有告诉大家如何创建交换链

在创建交换链的时候，可以有重载的方法，允许传入特性功能等级列表，如下面代码

```csharp
            D3D11.Device.CreateWithSwapChain
            (
                /*
                 * 第一个参数 DriverType.Hardware 表示希望使用 GPU 渲染，设置 驱动设备类型 可以设置硬件设备（hardware device）、参考设备（reference device）、软件驱动设备（software driver device）
                   
                   - 硬件设备（hardware device）是一个运行在显卡上的D3D设备，在所有设备中运行速度是最快的
                   
                   - 软件驱动设备（software driverdevice）是开发人员自己编写的用于Direct3D的渲染驱动软件
                   
                   - 参考设备（reference device）是用于没有可用的硬件支持时在CPU上进行渲染的设备
                   
                   - WARP设备（WARPdevice）是一种高效的CPU渲染设备，可以模拟现阶段所有的Direct3D特性
                 */
                DriverType.Hardware,
                // 第二个参数选不使用特殊的方法，参见 [D3D11_CREATE_DEVICE_FLAG enumeration](https://msdn.microsoft.com/en-us/library/windows/desktop/ff476107(v=vs.85).aspx )
                D3D11.DeviceCreationFlags.None,
                // 特性等级数组 [Direct3D feature levels - Win32 apps | Microsoft Docs](https://docs.microsoft.com/en-us/windows/win32/direct3d11/overviews-direct3d-11-devices-downlevel-intro)
                featureLevels: new FeatureLevel[]
                {
                    // 特性等级 也可以翻译为 功能等级
                    // 为了兼容新设备新系统和古老设备的系统，在 Dx10.1 推出的时候，就引入了功能等级的概念。每个显卡都会根据它自身的 GPU 图形处理单元采用一定等级的 DirectX 功能。在 DirectX 11 引入的功能等级的概念是一组明确的 GPU 功能，也就是说这是一个沟通硬件 GPU 和编程人员中间的特性，在调用此方法创建设备的时候，可以尝试为请求的功能等级创建设备（_d3DDevice）如果设备创建成功了，那么证明此特性等级存在。否则，表示在此设备上不支持此功能等级，咱可以使用较低的功能等级重新创建设备
                    // 利用此特性，就可以为 Dx9 和 Dx11 和 Dx12 开发应用程序，然后在不同的支持 Dx12 和 Dx11 和 Dx9 的设备上运行程序，可以极大减少开发人员对具体硬件的关注
                    // 需要了解的是：
                    // - 默认的 GPU 是允许设备创建的特性等级等于或超过他的能支持的功能等级
                    // - 功能等级始终包含先前的低功能等级的功能，换句话说就是 Level_12_1 等级的包含了 Level_11_1 等级功能
                    // - 功能等级不代表性能，而仅代表功能。性能取决于硬件实现
                    // 不同的功能等级对应支持的功能列表请看  [Direct3D feature levels - Win32 apps | Microsoft Docs](https://docs.microsoft.com/en-us/windows/win32/direct3d11/overviews-direct3d-11-devices-downlevel-intro)
                    FeatureLevel.Level_12_1,
                    FeatureLevel.Level_11_1,
                    // 给 Win7 用的
                    FeatureLevel.Level_11_0,
                    // 给 xp 用的
                    FeatureLevel.Level_9_1,
                },
                // 第三个参数是输入上面的交换链描述
                swapChainDesc,
                // D3D设备（ID3D11Device）通常代表一个显示适配器（即显卡），它最主要的功能是用于创建各种所需资源，最常用的资源有：资源类（ID3D11Resource, 包含纹理和缓冲区），视图类以及着色器。此外，D3D设备还能够用于检测系统环境对功能的支持情况
                out _d3DDevice,
                out _swapChain)
```

如上面代码里面的注释所示 特性等级 也可以翻译为 功能等级 为了兼容新设备新系统和古老设备的系统，在 Dx11 推出的时候，就引入了功能等级的概念。每个显卡都会根据它自身的 GPU 图形处理单元采用一定等级的 DirectX 功能。在 DirectX 11 引入的功能等级的概念是一组明确的 GPU 功能，也就是说这是一个沟通硬件 GPU 和编程人员中间的特性，在调用此方法创建设备的时候，可以尝试为请求的功能等级创建设备（`_d3DDevice`）如果设备创建成功了，那么证明此特性等级存在。否则，表示在此设备上不支持此功能等级，咱可以使用较低的功能等级重新创建设备

利用此特性，就可以为 Dx9 和 Dx11 和 Dx12 开发应用程序，然后在不同的支持 Dx12 和 Dx11 和 Dx9 的设备上运行程序，可以极大减少开发人员对具体硬件的关注

需要了解的是：

- 默认的 GPU 是允许设备创建的特性等级等于或超过他的能支持的功能等级
- 功能等级始终包含先前的低功能等级的功能，换句话说就是 Level_12_1 等级的包含了 Level_11_1 等级功能
- 功能等级不代表性能，而仅代表功能。性能取决于硬件实现


不同的功能等级对应支持的功能列表请看  [Direct3D feature levels - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/direct3d11/overviews-direct3d-11-devices-downlevel-intro?WT.mc_id=WD-MVP-5003260)

## 判断设备的支持功能等级

可以使用 CheckD3D113Features 等方法判断，如下面代码

```csharp
if (_d3DDevice.CheckD3D113Features4().ExtendedNV12SharedTextureSupported==true)
{
    // 1. 特性等级的支持情况取决于当前使用的显示适配器，只要显示适配器支持某一特性等级，意味着它能够支持该特性等级下的统一功能（如特性等级11.0支持纹理宽高最大为16384，而10.1仅支持纹理宽高最大为8192）
    // 2. D3D设备的版本取决于所处的系统（有时候可以打特定的系统补丁来支持高版本的DX，比如让Win7支持DX12的部分）
}
```

或者获取设备的 FeatureLevel 属性

```csharp
// 该函数可以创建Direct3D 11.0或更高子版本的D3D设备与设备上下文，但都统一输出 _d3DDevice 设备
var featureLevel = _d3DDevice.FeatureLevel;
```

如上面代码在我的设备上输出的 FeatureLevel 是 `SharpDX.Direct3D.FeatureLevel.Level_12_1` 因为我在 Win10 的设备上运行

![](http://image.acmx.xyz/lindexi%2F20211101215457949.jpg)

在 WPF 中的 MIL 层的渲染其实也用到了这个功能，这样也就支持了在不同的设备上能跑起来。如上文所说，功能等级只是代表有多少功能而已，和性能无关

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/12811cf7/Dx ) 欢迎小伙伴访问

欢迎加入QQ群: 727623616 可以一起探讨DX11，以及有什么问题也可以在这里问群主（不要问我）

## 参考

[DirectX11 With Windows SDK--01 DirectX11初始化_X_Jun的博客-CSDN博客](https://blog.csdn.net/X_Jun96/article/details/80293708)

[SharpDX Beginners Tutorial Part 3: Initializing DirectX - Johan Falk](http://www.johanfalk.eu/blog/sharpdx-beginners-tutorial-part-3-initializing-directx )

[SharpDX 系列](https://blog.lindexi.com/post/sharpdx.html )

[WPF 底层渲染](https://blog.csdn.net/lindexi_gd/category_9276313.html )

[Directx11入门之D3D程序初始化 - 九野的博客 - CSDN博客](https://blog.csdn.net/acmmmm/article/details/79369294 )

[Directx11入门之第五章 渲染管线 - 九野的博客 - CSDN博客](https://blog.csdn.net/acmmmm/article/details/79394416 )

[Direct3D 11入门级知识介绍](https://blog.csdn.net/pizi0475/article/details/7786348 )

[Direct3D设备](https://blog.csdn.net/nightelve/article/details/6460477 )

[D3D11_CREATE_DEVICE_FLAG enumeration](https://msdn.microsoft.com/en-us/library/windows/desktop/ff476107(v=vs.85).aspx )

[Direct3D feature levels - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/direct3d11/overviews-direct3d-11-devices-downlevel-intro?WT.mc_id=WD-MVP-5003260)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。