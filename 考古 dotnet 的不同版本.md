# 考古 dotnet 的不同版本

本文来考古一下 dotnet 发布过的版本，相信本文里面有很多个版本都是大家很少听过的

<!--more-->
<!-- CreateTime:2021/5/21 8:31:35 -->


<!-- 发布 -->

什么才能称为一个 dotnet 版本？从 .NET Framework 1.0-4.8 算多少个版本？其实我这里说的版本指的是不同的实现，我将整个 .NET Framework 算作一个版本。而将 .NET 的不同的分支，分别作为不同的版本

下面列举一下 dotnet 的不同版本

## .NET Framework

这大概就是咱最熟悉的一个 dotnet 实现版本，从 2002 年发布到现在，当前最新是 .NET Framework 4.8 版本

## .NET Core

这是 dotnet 的里程碑，作为完全开源的，基于 MIT 协议的实现版本，从构建到运行时完全开源

这也是跨平台版本的 dotnet 实现版本，可以在多个平台运行。相信有关注技术的开发者都对 .NET Core 非常熟悉

## .NET

当前的 dotnet 主实现版本，从 .NET 5 开始，官方将定 .NET 作为主版本。将原有的 .NET Core 等版本合入在一起，放在一个仓库进行开发

当然，如果强行说 .NET 合并了 .NET Core 和 .NET Framework 和 Mono 等实现版本，这也不算全错，也不能说对。可以认为 .NET Core 和 .NET Framework 等实现版本的下一代就是 .NET 版本

## Mono

这是 dotnet 跨平台的先锋，这是一个完全独立、跨平台的CLI实现。当前也合入到 .NET 主版本

据说 Unity3D 里面有基于 Mono 维护过一个自己的版本，但只是据说哈

在 Blazor 预览版本，也有基于 Mono 实现的 WASM 版本，当前这个版本合入到 .NET 版本

## WinRT

这是在微软通用平台开发概念提出写的框架，本质上说，这不算是一个 .NET 的实现版本，当前这个 WinRT 依然在维护中，可以通过 NuGet 在 .NET Core 和 .NET 5 使用。通过 WinRT 可以访问现代化的 Windows 系统的功能

## .NET Native

这也是 .NET 的很特殊的实现版本，大多数的 .NET 实现版本，在构建阶段都是将 C# 等代码构建为 IL 代码，在运行时进行翻译为本机代码。而 .NET Native 是将 IL 构建为本机代码，让 C# 代码一步生成为本机代码的实现。提供了很快的启动性能，以及减少运行时的资源占用。此实现当前依然在开发中。通过 .NET Native 技术可以压缩发布文件体积，减少环境依赖

如果将 .NET Native 算作 .NET 的一个实现版本，那么 CrossGen 和 CrossGen2 也可以算做 .NET 的一个实现，这两个实现是前后继承的关系，在 .NET 6 准备开始切为 CrossGen2 实现，使用 CrossGen2 可以提供在开发者构建时，将一部分逻辑提前加上本机代码构建，用来提升启动性能

## IL2CPP

本质上说这勉强能算 .NET 的一个实现版本，此版本和 .NET Native 在功能上基本重叠。在 Unity3D 上大量采用 IL2CPP 技术



以上就是大部分开发者熟悉的版本，下面是一些大家很少听过的版本

## Rotor

也是 Shared Source CLI 版本，这是基于教育和学术的目的开放的版本，在 2002 年发布的版本。这不是一个用来做商业软件的版本，只是让开发者了解 CLR 细节的版本

## .NET Compact Framework

从 Windows CE 和 Windows Mobile 到 Xbox 360 都有使用过这个版本，这是 .NET 的移动端开始的版本。可以认为 .NET Core 的跨平台就起源在这里，在 .NET Core 有很多代码和实现都从这里拿到

另一个移动版本是 Windows Phone 7.x，Windows Phone 8.x 和 Windows 10 移动版。在 Windows Phone 7.x 是基于 .NET Compact Framework 3.7 的逻辑，而在 Windows Phone 8.x 就基于 .NET Framework 4.5 构建和更改

## Silverlight

这是 .NET 的浏览器开发的开始，是 Web 浏览器插件。在 .NET Framework 2.0 时代开始开发。也被移植到了OSX平台，而 .NET Core 对 OSX 的支持也从这里拿到一些逻辑

提到 SL 这个框架，在这个框架的时代就是 .NET 最黑暗的时代，不过好在现在有 .NET Core 重新撑起来

## .NET Micro Framework

这是给 IOT 等设备准备的开源独立版本，这个版本是最特殊的 .NET 实现版本

## Itanium

其实这个版本我没有考古到名字，这是为 Itanium 处理器特别构建的版本，后面被合入到 .NET Framework 主版本


本文考古内容参考了很多文档，特别感谢 伟民哥翻译的 《.NET内存管理宝典 - 提高代码质量、性能和可扩展性》 这本书提供的详细参考

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
