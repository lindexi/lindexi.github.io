# UWP 和 WPF 对比

本文告诉大家 UWP 和 WPF 的一些不同之处

<!--more-->
<!-- CreateTime:2018/5/5 17:23:33 -->

<!-- csdn -->

<!-- 标签：wpf,uwp,dotnetcore -->

从技术上讲 Universal Windows Platform (UWP) 和 Windows Presentation Foundation (WPF) 是不相同的，虽然都可以做界面和桌面开发，但 WPF 是 Win32 系的，而 UWP 作为一个新的 UI 框架，属于 WinRT 系的。在微软的野心驱动下 UWP 是支持很多平台，虽然这些平台除了 Windows 外，几乎没一个能打的，但从支持的平台数量上就是比 WPF 多

开始之前必须说明的是 WPF 和 UWP 不是两个对立的框架，而是两个可以相互嵌套同时使用的框架。换句话说我可以在一个应用里面同时使用 WPF 和 UWP 框架。我可以按照我的心情决定哪部分界面使用 UWP 写哪部分界面使用 WPF 写

那么 UWP 可以使用什么语言写？

- xaml 做的 UI 和 C#、VB 写的后台逻辑代码

- xaml 的 UI 和 C++ Native 写的后台逻辑代码

- DirectX 的 UI 和 C++ Native 写的后台逻辑代码

- JavaScript 和 HTML 的组合，只不过这个和废物一样

而 WPF 呢？他可以使用 xaml 做的 UI 界面，使用 C#、VB、F#、C++写的后台逻辑代码

不过需要知道的是，以上说的 WPF 的 C++ 后台逻辑代码，使用的是托管的 C++ 方式，而不是传统的 C++ 方式

那么网上怎么好多小伙伴说 UWP 的性能比 WPF 好？

性能好不好有多个方面，讨论性能问题可以分为两个方面，一个是框架级方面，一个是业务级方面。框架级上核心的不同在于 UWP 默认用上了 [DirectComposition](https://msdn.microsoft.com/zh-cn/library/windows/desktop/hh437376.aspx ) 技术进行渲染，而 WPF 框架默认没有用上。 从渲染上的性能差倒是和 DX9 等差异不大。业务级上的最大不同在于 UWP 砍掉了大量的 WPF 功能，将那些可能会影响性能的控件和写法大量的砍掉，从而减少开发者写出降低性能的代码，比如说 VisualBrush 等

<!-- 因为 UWP 的渲染使用的是 [DirectComposition](https://msdn.microsoft.com/zh-cn/library/windows/desktop/hh437376.aspx ) 而 WPF 使用的 [Desktop Window Manager](https://msdn.microsoft.com/en-us/library/windows/desktop/aa969540(v=vs.85).aspx )，请不要在这里和我说 WPF 使用的 DX9 。 -->

<!-- 虽然 WPF 渲染是通过 Dx9 但是最后显示出来是需要 DWM ，所以上面这样说。

之外，UWP 使用 dot net core 编译出来的是 Native 本地代码，WPF 使用 dot net Framework 编译出来是 IL 代码，需要知道 编译出来 Native 代码的性能是 80% C++非托管。所以代码运行会快很多。

这时不要说 IL 可以针对每个 CPU 做优化，因为 dot net core 编译的代码就是对不同的 CPU 做优化。如果还需要对特殊CPU做优化，我还没找到。 -->

## 发布时间

WPF 在 2006 年发布，那时使用的 .NET Framework 3.0 ，现在已经是 WPF 7 了，支持的 .NET 已经到 7 了

UWP 在 2015 年发布，那时还没有 .NET 1.0 的发布，这也导致了 10240 这个版本的存在许多诡异的问题。在 2302 年的时候，应该很少有开发者会考虑使用 10240 如此低的版本了

<!-- 所以垃圾微软的 UWP 有兼容问题，如果选择最低平台，千万不要 10240 这个版本的 api 很多后来系统没有提供的，这是兼容的问题。很多之前的没有公布的 api 已经去掉，很多以前的api已经被标记过时了。 -->

## 系统要求

采用 WPF 框架搭配 .NET Framework 4.0 可以最低支持到 XP 系统。 但值得一提的是 .NET Framework 在 4.5 之后才是大力优化提升，修复了大量的问题，不仅仅只是 WPF 框架层的问题，也修复了 .NET Framework 的问题

跟随 Win10 发布的 UWP 自然没话说，最低系统要求就是 Win10 系统。由于 UWP 使用了 Win10 系统的 WinRT 且使用 Win10 系统的 Pointer 输入系统，同时也使用到 Win10 的渲染架构，这就意味着从技术上 UWP 就不能在 Win10 以下的版本运行

<!-- 因为 WPF 发布的时候还没有 Win7 所以 WPF 是支持 xp 的。但是如果需要支持 xp 就需要使用不大于 .net Framework 4.0 的版本，如果比 4.0 大就无法支持 xp 啦。

需要知道，在 4.5之后 WPF 才修复很多 bug ，提升性能，能不支持 xp 就不要支持 xp。

UWP 发布的时候，因为使用的是 WinRT ，虽然底层和 WPF 一样使用的是 COM 但是添加了很多以前系统不支持的特性。微软为了减少开发或者基于某些考虑，于是UWP不支持以前系统，最低是 win10. -->

<!-- ## 平台

虽然 WPF 很厉害，但是发布的时候几乎没有人知道多平台，所以 WPF 只能支持桌面和 windows 平板。但是现在有 [Avalonia](https://github.com/avaloniaUI/Avalonia) 和 [Xamarin WPF](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/platform/wpf )，这两个都是可以支持很多平台，如 mac 和 Linux ，需要说的是，我一个在开发 Xamarin 的小伙伴说，WPF 是一个恐怖的工程，他不觉得很快就可以把Xamarin WPF放在实际项目。

但是 [Avalonia](https://github.com/avaloniaUI/Avalonia) 只要可以注入渲染就可以使用，输入部分做很少修改就可以在很多平台跑，虽然几个小伙伴告诉我他的树莓派无法运行 [Avalonia](https://github.com/avaloniaUI/Avalonia) ，不过对于一个开源软件，要修改还是很简单。

对比 WPF ，全平台 UWP 支持PC， Notebook， Tablet， Phone， Xbox， IoT， Surface Hub，需要说的最后一个大家不用支持，因为说了10年还没有卖。但是 UWP 支持的都是 微软的系统，对于 Mac 和 Linux 暂时是无法支持的。如果开发的客户需要使用 Mac 和 Linux ，自己不想开发多个代码就不可以选择 UWP 。

但是如果需要性能和支持好的触摸，建议选择 UWP ，不过 UWP 的坑还是好多。 -->

## 输入

默认的 WPF 是触摸友好的，但也要说版本，在 .NET Framework 4.7 之前的触摸支持较弱，会有一些诡异的触摸问题。这里说的 .NET Framework 4.7 指的是运行时框架版本，而不是开发 SDK 版本。意味着即使开发者使用 SDK 版本是 .NET Framework 4.5 版本，但只要用户端安装的是 .NET Framework 4.7 以上版本，即可修复许多触摸问题

和 UWP 相同的是 WPF 也可以在 .NET Framework 4.7 或 .NET Core 以上版本开启 Pointer 消息。通过 Pointer 消息可以支持触摸、鼠标、视线等输入

<!-- 虽然微软说 WPF 是支持触摸的，但是在 4.7 之前的触摸是很差的。所以 WPF 支持鼠标键盘。

但是 UWP 是支持触摸的，鼠标、键盘。

对于 触摸的支持，uwp 是做的很好的，不仅支持了 4.7.1 的指针消息而且还内部支持很多手势。

对于AR的输入，uwp也是支持的。 -->

## 界面

虽然 WPF 和 UWP 都使用 xaml 做界面，但是渲染是不相同的。 WPF 的渲染都是使用托管代码计算，然后通过 Channel 通道给到 GFX 层，底层调用 DirectX 9 的 API 进行渲染。渲染完成给 DWM 显示到屏幕上

而 UWP 的渲染底层采用 DirectX + [DirectComposition](https://msdn.microsoft.com/zh-cn/library/windows/desktop/hh437376.aspx ) 方式，通过 [DirectComposition](https://msdn.microsoft.com/zh-cn/library/windows/desktop/hh437376.aspx ) 方式提升了渲染输出性能

从 Demo 级应用上来说，采用 UWP 的渲染损耗更低，性能更高。从大应用来说，由于性能点不会在渲染输出等的性能损耗上，总的来说 WPF 和 UWP 的渲染差异极限情况下可以持平，或者说 WPF 略弱于 UWP 的渲染性能。在使用 Windows App SDK 后，即可在 WPF 用上 [DirectComposition](https://msdn.microsoft.com/zh-cn/library/windows/desktop/hh437376.aspx ) 渲染，此时两者底层渲染相同，理论上渲染性能相同

两者相同的是，界面也可以完全扔掉 XAML 换成完全的 C# 代码完成界面

<!-- 。但是 WPF 没有使用 DirectX 9 的性能，所以渲染是比较慢的。听说 WPF 可以使用 dx11 dx12都是使用优化级别是 fl9 。 -->

<!-- 我尝试使用 WriteableBitmap 渲染，结果性能比 WPF 渲染快。 其实纯是当时写博客那会我的技术还不足 -->

<!-- 但是 UWP 的渲染很快，因为他使用[DirectComposition](https://msdn.microsoft.com/zh-cn/library/windows/desktop/hh437376.aspx )直接渲染，使用 DX11 渲染。DirectComposition 是通过集成 DWM 渲染的。组合的图形和动画通过 DirectComposition 构建然后传到 DWM 渲染到屏幕。所以使用 DirectComposition 不需要特殊的渲染框架。而且渲染的代码都是编译本地，比较多使用 DX11 ，但是对于很多硬件都支持 dx12 。

那么 DWM 的作用是什么，实际上从[博客](https://msdn.microsoft.com/magazine/dn745861 )可以看到 DWM 实际作用 Windows 组合引擎或合成程序，需要每个窗口把显示的内容给屏外表面或缓冲区，缓冲区是系统给每个顶层窗口分配的，所有的 GDI、D3D、D2D 到先渲染到这里。然后 DWM 决定如何显示，是组合窗口还是做特效，最后再把缓存放到显卡。

这个说法也是不全对的

参见：[Why use DirectComposition? (Windows)](https://msdn.microsoft.com/en-us/library/windows/desktop/hh449195(v=vs.85).aspx ) -->

## 定制

古老的 WPF 是完全封闭的，也是闭源的，定制不友好。但是在 2016 年之后，整个 WPF 具有极高的定制性，当前的 WPF 在 [https://github.com/dotnet/wpf](https://github.com/dotnet/wpf) 完全开源，使用友好的 MIT 协议，意味着允许任何人任何组织和企业任意处置，包括使用，复制，修改，合并，发表，分发，再授权，或者销售。在仓库里面包含了完全的构建逻辑，只需要本地的网络足够好（因为需要下载一堆构建工具），即可进行本地构建。开源的，也就完全可以定制，例如我现在就使用自己的定制版

而 UWP 的定制性就非常有局限了，如果想要有更高的定制，可以考虑采用 WinUI 3 框架

<!-- 虽然开始的 WPF 定制不好，但是现在的 WPF 定制是很好的，可以使用 Host 加入其他的程序，可以使用 dx 加入。如何在 WPF 使用 dx 是相对比较难的，但是可以使用 SharpDx 和 SharpGL 使用 dx 和 opg。

但是 UWP 的定制虽然像 UWP 但是限制很多，一个就是他的源代码看不到，其他的就是很多功能无法使用，如 Adorner 和继承属性。 -->

## 样式

虽然看起来 WPF 和 UWP 的样式定义是一样的，但是 UWP 没有了功能很好的 Trigger 和样式继承。这样 UWP 的功能就没有 WPF 那么容易定制。

而且 WPF 和 UWP 的设计器经常无法使用，好在这两个都可以在运行修改样式。另一个差别是在运行时 WPF 可以通过 Snoop 工具查看元素的值，但是 UWP 不可以，总的来说调试 UWP 界面还是比较难的

## 调试

在 WPF 如果有一个代码抛异常，那么 VisualStudio 很容易告诉大家是哪里异常，也许 VisualStudio 也是 WPF 写的原因，一切工作的挺好的

如果是 WPF 框架层抛出的异常，在采用 .NET Core 版本时，可以在 VisualStudio 2022 或以上版本自动拉取 GitHub 上开源的代码，如此调试更加方便。在采用 .NET Framework 版本时，可以找到部分源代码，也可以辅助调试

由于 UWP 大量使用到 WinRT 组件，基于 COM 的 WinRT 组件经常只能给出一个 COM 异常，接近无法更近一步进行调试，除非开启本机代码调试

<!-- 。但是如果 dot net core 抛异常，那么 VisualStudio 很难告诉但是哪里异常，不过 UWP 是 dot net core 写的，所以 UWP 的异常很难知道是哪里异常，特别是界面异常，经常告诉大家出现一个未知的异常。 -->

<!-- 对比 WPF，现在的微软开放了部分 dot net framework 的源代码，在 WPF ，即使没有源代码，自己反编译也可以看到。但是在 UWP ，没有源代码，而且难以反编译，如果遇到坑都不知道是不是微软的代码写的。 -->

<!-- 需要说的是 UWP 用的 dot net core 是开放源代码的，如果大家想读代码可以在 github 搜索 dot net core 就可以下载。 -->

## 安装

现在的 WPF 可以做绿色版，直接运行就可以。在 .NET Core 3.1 及以上版本，可以带上框架环境，完全绿色不需要用户环境安装框架

现在的 UWP 可以通过应用商店正常发布，也可以通过旁加载方式做私有发布。在国内发布 UWP 产品应用，我基本都是通过私有发布方式。详细请看 [加强版在国内分发 UWP 应用正确方式 通过win32安装UWP应用](https://blog.lindexi.com/post/%E5%8A%A0%E5%BC%BA%E7%89%88%E5%9C%A8%E5%9B%BD%E5%86%85%E5%88%86%E5%8F%91-UWP-%E5%BA%94%E7%94%A8%E6%AD%A3%E7%A1%AE%E6%96%B9%E5%BC%8F-%E9%80%9A%E8%BF%87win32%E5%AE%89%E8%A3%85UWP%E5%BA%94%E7%94%A8.html )

<!-- 但是 dot net core可以带所有的环境，所以 UWP 安装不需要要求用户有 .net Framework 环境，不过 UWP 只能通过应用商店和开发者方式安装。

很多小伙伴还不会安装 UWP 程序，虽然双击就可以安装。

虽然 UWP 不需要 .net framework 环境，但是他需要 win10 ，现在很多用户的电脑都是 win7 所以暂时很难直接告诉大家使用 UWP ，很多人无法安装。 -->

## 文件权限

关于权限方面，基于 Win32 的 WPF 绝对能完成基本所有的流氓任务，没有什么限制。对比 WPF 这么不安全的方式，如果开发者使用 WPF 去做坏事，那肯定用户会将锅砸在微软头上，于是微软十分机智的给 UWP 加上了一大堆限制。想要使用 UWP 做什么都会缺少权限

<!-- 在 WPF 几乎所有文件都能使用，如果发现有文件没权限，那么可以通过申请的方式拿到。对比 WPF 这么不安全，因为可能有开发者删掉了重要的文件，微软提出了安全的代码，所有的不是应用内的文件都需要用户申请才可以。 -->

## 成熟程度

总的来说 WPF 是比较成熟的，距离 WPF 第一个版本已经差不多 20 年了，而且这差不多 20 年的时间内都在不断更新迭代，有大量的开发者踩过了大量的坑留下大量的经验，也有非常多的基础库可以快速实现功能

<!-- 现在已经有 10 多年，有很多库，而且遇到的问题基本都有人遇到。

对于 UWP ，是比较不成熟，很多功能没有。 -->

而 UWP 发布到现在快 10 年了，也算比较成熟的技术了，只是在 2023 年时我的测试发现了 UWP 依然受系统环境影响极大，从质量的角度上还是不能和 WPF 比

参见：[UWP vs. WPF · jbe2277/waf Wiki](https://github.com/jbe2277/waf/wiki/UWP-vs.-WPF )

## 感谢

特别感谢 Naruto Mouri 指出文章的不足

![](https://i.loli.net/2018/04/08/5ac9fff835cfe.jpg)

