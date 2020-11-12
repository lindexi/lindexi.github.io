
# WPF 从 dotnet core 3 到 dotnet 5 的变更

本文收藏我所了解的从 dotnet core 3 到 2020.11.10 发布的 dotnet 5 的 WPF 的变更

<!--more-->


<!-- 发布 -->

## 修复坑

应用资源的线程问题

[App resource threading issue by SamBent · Pull Request #3577 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3577 )

[DataGrid.Copy: fail silently if clipboard is locked by SamBent · Pull Request #3576 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3576 )

[FixedPage SOM bugs by SamBent · Pull Request #3575 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3575 )

[ClearType anti-aliasing by SamBent · Pull Request #3570 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3570 )

[Reentrancy when closing ToolTip by SamBent · Pull Request #3568 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3568 )

修复多线程下的 HostVisual 多 UI 渲染问题

[HostVisual threading by SamBent · Pull Request #3567 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3567 )

文本排版渲染相关问题

[Avoid reflow of shaped text by SamBent · Pull Request #3566 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3566 )

[Avoid null item AutomationPeers by SamBent · Pull Request #3565 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3565 )

[Vsp freeze by SamBent · Pull Request #3564 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3564 )

[Ask-Mode: [release/5.0-rc2] Custom intermediate output paths shouldn't break markup compilation by ryalanms · Pull Request #3542 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3542 )

移除校验程序集 Dll 功能，可以提升一点性能，而且因为升级框架之后程序集 dll 文件有一些更改

[Remove VerifyWpfDllSet by agocke · Pull Request #3329 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3329 )

多绑定的判断问题

[Fix range of for statement by Lelary · Pull Request #3221 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3221 )

修复 WM Pointer 消息因为使用屏幕坐标带来的兼容性问题，这个提交是对触摸应用来说十分重要的更改。偷偷说一下，这是我参加怂恿的提交

[Fix raw stylus data to support per-monitor DPI by rladuca · Pull Request #2891 · dotnet/wpf](https://github.com/dotnet/wpf/pull/2891 )

修复触摸相关问题，多窗口多线程问题

[Fix PenIMC Activation Context Handling by rladuca · Pull Request #2890 · dotnet/wpf](https://github.com/dotnet/wpf/pull/2890 )

修复触摸相关问题

[Ensure PimcContexts are eagerly release when no longer needed by rladuca · Pull Request #2851 · dotnet/wpf](https://github.com/dotnet/wpf/pull/2851 )

文本渲染相关问题

[Fix CGlyphRunResource::GetDWriteRenderingMode to choose correct DWriteRenderingMode under ClearType by rladuca · Pull Request #2668 · dotnet/wpf](https://github.com/dotnet/wpf/pull/2668 )

## 性能提升

在上面修复坑上就有部分是能提升性能的，以下更改为完全为了提升性能

[Use faster char based overload of String.IndexOf by davidwrighton · Pull Request #3278 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3278 )

## 调试优化

[Move StackTrace to error branch by SamBent · Pull Request #3574 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3574 )

绑定调试优化

[Add BindingFailed event for diagnostics of binding failures by spadapet · Pull Request #3505 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3505 )

## 更新过时的 API 调用以及修复判空

[Temporarily suppress obsolete API errors to get WPF master building by ryalanms · Pull Request #3659 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3659 )

可能有坑的更改 [https://github.com/dotnet/wpf/pull/3659/files#diff-2a913b21f14e90b5fcc2e41824a0f5df8dafdefb28f01104605bcdb62ceeb7b2R46](https://github.com/dotnet/wpf/pull/3659/files#diff-2a913b21f14e90b5fcc2e41824a0f5df8dafdefb28f01104605bcdb62ceeb7b2R46)

可空逻辑的特性支持，但是这个支持也许会有坑

[Replace null comparisons on non-nullable types that now cause compilation errors, due to the .NET SDK update. by ryalanms · Pull Request #3649 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3649 )

## 构建优化

尽管官方说不会去做跨平台的时候，但是依然有一部分 MR 是用来适配在其他平台进行构建，但是当前依然无法在 Linux 平台完成整个 WPF 的构建

[Fix the casing of 'Shared' in WpfSharedDir by FraGag · Pull Request #3594 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3594 )

[Support PackageReferences in WPF projects by ryalanms · Pull Request #3585 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3585 )

[Fix RuntimeFrameworkReferences for RTM builds by mmitche · Pull Request #3406 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3406 )

[Add CPD strict dependencies for dotnet/installer by mmitche · Pull Request #3372 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3372 )

[adding XamlRuntime msbuild property/metadata by stevenbrix · Pull Request #3332 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3332 )

[Update guardian package by wtgodbe · Pull Request #3274 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3274 )

[[master] Add internal feed build steps by mmitche · Pull Request #3267 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3267 )

[Remove workarounds by wli3 · Pull Request #3248 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3248 )

[Add SDL vars config file by mmitche · Pull Request #3234 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3234 )

[Add the property for MicrosoftWindowsDesktopSdkImported by wli3 · Pull Request #3207 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3207 )

[Recategorize dependencies by mmitche · Pull Request #3188 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3188 )

这个提交看起来是默认支持到 Win7 系统，默认通过 dotnet 构建的输出可执行版本是 Win7 系统支持版本

[Setting default TargetPlatformVersion by sfoslund · Pull Request #3177 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3177 )

[.NET Core WPF Build error on custom BaseIntermediateOutputPath #1718 by ryalanms · Pull Request #3120 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3120 )

[Shuffle property location by wli3 · Pull Request #3111 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3111 )

这个更改有利于在其他平台进行构建

[Replace hardcoded backslashes with Path.DirectorySeparatorChar and fix WinFX casing (daviddenis-stx, Nirmal4G, PaulEremeeff) by ryalanms · Pull Request #3101 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3101 )

[Fix RuntimeFrameworkVersion logic by vatsan-madhavan · Pull Request #2817 · dotnet/wpf](https://github.com/dotnet/wpf/pull/2817 )

这个也是有利于在其他平台进行构建

[Make ArtifactsTmpDir platform-specific by vatsan-madhavan · Pull Request #2525 · dotnet/wpf](https://github.com/dotnet/wpf/pull/2525 )

[Remove usage of Microsoft.NETCore.App.Internal package. by jkoritzinsky · Pull Request #3791 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3791 )

## 安全性

[Fixing Microsoft Security Advisory CVE-2020-0605 : .NET Core Remote Code Execution Vulnerability- Variant (.Net Core 3.1) by arpitmathur · Pull Request #3020 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3020 )

[Fixing Microsoft Security Advisory CVE-2020-0605 : .NET Core Remote Code Execution Vulnerability- Variant (.Net 5.0) by arpitmathur · Pull Request #3019 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3019 )

[Fixing Microsoft Security Advisory CVE-2020-0605 : .NET Core Remote Code Execution Vulnerability (3.1 Fixed PR) by rladuca · Pull Request #2428 · dotnet/wpf](https://github.com/dotnet/wpf/pull/2428 )

[Fixing Microsoft Security Advisory CVE-2020-0606 : .NET Core Remote Code Execution Vulnerability (3.0 Merge Fix) by rladuca · Pull Request #2429 · dotnet/wpf](https://github.com/dotnet/wpf/pull/2429 )

[Microsoft Security Advisory CVE-2020-0606 : .NET Core Remote Code Execution Vulnerability (5.0 PR) by arpitmathur · Pull Request #2430 · dotnet/wpf](https://github.com/dotnet/wpf/pull/2430 )

## 完全开源

这是原本的 MIL 层的代码，以及触摸 PenIMC 底层代码

[Open Sourcing WpfGfx and PenImc by vatsan-madhavan · Pull Request #2553 · dotnet/wpf](https://github.com/dotnet/wpf/pull/2553 )

在这两个代码开放之后，我就水了 [WPF 触摸底层 PenImc 是如何工作的](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%BA%95%E5%B1%82-PenImc-%E6%98%AF%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%E7%9A%84.html ) 和 [WPF 从最底层源代码了解 AllowsTransparency 性能差的原因](https://blog.lindexi.com/post/WPF-%E4%BB%8E%E6%9C%80%E5%BA%95%E5%B1%82%E6%BA%90%E4%BB%A3%E7%A0%81%E4%BA%86%E8%A7%A3-AllowsTransparency-%E6%80%A7%E8%83%BD%E5%B7%AE%E7%9A%84%E5%8E%9F%E5%9B%A0.html )

## 文档和注释

理论上文档和注释不会影响行为

看起来我的提交里面水的文档和注释最多了

[1228498 [ wpf ][ PoliCheck ] - Defect : Term "nuked" by ryalanms · Pull Request #3642 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3642 )

[Update intellisense version by AdamYoblick · Pull Request #3640 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3640 )

[update_intellisense_artifacts by AdamYoblick · Pull Request #3600 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3600 )

[Remove or replace Policheck violations in code comments by ryalanms · Pull Request #3606 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3606 )

[update intellisense artifacts by AdamYoblick · Pull Request #3581 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3581 )

[Fix spelling errors in code comments by lindexi · Pull Request #2841 · dotnet/wpf](https://github.com/dotnet/wpf/pull/2841 )

[LOC CHECKIN  dotnet/wpf  20200403 by v-chmart · Pull Request #2837 · dotnet/wpf](https://github.com/dotnet/wpf/pull/2837 )

## 不知道用来干啥的更改

[#include tuple to matrix_t.hpp by vatsan-madhavan · Pull Request #3056 · dotnet/wpf](https://github.com/dotnet/wpf/pull/3056 )

[Using constants for strings to keep the code consistent by lindexi · Pull Request #2907 · dotnet/wpf](https://github.com/dotnet/wpf/pull/2907 )

## 官方的规划

尽管现在 WPF 所有权在 dotnet 基金会下

但是 WPF 还没全准备好接受来自社区的提交代码，只有在单元测试完成之后才敢接受。而 WPF 的单元测试部分是之前已经写好，但是没有开源。这部分单元测试的开源就是最近从 5 月开始做到现在的内容

当前进度会比原计划慢，客观原因相信大家也知道 However, the global COVID-19 pandemic has caused hiring to be slower than usual. 而在 dotnet 5 更新的时候构建部分还没完全跟上，因此暂时私有版本在 2020.11.12 还不能构建到 dotnet 5 版本

后续将会继续做的修坑和优化性能，然后接受来自社区的更改，走开源管理

更多细节请看官方仓库





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。