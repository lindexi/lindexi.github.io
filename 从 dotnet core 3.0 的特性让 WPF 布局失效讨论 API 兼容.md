# 从 dotnet core 3.0 的特性让 WPF 布局失效讨论 API 兼容

在 dotnet core 3.0 为了支持 IEEE 浮点数计算标准，修改了 Math.Max 的算法，于是在 WPF 的 Track 里面的布局依赖于之前的计算，于是在 dotnet core 3.0 的修改就让布局计算不对了。改动现有 API 的行为会让现有的代码出现不兼容问题，那么要让一个框架能稳定支持升级需要满足什么条件

<!--more-->
<!-- CreateTime:2019/4/17 10:05:35 -->

<!-- csdn -->

在我准备睡的过程，某 [神樹 桜乃](https://github.com/KodamaSakuno) 告诉我现在的 Math.Max 对于 0 和 NaN 的返回和之前不一样，于是在他的帮助下，我找到了[dotnet core 3.0 的新特性，符合 IEEE 标准的浮点计算的提交](https://github.com/dotnet/coreclr/pull/20912 ) 在这个提交里面，作者说明了这个提交将会是改变 API 行为的

于是刚好在 WPF 的 Slider 用到了 [Track](https://referencesource.microsoft.com/#PresentationFramework/src/Framework/System/Windows/Controls/Primitives/Track.cs,484) 的布局，刚好在布局里面就依赖了 Math.Max(0,NaN) 返回 的是 NaN 而不是 0 的坑，详细请看 [神樹的回复](https://github.com/dotnet/wpf/issues/521#issuecomment-483669504)

本来 dotnet core 遵守 IEEE 标准是好的，因为有一个标准可以让多个语言迁移成本降低，但是在 .NET Framework 已经用了这个坑很久了，没有人能说明有多少以前的代码会依赖于这个坑。于是修改 dotnet core 3.0 浮点计算的作者就提议不更改原有的 API 的行为，另外新增新的 API 而不是改变原有的 API 用来兼容现在的代码，请看[Expose Math.MaxNumber and Math.MinNumber functions that don't propagate NaN](https://github.com/dotnet/corefx/issues/36925 )

我十分同意一个稳定的框架的 API 设计是能做到上下兼容的，一个不兼容的 API 只需要满足以下条件

- 接口更改，包含方法或属性名等的变更
- 返回值更改
- 公开属性更改，包括属性类型和属性可访问
- 公开 API 行为修改
- 方法参数变更，包含参数类型和参数个数

作为一个公开的框架，将会有很多历史问题，一个发布出去的 API 将会被很多小伙伴在很多地方使用，如果变更了不兼容的 API 从版本号规范上，需要升级主版本号。在版本号规则，升级主版本号就是表示存在 API 不兼容，基本上就是需要修改现在的代码才能跑起来。而升级次版本号就是表示有新增的 API
 可以不更改原有的代码就可以升级库。也就是只要不是主版本号更改，那么就是愉快升级库而不需要考虑兼容

作为超级多项目引用的基础 dotnet core 库是需要做到上下兼容的，任何很小的 API 不兼容都需要其他很多项目很大的兼容代价

如何设计一个好的框架，请看[好的框架需要好的 API 设计 —— API 设计的六个原则 - walterlv](https://blog.walterlv.com/post/framework-api-design.html )

现在 WPF 和 dotnet core 都是开源的，如果遇到任何的问题都可以在社区上询问

[Some cleanup of the Math functions from #20788 by tannergooding · Pull Request #20912 · dotnet/coreclr](https://github.com/dotnet/coreclr/pull/20912 )

[Expose Math.MaxNumber and Math.MinNumber functions that don't propagate NaN · Issue #36925 · dotnet/corefx](https://github.com/dotnet/corefx/issues/36925 )

[Problem with WPF's Slider style (and a few thoughts on porting DX11 rendering engine to Core3) · Issue #521 · dotnet/wpf](https://github.com/dotnet/wpf/issues/521 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
