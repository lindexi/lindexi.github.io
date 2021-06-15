# WPF 框架开发 调试和开发 XAML 构建过程的 PresentationBuildTasks 方法

阅读本文，你可以了解如何编写开发和调试 XAML 构建为 Baml 和 g.cs 文件的过程和工具。本文也适合想要了解 WPF 的 XAML 构建过程的开发者阅读，本文提供了可以断点调试 WPF 的 XAML 构建过程的方法和代码

<!--more-->
<!-- CreateTime:2021/6/9 20:01:02 -->

<!-- 发布 -->
<!-- 标签：Roslyn,MSBuild,编译器,WPF,XAML -->

本文非新手友好，有大量构建和预编译知识，请在阅读本文之前自行了解这部分知识。更多请看 [手把手教你写 Roslyn 修改编译](https://blog.lindexi.com/post/roslyn.html )

在 WPF 中，构建 XAML 用的是 PresentationBuildTasks 任务进行构建，核心使用的是 dotnet sdk 提供的构建调度功能，将 XAML 的构建调度到 PresentationBuildTasks 任务，由此工具进行构建。使用 PresentationBuildTasks 任务可以构建出 Baml 和 g.cs 等文件

在 WPF 开源仓库里面，包含了 PresentationBuildTasks 的所有源代码。在 dotnet sdk 里面，包含了调试 XAML 构建的后门，允许开发者指定 PresentationBuildTasks 为自己的开发版本

下面以调试 Walterlv.Demo.XamlProperties 测试项目的 XAML 构建过程作为例子，告诉大家如何让 dotnet 在构建 WPF 项目时，使用自定义的 PresentationBuildTasks 任务进行构建

先在 Walterlv.Demo.XamlProperties.csproj 文件里面添加如下代码

```xml
  <PropertyGroup>
    <_PresentationBuildTasksTfm Condition="'$(MSBuildRuntimeType)' == 'Core'">netcoreapp2.1</_PresentationBuildTasksTfm>
    <_PresentationBuildTasksTfm Condition="'$(MSBuildRuntimeType)' != 'Core'">net472</_PresentationBuildTasksTfm>
    <_PresentationBuildTasksAssembly>$(MSBuildThisFileDirectory)..\PresentationBuildTasks\bin\Debug\$(_PresentationBuildTasksTfm)\PresentationBuildTasks.dll</_PresentationBuildTasksAssembly>
  </PropertyGroup>
```

以上的代码的 `_PresentationBuildTasksAssembly` 属性需要修改为你自己的 PresentationBuildTasks 代码构建出来的输出文件路径。在 dotnet 里面，如果在 VisualStudio 里面，那么将加载 .NET Framework 4.7.2 版本的 PresentationBuildTasks.dll 的文件。如果是在命令行执行 dotnet build 命令，此时将加载 .NET Core 2.1 的 PresentationBuildTasks.dll 的文件。此部分知识请参阅 [从零开始制作 NuGet 源代码包（全面支持 .NET Core / .NET Framework / WPF 项目） - walterlv](https://blog.walterlv.com/post/build-source-code-package-for-wpf-projects.html ) 和 [在项目文件 / MSBuild / NuGet 包中编写扩展编译的时候，正确使用 props 文件和 targets 文件 - walterlv](https://blog.walterlv.com/post/write-msbuild-codes-into-props-or-targets.html ) 和 [如何创建一个基于 MSBuild Task 的跨平台的 NuGet 工具包 - walterlv](https://blog.walterlv.com/post/create-a-cross-platform-msbuild-task-based-nuget-tool.html )

而 PresentationBuildTasks 的代码可以从 WPF 开源仓库里面的 `src\Microsoft.DotNet.Wpf\src\PresentationBuildTasks` 文件夹拿到代码，只是这里面的代码构建需要做一些配置

我给大家提供了我的版本，此版本包含了 Walterlv.Demo.XamlProperties 测试项目本身，使用方法是将我的代码拉到你的本地。请在使用时，安装好 .NET 5 或更新版本的 SDK 然后将你的需要测试的 XAML 文件和代码加入到 Walterlv.Demo.XamlProperties 测试项目里面。先打开 PresentationBuildTasks.sln 项目，接着使用命令行 dotnet build 构建 Walterlv.Demo.XamlProperties 测试项目

构建时将会弹出 VisualStudio 附加进程调试窗口，选择使用 PresentationBuildTasks.sln 所在的 VisualStudio 进行调试，下一步按下 F10 就可以看到 PresentationBuildTasks 的源代码

获取以上调试版本的方法是先创建一个本地源代码文件夹，此文件夹是一个空文件夹，用来从 GitHub 上拉我的代码。先使用命令行进入到此空文件夹，接下来输入以下代码用来拉代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin e9982404d4b51f184f483ba8663ee160befdc8e8
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

以上代码包含了作为测试项目的 Walterlv.Demo.XamlProperties.csproj 文件，以及我从 WPF 项目里面拷贝的 PresentationBuildTasks 代码。对比放在 WPF 项目的 PresentationBuildTasks 的代码，我提供的测试代码是没有 WPF 项目的依赖的，不仅构建方便，同时构建速度也快

当然缺点是没有更上 WPF 的源代码版本，需要大家自己手动去拷贝最新的代码

本文提供的代码，其实是准备调试 [AttachedProperty cannot be assigned in a XAML file if it is declared in the same project. · Issue #4544 · dotnet/wpf](https://github.com/dotnet/wpf/issues/4544 )

欢迎大家参与 WPF 框架的开发

当前的 WPF 在 [https://github.com/dotnet/wpf](https://github.com/dotnet/wpf) 完全开源，使用友好的 MIT 协议，意味着允许任何人任何组织和企业任意处置，包括使用，复制，修改，合并，发表，分发，再授权，或者销售。在仓库里面包含了完全的构建逻辑，只需要本地的网络足够好（因为需要下载一堆构建工具），即可进行本地构建

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
