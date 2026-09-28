---
title: WpfRuntime 提前体验尚未发布的 WPF 新功能
description: 本文介绍我维护的实验性 WPF 运行时发行项目 WpfRuntime，它把一套 WPF 的参考程序集、实现程序集以及必要的运行时文件打包成 NuGet 包，让 .NET 8 及以上的 WPF 应用可以提前体验尚未发布的 WPF 新功能。有伙伴有空来帮我试验一下没
tags: WPF
category: 
---

<!-- CreateTime:2026/09/21 07:12:47 -->

<!-- 发布 -->
<!-- 博客 -->

本文内容由人类主导 AI 辅助编写

## 背景

这个项目的起因很简单，官方动作太慢。

WPF 是 .NET Desktop Runtime 的组成部分。一个 WPF 的修复或者改进，要先提交到上游，等官方审核和合并，一个 PR 有时一个月都未必能合并完成，合并之后还要随某个正式 .NET 版本发布，最后等用户升级机器上的 .NET Desktop Runtime，才算真正落到应用上。在它正式发布之前，我没有办法把它交给真实项目运行。

而 WPF 是一个庞大的框架，仅靠少数人手上的几个项目，远不足以把它测完备，需要许许多多真实项目参与进来，才能覆盖到足够多的场景。

另一个原因是希望新功能能有更快的开发节奏，很多改动已经等不及了。

所以这个项目想要提供一条更短的通道：修改 WPF 源码，构建并打包，应用安装后即可体验。

## WpfRuntime 是什么

WpfRuntime 是一个基于开源 `dotnet/wpf` 源码构建的实验性 WPF 运行时发行项目，源码来自微软维护的 <https://github.com/dotnet/wpf>，并受原项目的许可证约束。项目早期也用过 WPF vNext 这个名字。

它把一套 WPF 的参考程序集、实现程序集以及必要的运行时文件打进 NuGet 包，由普通 WPF 应用通过 PackageReference 使用：

```xml
<PackageReference Include="WpfLab.WpfRuntime" Version="1.0.0" />
```

这个包已经发布在 NuGet 上，直接安装即可使用。可以只在入口项目安装，无需在库项目安装。它的影响范围只落在引用了该包的项目内，改动只作用于这一个应用在编译和运行时使用的 WPF 实现，同一个解决方案里的其他 WPF 项目仍然使用机器上已安装的正式 WPF。

## 怎么开始用

如果你有一个 .NET 8 或者更高版本的 WPF 项目，装上包，然后照常构建运行即可。

```
dotnet add package WpfLab.WpfRuntime
```

安装之后，编译 C# 和 XAML 用的就是包里的参考程序集，运行用的是包里的实现程序集，它们会进入应用输出目录，由运行时直接从应用目录加载。

如果你想改的是 WPF 本身，也可以直接从 <https://github.com/WpfLab/WpfRuntime> 拉取代码。原本的 `dotnet/wpf` 仓库构建起来有不小的难度，光是环境和依赖就能劝退不少人，WpfRuntime 在这上面花了大功夫，现在预期用 Visual Studio 2026 打开仓库根目录的解决方案文件就能构建成功，门槛比原来低了很多。

## 怎么确认用的是包里的 WPF

装完之后最值得确认的一件事，是运行时到底加载了哪一份 WPF。

在 Visual Studio 里调试运行你的应用，打开 `调试` → `窗口` → `模块`，在模块列表里找到 `PresentationFramework`、`WindowsBase` 和 `DirectWriteForwarder`，看它们的路径是否落在应用输出目录里，而不是 `C:\Program Files\dotnet` 下面。

如果路径指向应用输出目录，就说明包里的实现生效了。这一步能够稳定成立，靠的是包内程序集与正式共享框架在程序集身份上的隔离，具体做法放在后面说明。

## 目前已经验证过的部分

通过真实的消费测试，目前已经确认这些组合可以正常工作：.NET 8 与 .NET 9、win-x86 与 win-x64、单目标项目与多目标项目，`dotnet build` 与 `dotnet run --no-build` 都能跑通，应用目录中的 WPF 程序集确实被加载，`DirectWriteForwarder` 来自正确位置，文本 shaping 可以执行，XAML 控件可以创建，也在一个大型真实项目上验证过。

framework-dependent 和 self-contained 独立发布这两种方式都验证过。其中 framework-dependent 是更常见的场景，也更容易撞上共享框架的程序集解析问题，所以并没有只靠独立发布把它绕过去。

## 哪些项目适合体验

以下几类项目对验证最有价值：

- 可以随时创建分支进行试验的项目
- 有自动化测试作为保障的 WPF 项目
- 拥有复杂 XAML、大量资源字典和自定义模板的项目
- 使用了第三方控件库的项目
- 大型桌面客户端与内部工具
- Demo 或者开源的 WPF 应用
- 正好遇到某个 WPF 缺陷、希望提前体验修复的项目
- 对性能、DPI、输入、文本或者渲染有特殊要求的项目

在开发分支、测试环境或者可回退的版本上体验最为合适，有精力跟着一起维护是最关键的。

真实的 WPF 项目千差万别，我一个人跑不完，所以想请各位伙伴帮忙，在尽可能多的项目上装上去试一下。有伙伴有空来帮我试验一下 WpfRuntime 库没？包已经发布在 NuGet 上，安装 [WpfLab.WpfRuntime](https://www.nuget.org/packages/WpfLab.WpfRuntime) 就可以试

十分欢迎你来和我报告问题，可以通过 [GitHub Issues](https://github.com/WpfLab/WpfRuntime/issues) 反馈，也可以在博客评论或者私聊我

## 一些实现细节

前面讲的都是使用层面的东西，如果你对里面的原理感兴趣，可以继续往下看。

装完包之后，项目内部其实从编译期就开始了替换。编译 C# 和 XAML 时需要看到 WPF 的公开类型与成员，所以包里带有一套参考程序集，供 C# 编译器、XAML 编译任务和 MSBuild 的引用解析使用。这一层最容易漏掉：如果只在运行期复制新的 DLL，编译期仍然使用正式 SDK 里的旧参考程序集，那么新增的 API 编译不过，C# 和 XAML 还可能看到两套不一致的类型定义，构建结果在运行时就会出现成员缺失。

运行期用的是包里的实现程序集，它们会进入应用输出目录，例如 `WindowsBase.dll`、`System.Xaml.dll`、`PresentationCore.dll` 和 `PresentationFramework.dll`，还有自动化和主题相关的程序集。WPF 并非全部由托管 C# 代码构成，其中还有 C++/CLI 程序集和 native 资产，所以包会按平台把 x86 与 x64 需要的文件一并带上。这些文件还要通过 `buildTransitive` 规则登记到应用输出目录的 `.deps.json` 里，否则运行时不一定认可它们。

其中最值得单独说明的是程序集版本。即使应用输出目录里已经放好了包内的 WPF 文件，在 framework-dependent 模式下，.NET 仍然可能因为程序集身份相同，转而加载正式 `Microsoft.WindowsDesktop.App` 共享框架中的同名程序集。于是就会出现一种很隐蔽的混用：一部分 WPF 程序集来自包内，另一部分同名程序集来自机器上的共享框架，输出目录看起来文件齐全，实际运行时却没有全部加载；一旦双方的 ABI 或 API 有差异，就会抛出 `MissingMethodException` 这类异常。

因此 WpfRuntime 自行构建的 WPF 运行时程序集，包括相关的 C++/CLI 程序集，统一使用了 `42.42.42.42424` 这个程序集版本。它不是 NuGet 包版本，而是用于区分程序集身份的内部版本，让包内程序集与共享框架中的同名程序集不再被视为同一份实现，运行时只能从应用目录加载。打包之前还会读取程序集的 PE 元数据，检查这些程序集是否都用了这个统一版本，只要有一个不一致，就会阻止打包。

上面这些事情都由仓库里的 Builder 负责，它按顺序构建 WPF 项目，分别收集参考程序集、实现程序集、主题程序集和 native 资产，生成包结构和 `buildTransitive` 规则，校验运行时资产与程序集版本，然后创建一个隔离的消费项目，像普通开发者那样把刚构建出来的包装一遍、跑一遍，再核对加载路径、MVID 和文件哈希。这也是前面提到打开解决方案就能构建成功的原因。

Builder 还带一个 `relay-pr` 能力，可以把一个正在评审的 `dotnet/wpf` PR 搬运过来，用同样的流程构建成试验包，让真实项目提前体验，再把结果反馈给改动作者。

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
