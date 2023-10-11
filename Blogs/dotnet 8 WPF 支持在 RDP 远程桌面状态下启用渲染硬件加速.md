本文将和大家介绍在 dotnet 8 里 WPF 引入的新功能之一，在 RDP 远程桌面状态下启用渲染硬件加速

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

在 dotnet 8 之前，在用户进行 RDP 远程桌面时 WPF 应用将默认关闭硬件渲染加速以获得更好的兼容性。随着系统层的渲染架构的优化，比如在 WDDM 驱动模型里面，进行远程桌面的硬件加速已经是非常简单且稳定的事情，这就意味着 WPF 框架底层可以不再判断当前是远程桌面模式时切换到软渲染模式，可以依旧使用硬件渲染加速。只有在 XP 系统上的 XDDM/XPDM 时，才需要在 RDP 远程桌面开启 WPF 应用软渲染

这个优化点在 2020 时就被 [Vatsan Madhavan](https://github.com/vatsan-madhavan) 大佬提出，详细请看 https://github.com/dotnet/wpf/issues/3215

由于 XP 已经过于考古且 dotnet 8 接近完全无法在 XP 系统上运行，于是我就对此进行优化，请看 https://github.com/dotnet/wpf/pull/7015

然而大佬们经过测试发现了一些额外的问题，且担心这个改动过于底层加上没有充分的测试，怕在一些奇怪的设备上运行错误。于是大佬重新提了 [https://github.com/dotnet/wpf/pull/7684](https://github.com/dotnet/wpf/pull/7684) 用来提供配置的方式控制此功能，默认行为依然保持在 RDP 远程桌面时 WPF 应用将关闭硬件渲染加速，依然使用软渲染方式，保持和旧版本行为相同

只有在进行 `Switch.System.Windows.Media.EnableHardwareAccelerationInRdp` 配置之后，才会让 WPF 应用在远程桌面时开启硬件渲染

简单来说就是在 dotnet 8 下，默认情况这个新功能对任何开发者或用户是没有影响的，只有在开发者通过 `Switch.System.Windows.Media.EnableHardwareAccelerationInRdp` 配置启用 RDP 远程桌面时使用硬件渲染加速功能才会生效

以下是配置启用 RDP 远程桌面时使用硬件渲染加速功能的方法，以下代码推荐放在 App 构造函数，否则将可能渲染线程已经跑起来导致配置无效

```csharp
        public App()
        {
            AppContext.SetSwitch("Switch.System.Windows.Media.EnableHardwareAccelerationInRdp", true);
        }
```

此功能在 dotnet 8 的行为是默认不开启的，这是因为缺乏足够的设备进行测试。计划在 dotnet 9 里面作为正式的功能
