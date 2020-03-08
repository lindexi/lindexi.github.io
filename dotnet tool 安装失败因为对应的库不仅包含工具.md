# dotnet tool 安装失败因为对应的库不仅包含工具

在开发 dotnet tool 时，我将规范编码的库作为 dotnet tool 发布，但是在发布的时候本地进行安装提示DotnetToolReference 项目类型仅可包含 DotnetTool 类 型的引用

<!--more-->
<!-- CreateTime:2020/3/1 10:40:47 -->

<!-- 发布 -->

因为我的库本身也是作为可被引用的项目库发布的，在发布的时候我选择的库是多个平台的。多个平台的框架的写法请看 [让一个 csproj 项目指定多个开发框架 - walterlv](https://blog.walterlv.com/post/configure-projects-to-target-multiple-platforms.html )

但是这样写的多框架的包如果作为 dotnet tool 发布，那么将会在安装的时候有下面代码提示

```
error NU1212: dotnetCampus.EncodingNormalior 1.3.0 的项目包组合无效。DotnetToolReference 项目类型仅可包含 DotnetTool 类 型的引用
无法还原工具包。
工具“dotnetcampus.encodingnormalior”安装失败。此故障可能由以下原因导致:

* 你尝试安装预览版，但未使用 --version 选项来指定该版本。
* 已找到具有此名称的包，但是它不是 .NET Core 工具。
* 无法访问所需的 NuGet 源，这可能是由于 Internet 连接问题导致。
* 工具名称输入错误。

有关更多原因(包括强制包命名)，请访问 https://aka.ms/failure-installing-tool
```

英文版本请看下面

```
error NU1212: Invalid project-package combination for awesome-tool 1.0.0. DotnetToolReference project style can only contain references of the DotnetTool type
```

从 [dotnet 手工打一个 dotnet tool 包](https://blog.lindexi.com/post/dotnet-%E6%89%8B%E5%B7%A5%E6%89%93%E4%B8%80%E4%B8%AA-dotnet-tool-%E5%8C%85.html ) 可以知道一个工具包需要在 nuspec 文件里面包含下面代码

```xml
    <packageTypes>
      <packageType name="DotnetTool" />
    </packageTypes>
```

而在压缩包里面存在 Tools 文件夹，而这个文件夹里面有一个 DotnetToolSettings.xml 文件，如果这个要求不满足，那么在 dotnet core 3.1 的版本将安装失败

也就是调试方法是打开打包出来的 NuGet 包，因为本质 NuGet 包就是压缩文件，解压缩，然后看看是否不符合上面两个点，如果不符合那就是本文说的原因了

解决方法是将 csproj 设置为单框架项目，要求这是一个 dotnet core 框架，然后设置 PackAsTool 和 ToolCommandName 属性，详细请看 [dotnet 用 NuGet 将自己的工具作为 dotnet tool 分发](https://blog.lindexi.com/post/dotnet-%E7%94%A8-NuGet-%E5%B0%86%E8%87%AA%E5%B7%B1%E7%9A%84%E5%B7%A5%E5%85%B7%E4%BD%9C%E4%B8%BA-dotnet-tool-%E5%88%86%E5%8F%91.html)

如果我的库同时也是作为依赖库，显然这个方法是不成的，我找了很久没有找到在一个 csproj 里面能做出来的解决方法。但是我可以重新创建一个 csproj 文件，让这个 csproj 文件负责打包工具包。这个还请小伙伴看我的源代码，我原本的文件是 [EncodingNormalior.csproj](https://github.com/dotnet-campus/EncodingNormalior/blob/9d156eb674457a726b09688ccca1e94a072a8afe/EncodingNormalior/EncodingNormalior.csproj ) 文件，然后我将这个文件修改为 SDK Style 的格式，修改后请看 [EncodingNormalior.csproj](https://github.com/dotnet-campus/EncodingNormalior/blob/5a5e85d335da9c4cf1082f40c78f2172a95c9ec7/EncodingNormalior/EncodingNormalior.csproj ) 但是这个源代码存在一个问题是需要作为 dotnet framework 4.6.1 的库。这和 dotnet tool 冲突

于是我在相同文件夹再创建一个 [dotnetCampus.EncodingNormalior.csproj](https://github.com/dotnet-campus/EncodingNormalior/blob/5a5e85d335da9c4cf1082f40c78f2172a95c9ec7/EncodingNormalior/dotnetCampus.EncodingNormalior.csproj ) 就解决了

这个迁移的代码请看 [添加自动打包 by lindexi · Pull Request #27 · dotnet-campus/EncodingNormalior](https://github.com/dotnet-campus/EncodingNormalior/pull/27 )

[dotnet tool install of local tool fails due to NU1212 · Issue #9775 · dotnet/sdk](https://github.com/dotnet/sdk/issues/9775 )

[.NET Core Global Tools and Gotchas](https://natemcmaster.com/blog/2018/02/02/dotnet-global-tool/ )

[dotnet 手工打一个 dotnet tool 包](https://blog.lindexi.com/post/dotnet-%E6%89%8B%E5%B7%A5%E6%89%93%E4%B8%80%E4%B8%AA-dotnet-tool-%E5%8C%85.html ) 

[dotnet 用 NuGet 将自己的工具作为 dotnet tool 分发](https://blog.lindexi.com/post/dotnet-%E7%94%A8-NuGet-%E5%B0%86%E8%87%AA%E5%B7%B1%E7%9A%84%E5%B7%A5%E5%85%B7%E4%BD%9C%E4%B8%BA-dotnet-tool-%E5%88%86%E5%8F%91.html)

[让一个 csproj 项目指定多个开发框架 - walterlv](https://blog.walterlv.com/post/configure-projects-to-target-multiple-platforms.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
