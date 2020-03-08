# UWP 和 WPF 对比

本文告诉大家 UWP 和 WPF 的不同。

如果在遇到技术选择或者想和小伙伴吹的时候可以让他以为自己很厉害，那么请继续看。

<!--more-->
<!-- CreateTime:2018/5/5 17:23:33 -->

<!-- csdn -->

<!-- 标签：wpf,uwp,dotnetcore -->

如果在看这文章还不知道什么是 UWP 和 WPF 那么也没关系，下面会告诉大家。

实际上 Universal Windows Platform (UWP) 和 Windows Presentation Foundation (WPF) 是不相同的，虽然都可以做界面和桌面开发，但是 UWP 是一个新的 UI 框架，而且 UWP 是支持很多平台，至少比 WPF 多。


那么 UWP 可以使用什么写？

- xaml 做的 UI 和 C#、VB 写的后台

- xaml 的 UI 和 C++ Native 写的后台

- DirectX 的 UI 和 C++ Native 写的后台

- JavaScript 和 HTML

WPF 呢？他可以使用 xaml 做的前台，C#、VB、F#、C++写的后台。

不过需要知道，WPF 的C++ 后台使用的是托管的C++。

那么网上怎么好多小伙伴说 UWP 的性能比 WPF 好？

因为 UWP 的渲染使用的是 [DirectComposition](https://msdn.microsoft.com/zh-cn/library/windows/desktop/hh437376.aspx ) 而 WPF 使用的 [Desktop Window Manager](https://msdn.microsoft.com/en-us/library/windows/desktop/aa969540(v=vs.85).aspx )，请不要在这里和我说 WPF 使用的 DX9 。

虽然 WPF 渲染是通过 Dx9 但是最后显示出来是需要 DWM ，所以上面这样说。

之外，UWP 使用 dot net core 编译出来的是 Native 本地代码，WPF 使用 dot net Framework 编译出来是 IL 代码，需要知道 编译出来 Native 代码的性能是 80% C++非托管。所以代码运行会快很多。

这时不要说 IL 可以针对每个 CPU 做优化，因为 dot net core 编译的代码就是对不同的 CPU 做优化。如果还需要对特殊CPU做优化，我还没找到。

## 发布时间

WPF 在 2006 年发布，那时使用的 .net Framework 3.0 ，现在已经是 WPF 4 了，支持的 .net Framework 已经到 4.7.x 了。

UWP 在 2015 年发布，那时还没有 dot net core 1.0

所以垃圾微软的 UWP 有兼容问题，如果选择最低平台，千万不要 10240 这个版本的 api 很多后来系统没有提供的，这是兼容的问题。很多之前的没有公布的 api 已经去掉，很多以前的api已经被标记过时了。

## 系统要求

因为 WPF 发布的时候还没有 Win7 所以 WPF 是支持 xp 的。但是如果需要支持 xp 就需要使用不大于 .net Framework 4.0 的版本，如果比 4.0 大就无法支持 xp 啦。

需要知道，在 4.5之后 WPF 才修复很多 bug ，提升性能，能不支持 xp 就不要支持 xp。

UWP 发布的时候，因为使用的是 WinRT ，虽然底层和 WPF 一样使用的是 COM 但是添加了很多以前系统不支持的特性。微软为了减少开发或者基于某些考虑，于是UWP不支持以前系统，最低是 win10.

## 平台

虽然 WPF 很厉害，但是发布的时候几乎没有人知道多平台，所以 WPF 只能支持桌面和 windows 平板。但是现在有 [Avalonia](https://github.com/avaloniaUI/Avalonia) 和 [Xamarin WPF](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/platform/wpf )，这两个都是可以支持很多平台，如 mac 和 Linux ，需要说的是，我一个在开发 Xamarin 的小伙伴说，WPF 是一个恐怖的工程，他不觉得很快就可以把Xamarin WPF放在实际项目。

但是 [Avalonia](https://github.com/avaloniaUI/Avalonia) 只要可以注入渲染就可以使用，输入部分做很少修改就可以在很多平台跑，虽然几个小伙伴告诉我他的树莓派无法运行 [Avalonia](https://github.com/avaloniaUI/Avalonia) ，不过对于一个开源软件，要修改还是很简单。

对比 WPF ，全平台 UWP 支持PC， Notebook， Tablet， Phone， Xbox， IoT， Surface Hub，需要说的最后一个大家不用支持，因为说了10年还没有卖。但是 UWP 支持的都是 微软的系统，对于 Mac 和 Linux 暂时是无法支持的。如果开发的客户需要使用 Mac 和 Linux ，自己不想开发多个代码就不可以选择 UWP 。

但是如果需要性能和支持好的触摸，建议选择 UWP ，不过 UWP 的坑还是好多。

## 输入

虽然微软说 WPF 是支持触摸的，但是在 4.7 之前的触摸是很差的。所以 WPF 支持鼠标键盘。

但是 UWP 是支持触摸的，鼠标、键盘。

对于 触摸的支持，uwp 是做的很好的，不仅支持了 4.7.1 的指针消息而且还内部支持很多手势。

对于AR的输入，uwp也是支持的。

## 界面

虽然 WPF 和 UWP 都使用 xaml 做界面，但是渲染是不相同的。 WPF 的渲染都是使用托管代码计算，然后通过通道使用 DirectX 9 渲染。渲染完成给 DWM 选择是否显示。但是 WPF 没有使用 DirectX 9 的性能，所以渲染是比较慢的。听说 WPF 可以使用 dx11 dx12都是使用优化级别是 fl9 。

我尝试使用 WriteableBitmap 渲染，结果性能比 WPF 渲染快。

但是 UWP 的渲染很快，因为他使用[DirectComposition](https://msdn.microsoft.com/zh-cn/library/windows/desktop/hh437376.aspx )直接渲染，使用 DX11 渲染。DirectComposition 是通过集成 DWM 渲染的。组合的图形和动画通过 DirectComposition 构建然后传到 DWM 渲染到屏幕。所以使用 DirectComposition 不需要特殊的渲染框架。而且渲染的代码都是编译本地，比较多使用 DX11 ，但是对于很多硬件都支持 dx12 。

那么 DWM 的作用是什么，实际上从[博客](https://msdn.microsoft.com/magazine/dn745861 )可以看到 DWM 实际作用 Windows 组合引擎或合成程序，需要每个窗口把显示的内容给屏外表面或缓冲区，缓冲区是系统给每个顶层窗口分配的，所有的 GDI、D3D、D2D 到先渲染到这里。然后 DWM 决定如何显示，是组合窗口还是做特效，最后再把缓存放到显卡。

参见：[Why use DirectComposition? (Windows)](https://msdn.microsoft.com/en-us/library/windows/desktop/hh449195(v=vs.85).aspx )

## 定制

虽然开始的 WPF 定制不好，但是现在的 WPF 定制是很好的，可以使用 Host 加入其他的程序，可以使用 dx 加入。如何在 WPF 使用 dx 是相对比较难的，但是可以使用 SharpDx 和 SharpGL 使用 dx 和 opg。

但是 UWP 的定制虽然像 UWP 但是限制很多，一个就是他的源代码看不到，其他的就是很多功能无法使用，如 Adorner 和继承属性。

## 样式

虽然看起来 WPF 和 UWP 的样式定义是一样的，但是 UWP 没有了功能很好的 Trigger 和样式继承。这样 UWP 的功能就没有 WPF 那么容易定制。

而且 WPF 和 UWP 的设计器经常无法使用，不过两个都可以在运行修改样式。但是在运行时可以 WPF 可以通过 Snoop 查看元素的值，但是 UWP 不可以，所以调试 UWP 界面还是比较难。

## 调试

在 WPF 如果有一个代码抛异常，那么 VisualStudio 很容易告诉大家是哪里异常，因为 VisualStudio 也是 WPF 写的。但是如果 dot net core 抛异常，那么 VisualStudio 很难告诉但是哪里异常，不过 UWP 是 dot net core 写的，所以 UWP 的异常很难知道是哪里异常，特别是界面异常，经常告诉大家出现一个未知的异常。

对比 WPF，现在的微软开放了部分 dot net framework 的源代码，在 WPF ，即使没有源代码，自己反编译也可以看到。但是在 UWP ，没有源代码，而且难以反编译，如果遇到坑都不知道是不是微软的代码写的。

需要说的是 UWP 用的 dot net core 是开放源代码的，如果大家想读代码可以在 github 搜索 dot net core 就可以下载。

## 安装

现在的 WPF 可以做绿色版，直接运行就可以。不过要求用户的电脑有 .net Framework 。

但是 dot net core可以带所有的环境，所以 UWP 安装不需要要求用户有 .net Framework 环境，不过 UWP 只能通过应用商店和开发者方式安装。

很多小伙伴还不会安装 UWP 程序，虽然双击就可以安装。

虽然 UWP 不需要 .net framework 环境，但是他需要 win10 ，现在很多用户的电脑都是 win7 所以暂时很难直接告诉大家使用 UWP ，很多人无法安装。

## 文件

在 WPF 几乎所有文件都能使用，如果发现有文件没权限，那么可以通过申请的方式拿到。对比 WPF 这么不安全，因为可能有开发者删掉了重要的文件，微软提出了安全的代码，所有的不是应用内的文件都需要用户申请才可以。

## 成熟

WPF 是比较成熟的，现在已经有 10 多年，有很多库，而且遇到的问题基本都有人遇到。

对于 UWP ，是比较不成熟，很多功能没有。



参见：[UWP vs. WPF · jbe2277/waf Wiki](https://github.com/jbe2277/waf/wiki/UWP-vs.-WPF )

## 感谢

特别感谢 Naruto Mouri 指出文章的不足

![](https://i.loli.net/2018/04/08/5ac9fff835cfe.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
