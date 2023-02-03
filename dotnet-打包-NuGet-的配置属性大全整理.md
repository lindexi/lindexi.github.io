
# dotnet 打包 NuGet 的配置属性大全整理

本文整理 dotnet 打包 CBB 组件为 NuGet 包时可以使用的配置的各个属性

<!--more-->


<!-- CreateTime:2023/1/29 11:43:55 -->

<!-- 发布 -->

本文将会持续更新，可以通过搜 《dotnet 打包 NuGet 的配置属性大全整理 林德熙》 找到我主站的博客，避免各个备份地址陈旧的内容误导

本文更新于：2023.01.31

如更新时间距离当前阅读时间过远，则表示可能你阅读的是转发的或转载的文章，推荐去到我主站的博客，了解更新的知识

<div id="toc"></div>

## 基础知识

在编辑 NuGet 的打包配置属性之前，我期望你了解一些基础知识。了解这部分知识减少一些奇怪的问题和奇怪的决策

基本上使用 dotnet 打包 NuGet 包时，都是通过配置 csproj 项目文件来完成实现功能。其中 csproj 文件有多个版本，当前主力推荐使用的是 SDK 风格的 csproj 格式。可参阅[此博客](https://blog.lindexi.com/post/%E4%BB%8E%E4%BB%A5%E5%89%8D%E7%9A%84%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F%E8%BF%81%E7%A7%BB%E5%88%B0-VS2017-%E6%96%B0%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F.html)提供的方法将旧的 csproj 格式升级到 SDK 风格的 csproj 格式

在 csproj 项目文件里面，支持编辑内容，在 PropertyGroup 标签里面添加属性值。例如加入 TargetFramework 属性之后的 csproj 的代码大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
  </PropertyGroup>

</Project>
```

更多关于 csproj 项目文件格式，请参阅 [理解 C# 项目 csproj 文件格式的本质和编译流程 - walterlv](https://blog.walterlv.com/post/understand-the-csproj )

一些前置知识博客：

- [理解 C# 项目 csproj 文件格式的本质和编译流程 - walterlv](https://blog.walterlv.com/post/understand-the-csproj )
- [项目文件中的已知 NuGet 属性（使用这些属性，创建 NuGet 包就可以不需要 nuspec 文件啦） - walterlv](https://blog.walterlv.com/post/known-nuget-properties-in-csproj )
- [Roslyn 如何了解某个项目在 msbuild 中所有用到的属性以及构建过程](https://blog.lindexi.com/post/Roslyn-%E5%A6%82%E4%BD%95%E4%BA%86%E8%A7%A3%E6%9F%90%E4%B8%AA%E9%A1%B9%E7%9B%AE%E5%9C%A8-msbuild-%E4%B8%AD%E6%89%80%E6%9C%89%E7%94%A8%E5%88%B0%E7%9A%84%E5%B1%9E%E6%80%A7%E4%BB%A5%E5%8F%8A%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B.html )

## CSPROJ 系属性

### PackageId

包的 Id 属性，这是不区分大小写的包标识符，该标识符在 nuget.org 或包所在的私有的 NuGet 源中必须是唯一的。不写默认等同于 AssemblyName 程序集名，即 `$(AssemblyName)` 的值。此 ID 不能包含对于URL无效的空格或字符，且通常遵循.NET命名空间规则

```xml
  <PropertyGroup>
    <PackageId>Foo.Fx</PackageId>
  </PropertyGroup>
```

更多 Id 相关，请参阅 [ID Prefix Reservation Microsoft Learn](https://learn.microsoft.com/en-us/nuget/nuget-org/id-prefix-reservation )

### Title

包的人类阅读友好标题，通常在UI显示中使用，如在 nuget.org 和 Visual Studio 中的包管理器上显示给开发者

默认不写等同于 PackageId 内容

```xml
  <PropertyGroup>
    <Title>标题内容</Title>
  </PropertyGroup>
```

由于存在语言文化相关问题，如果是公开发布的包且期望国际上的朋友使用，则不建议写入中文。此标题限制为 256 个字符长度

### PackageVersion

包版本号，默认不写为 1.0.0 版本号。可使用语义版本号，详细请参阅 [语义版本号（Semantic Versioning） - walterlv](https://blog.walterlv.com/post/semantic-version.html )

```xml
  <PropertyGroup>
    <PackageVersion>1.0.0</PackageVersion>
  </PropertyGroup>
```

与此相关的还有 Version 属性，大部分情况下都采用 Version 属性。此 Version 属性将会被 PackageVersion 所使用。如果没有明确设置 PackageVersion 属性，将会使用已设置的 Version 属性

```xml
  <PropertyGroup>
    <Version>1.0.0</Version>
  </PropertyGroup>
```

默认 dotnet 规范请参阅： [NuGet 包版本引用 Microsoft Learn](https://learn.microsoft.com/zh-cn/nuget/concepts/package-versioning )

如项目没有配置 AssemblyVersion 程序集版本号和 FileVersion 文件版本号，那么默认将使用此 Version 内容作为版本号

如期望自动生成版本号，请参阅 [VisualStudio 2017 项目格式 自动生成版本号](https://blog.lindexi.com/post/VisualStudio-2017-%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F-%E8%87%AA%E5%8A%A8%E7%94%9F%E6%88%90%E7%89%88%E6%9C%AC%E5%8F%B7.html )

### Owners

此包的拥有者，可以不同于作者。大部分作用是在开源组织上，由开源组织拥有此包，然后由具体开发者作为作者。这里的拥有者是可以有多个，推荐多个之间使用分号分割。大部分情况下 Owners 拥有者将和 Company 公司相同

```xml
<Project>
  <PropertyGroup>
    <Company>dotnet-campus</Company>
    <Owners>$(Company)</Owners>
  </PropertyGroup>
</Project>
```

### Company

公司，也可以当成是组织。一般写全商标注册的公司信息。对外可以使用 Owners 写简称

### Authors

作者，表示这个包由谁谁制作。作者不一定拥有此包的所有权，和 Owners 不相同。例如公司雇用你打工，你帮助公司发布的包，自然此包的 所有权 就在公司上，而你自己就是此包的作者

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <PackageId>ClassLibDotNetStandard</PackageId>
    <Version>1.0.0</Version>
    <Authors>your_name</Authors>
    <Company>your_company</Company>
  </PropertyGroup>
</Project>
```

### Copyright

版权信息，官方推荐的格式是 `"Copyright (c) <name/company> <year>` 的格式。正经的包一般都会如此遵守。年份上是可以写范围或固定某个年

```xml
<Project>
  <PropertyGroup>
    <Copyright>Copyright (c) dotnet-campus 2020-2023</Copyright>
  </PropertyGroup>
</Project>
```

详细请参阅 [Package authoring best practices Microsoft Learn](https://learn.microsoft.com/en-us/nuget/create-packages/package-authoring-best-practices )

<!-- 

licenseUrl

projectUrl

Description 描述信息

IsPackable 是否可打包

 -->

### PackageIcon

包的图标，详细请看 [NuGet 如何设置图标](https://blog.lindexi.com/post/NuGet-%E5%A6%82%E4%BD%95%E8%AE%BE%E7%BD%AE%E5%9B%BE%E6%A0%87.html )

现在推荐将图标作为文件放入到包里面，而不是使用外链图片下载地址，解决一些奇怪的地方无法拉到包或泄露隐私





### EmbedAllSources

将源代码嵌入到 PDB 文件里面，此时构建时生成的 PDB 文件里面将包含项目的所有生成相关的源代码。如此可以方便在发布给其他开发者使用时，其他开发者在调试时可以获取到只读的源代码，从而让其他开发者更好进行调试

```xml
  <PropertyGroup>
    <!-- 嵌入源代码到符号文件，方便调试 -->
    <EmbedAllSources>true</EmbedAllSources>
  </PropertyGroup>
```

默认是 false 不将源代码嵌入到符号文件。推荐在源代码无需保护的项目，如内部开源项目或外部开源项目，以及 PDB 不对外发布的项目里，设置此属性为 true 从而将源代码嵌入到 PDB 文件里面，方便调试

详细请参阅 [Roslyn 通过 EmbedAllSources 将源代码嵌入到 PDB 符号文件中方便开发者调试](https://blog.lindexi.com/post/Roslyn-%E9%80%9A%E8%BF%87-EmbedAllSources-%E5%B0%86%E6%BA%90%E4%BB%A3%E7%A0%81%E5%B5%8C%E5%85%A5%E5%88%B0-PDB-%E7%AC%A6%E5%8F%B7%E6%96%87%E4%BB%B6%E4%B8%AD%E6%96%B9%E4%BE%BF%E5%BC%80%E5%8F%91%E8%80%85%E8%B0%83%E8%AF%95.html )

### AllowedOutputExtensionsInPackageBuildOutputFolder

允许哪些扩展名的输出文件带入到 NuGet 包里面

比如说最常用的是将 PDB 文件放入到 NuGet 里面，即可通过此属性设置输出文件里面的 pdb 文件需要被添加到包里面，如以下代码

```xml
  <PropertyGroup>
    <!-- 输出 pdb 文件 NuGet 包 -->
    <AllowedOutputExtensionsInPackageBuildOutputFolder>$(AllowedOutputExtensionsInPackageBuildOutputFolder);.pdb</AllowedOutputExtensionsInPackageBuildOutputFolder>
  </PropertyGroup>
```

此属性只能决定哪些后缀名的文件会打包到 NuGet 包里面，不合适用来决定某些文件需要打包。如果需要特殊指定某些文件，请参阅 [Roslyn 打包自定义的文件到 NuGet 包](https://blog.lindexi.com/post/Roslyn-%E6%89%93%E5%8C%85%E8%87%AA%E5%AE%9A%E4%B9%89%E7%9A%84%E6%96%87%E4%BB%B6%E5%88%B0-NuGet-%E5%8C%85.html )

虽然将 PDB 打包到 NuGet 包里面，有些版本的 VisualStudio 不会自动拷贝 PDB 文件，解决方法请看 [修复 VisualStudio 构建时没有将 NuGet 的 PDB 符号文件拷贝到输出文件夹](https://blog.lindexi.com/post/%E4%BF%AE%E5%A4%8D-VisualStudio-%E6%9E%84%E5%BB%BA%E6%97%B6%E6%B2%A1%E6%9C%89%E5%B0%86-NuGet-%E7%9A%84-PDB-%E7%AC%A6%E5%8F%B7%E6%96%87%E4%BB%B6%E6%8B%B7%E8%B4%9D%E5%88%B0%E8%BE%93%E5%87%BA%E6%96%87%E4%BB%B6%E5%A4%B9.html )



### ContinuousIntegrationBuild

这个属性是比较复杂的，用于 CI 的确定性构建，默认不开。和 [Roslyn 的确定性构建](https://blog.walterlv.com/post/deterministic-builds-in-roslyn.html ) 使用的 Deterministic 属性是不相同的两个概念。此 ContinuousIntegrationBuild 是为了 [SourceLink](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-SourceLink-%E5%B0%86-NuGet-%E9%93%BE%E6%8E%A5%E6%BA%90%E4%BB%A3%E7%A0%81%E5%88%B0-GitHub-%E7%AD%89%E4%BB%93%E5%BA%93.html ) 的功能而引入的。此 [SourceLink](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-SourceLink-%E5%B0%86-NuGet-%E9%93%BE%E6%8E%A5%E6%BA%90%E4%BB%A3%E7%A0%81%E5%88%B0-GitHub-%E7%AD%89%E4%BB%93%E5%BA%93.html ) 功能是在 PDB 符号文件里面，嵌入源代码的下载地址，方便调试的时候获取到源代码，详细请看 [dotnet 使用 SourceLink 将 NuGet 链接源代码到 GitHub 等仓库](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-SourceLink-%E5%B0%86-NuGet-%E9%93%BE%E6%8E%A5%E6%BA%90%E4%BB%A3%E7%A0%81%E5%88%B0-GitHub-%E7%AD%89%E4%BB%93%E5%BA%93.html )

大家都知道，在 PDB 符号文件里面包含的是源代码的绝对路径，在 CI CD 打包服务器上的绝对路径是大部分开发者所不期望的，于是才有了 ContinuousIntegrationBuild 确定性构建的存在。用来实现无论在哪台打包服务器上以及在任何时候打包都会输出相同

这个 ContinuousIntegrationBuild 属性在本机构建调试时，都不应该设置为 true 的值。否则将会丢失本地构建的绝对路径，从而难以自动跳转源代码。只有在 CI 服务器上构建才需要设置

大部分时候设置时，都需要配合设置 SourceRoot 属性

```xml
  <ItemGroup>
    <SourceRoot Include="$(MSBuildThisFileDirectory)"/>
  </ItemGroup>
```

以上代码是推荐放在 `Directory.Build.props` 文件里面，详细关于 Directory.Build.props 请参阅 [Roslyn 使用 Directory.Build.props 文件定义编译](https://blog.lindexi.com/post/Roslyn-%E4%BD%BF%E7%94%A8-Directory.Build.props-%E6%96%87%E4%BB%B6%E5%AE%9A%E4%B9%89%E7%BC%96%E8%AF%91.html ) 和 [Roslyn 使用 Directory.Build.props 管理多个项目配置](https://blog.lindexi.com/post/Roslyn-%E4%BD%BF%E7%94%A8-Directory.Build.props-%E7%AE%A1%E7%90%86%E5%A4%9A%E4%B8%AA%E9%A1%B9%E7%9B%AE%E9%85%8D%E7%BD%AE.html ) 博客

例如在 GitHub 的 CI 构建时，自动设置此属性

```xml
  <PropertyGroup Condition="'$(GITHUB_ACTIONS)' == 'true'">
    <ContinuousIntegrationBuild>true</ContinuousIntegrationBuild>
  </PropertyGroup>
  
  <ItemGroup>
    <SourceRoot Include="$(MSBuildThisFileDirectory)"/>
  </ItemGroup>
```

详细请参阅

[Producing Packages with Source Link - .NET Blog](https://devblogs.microsoft.com/dotnet/producing-packages-with-source-link/ )

[.NET 5 Deterministic Builds & Source Linking Mitchel Sellers](https://mitchelsellers.com/blog/article/net-5-deterministic-builds-source-linking )

[Deterministic Builds in C#](https://gist.github.com/aelij/b20271f4bd0ab1298e49068b388b54ae )

[dotnet/reproducible-builds: Contains the DotNet.ReproducibleBuilds package](https://github.com/dotnet/reproducible-builds )




## 相关文档



[msbuild Roslyn 行为详解](https://blog.lindexi.com/post/msbuild-Roslyn-%E8%A1%8C%E4%B8%BA%E8%AF%A6%E8%A7%A3.html )

[Roslyn 的确定性构建 - walterlv](https://blog.walterlv.com/post/deterministic-builds-in-roslyn.html )

[如何创建一个基于命令行工具的跨平台的 NuGet 工具包 - walterlv](https://blog.walterlv.com/post/create-a-cross-platform-command-based-nuget-tool.html )

[MSBuild properties for Microsoft.NET.Sdk - .NET Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/core/project-sdk/msbuild-props#nuget-metadata-properties )

更多构建打包相关请看[手把手教你写 Roslyn 修改编译](https://blog.lindexi.com/post/roslyn.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。