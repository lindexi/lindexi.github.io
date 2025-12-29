# dotnet 打包 NuGet 的配置属性大全整理

本文整理 dotnet 打包 CBB 组件为 NuGet 包时可以使用的配置的各个属性

<!--more-->
<!-- CreateTime:2023/1/29 11:43:55 -->
<!-- 博客 -->
<!-- 发布 -->

本文将会持续更新，可以通过搜 《dotnet 打包 NuGet 的配置属性大全整理 林德熙》 找到我主站的博客，避免各个备份地址陈旧的内容误导

本文更新于：2025.07.01

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
    <TargetFramework>net9.0</TargetFramework>
  </PropertyGroup>

</Project>
```

更多关于 csproj 项目文件格式，请参阅 [理解 C# 项目 csproj 文件格式的本质和编译流程 - walterlv](https://blog.walterlv.com/post/understand-the-csproj )

一些前置知识博客：

- [理解 C# 项目 csproj 文件格式的本质和编译流程 - walterlv](https://blog.walterlv.com/post/understand-the-csproj )
- [项目文件中的已知 NuGet 属性（使用这些属性，创建 NuGet 包就可以不需要 nuspec 文件啦） - walterlv](https://blog.walterlv.com/post/known-nuget-properties-in-csproj )
- [Roslyn 如何了解某个项目在 msbuild 中所有用到的属性以及构建过程](https://blog.lindexi.com/post/Roslyn-%E5%A6%82%E4%BD%95%E4%BA%86%E8%A7%A3%E6%9F%90%E4%B8%AA%E9%A1%B9%E7%9B%AE%E5%9C%A8-msbuild-%E4%B8%AD%E6%89%80%E6%9C%89%E7%94%A8%E5%88%B0%E7%9A%84%E5%B1%9E%E6%80%A7%E4%BB%A5%E5%8F%8A%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B.html )

## 相关博客

以下是我记录的一些工具博客，便于查阅

- [项目文件中的已知 NuGet 属性（使用这些属性，创建 NuGet 包就可以不需要 nuspec 文件啦） - walterlv](https://blog.walterlv.com/post/known-nuget-properties-in-csproj )
- [项目文件中的已知属性（知道了这些，就不会随便在 csproj 中写死常量啦） - walterlv](https://blog.walterlv.com/post/known-properties-in-csproj.html )
- [msbuild 项目文件常用判断条件](https://blog.lindexi.com/post/msbuild-%E9%A1%B9%E7%9B%AE%E6%96%87%E4%BB%B6%E5%B8%B8%E7%94%A8%E5%88%A4%E6%96%AD%E6%9D%A1%E4%BB%B6.html )

更进阶内容：

- [msbuild Roslyn 行为详解](https://blog.lindexi.com/post/msbuild-Roslyn-%E8%A1%8C%E4%B8%BA%E8%AF%A6%E8%A7%A3.html )

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

与此相关的还有 Version 属性，大部分情况下都采用 Version 属性。此 Version 属性将会被 PackageVersion 和程序集版本号等所使用，用途较广。除非确实不想要让包版本号确实和程序集版本号相同，否则推荐使用 Version 属性。如果没有明确设置 PackageVersion 属性，将会使用已设置的 Version 属性

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

作者，表示这个包由谁制作。作者不一定拥有此包的所有权，和 Owners 不相同。例如公司雇用你打工，你帮助公司发布的包，自然此包的 所有权 就在公司上，而你自己就是此包的作者

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

版权信息，官方推荐的格式是 `Copyright (c) <name/company> <year>` 的格式。正经的包一般都会如此遵守。年份上是可以写范围或固定某个年

```xml
<Project>
  <PropertyGroup>
    <Copyright>Copyright (c) dotnet-campus 2020-2023</Copyright>
  </PropertyGroup>
</Project>
```

详细请参阅 [Package authoring best practices Microsoft Learn](https://learn.microsoft.com/en-us/nuget/create-packages/package-authoring-best-practices )

如果想要保持每次打包都是最新年份，不用每一年都手动更新，那可以使用 `$([System.DateTime]::Now.ToString(`yyyy`))` 来表示当前年份，如以下代码

```xml
<Copyright>Copyright (c) dotnet-campus 2020-$([System.DateTime]::Now.ToString(`yyyy`))</Copyright>
```

加入以上代码之后，即可每次打包都设置版权信息为当前的年份

而有些包则是先日期再公司名，如以下代码，这也是可以接受的。详细请参阅 [规范的版权Copyright说明怎么写？ 赵智功律师回答内容 - 知乎](https://www.zhihu.com/question/19916364 )

```xml
<Copyright>Copyright (c) 玄年-玄月 Metaphysical Algorithm Co.,Ltd</Copyright>
```

中间的 `(c)` 的正确表述应该是 UTF-8 编码的 169 号（0xA9）字符，这个字符经常存在编码问题。好在 csproj 等都采用 XML 格式，可以直接使用 XML 编码 `&#169;` 表示，如

```xml
<Copyright>Copyright &#169; dotnet-campus 2020-$([System.DateTime]::Now.ToString(`yyyy`))</Copyright>
```

#### CopyrightSlim

只是 Copyright 的较短版本，默认不设置将采用 Copyright 的值

较短版本一般来说采用的是公司的简称而不是全称，以及省略 `All Rights Reserved` 内容，其他信息不变，格式如下

```
Copyright (c) <name/company简称> <year>
```

如将 `Metaphysical Algorithm Co.,Ltd, All Rights Reserved.` 换成 `Metaphysical Algorithm` 简写

注： 这里的 `Metaphysical Algorithm Co.,Ltd` 是我根据太子最喜欢的 `玄学算法公司` 杜撰而来的，如有相同，纯属巧合

### PackageLicenseExpression

许可证信息，可以在 Copyright 不存在时勉强当成版权信息。可以打入的是当前的包使用的是什么协议进行许可，比如当前是给一个 MIT 协议开源的仓库进行打包的，可以使用如下设置当前的 NuGet 包使用最友好的 MIT 协议

```xml
<PropertyGroup>
    <PackageLicenseExpression>MIT</PackageLicenseExpression>
</PropertyGroup>
```

如果这个包是一个混合包，包含多个协议，比如 MIT 和 Apache-2.0 协议，可以这样写

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <PackageLicenseExpression>MIT OR Apache-2.0</PackageLicenseExpression>
  </PropertyGroup>
</Project>
```

### PackageLicenseFile

以上的 PackageLicenseExpression 是一个简单的写法，适合用在一些明确当前使用类型的许可证。但如果自己的许可证有些特殊，比如是公司的法务写的许可证，需要特殊的许可证文件，那可以使用 PackageLicenseFile 属性。通过 PackageLicenseFile 属性设置采用打入到 NuGet 包的哪个文件作为许可证文件

```xml
<PropertyGroup>
    <PackageLicenseFile>LICENSE.txt</PackageLicenseFile>
</PropertyGroup>

<ItemGroup>
    <None Include="..\LICENSE.txt" Pack="true" PackagePath=""/>
</ItemGroup>
```

这里需要明确的是 PackageLicenseExpression 和 PackageLicenseFile 以及不被推荐使用的 PackageLicenseUrl 三个只能同时存在其中一个

额外说明的是，对于许多仓库的 LICENSE 许可证文件来说，都是没有带后缀名的。由于历史原因，在 NuGet 里面对于没有后缀名的都是会被当成是文件夹。为了能够正确的进行打包，就必须带上 PackagePath 属性，如下面代码例子

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFrameworks>netstandard2.0</TargetFrameworks>
    <PackageLicenseFile>LICENSE</PackageLicenseFile>
  </PropertyGroup>

  <ItemGroup>
    <None Include="LICENSE" Pack="true" Visible="false" PackagePath=""/>
  </ItemGroup>
  
</Project>
```

以上的代码里面核心的逻辑在于 `PackagePath=""` 设置了无后缀名的 LICENSE 不是文件

上面代码是从 https://github.com/NuGet/Samples/blob/ec30a2b7c54c2d09e5a476444a2c7a8f2f289d49/PackageLicenseFileExtensionlessExample/PackageLicenseFileExtensionlessExample.csproj#L1 拷贝的

### PackageReadmeFile

打包到 NuGet 包里面的自述文件，一般可以将仓库里面的 README.md 文件打进来，如以下例子

```xml
<PropertyGroup>
    ...
    <PackageReadmeFile>README.md</PackageReadmeFile>
    ...
</PropertyGroup>

<ItemGroup>
    ...
    <None Include="..\..\README.md" Link="README.md" Pack="True" PackagePath="\"/>
    ...
</ItemGroup>
```

这里包括两个方面的内容，第一个是在 PropertyGroup 里面使用 PackageReadmeFile 属性标明 README.md 文件，然后在 ItemGroup 里面设置打包到 NuGet 包里面的是哪个文件当成 README.md 文件

常见写法也写在 [Directory.Build.props](https://blog.lindexi.com/post/Roslyn-%E4%BD%BF%E7%94%A8-Directory.Build.props-%E7%AE%A1%E7%90%86%E5%A4%9A%E4%B8%AA%E9%A1%B9%E7%9B%AE%E9%85%8D%E7%BD%AE.html ) 里面，这样可以复用仓库的 README.md 文件，大概的代码如下

```xml
  <PropertyGroup>
    <SlnDir>$(MSBuildThisFileDirectory)</SlnDir>
  </PropertyGroup>

  <!-- 以下是打 NuGet 包相关辅助方法 -->
  <PropertyGroup>
    <PackageReadmeFile>README.md</PackageReadmeFile>
  </PropertyGroup>
  <ItemGroup>
    <!-- 嵌入 README 文件 -->
    <None Include="$(SlnDir)README.md" Pack="true" PackagePath="\" Visible="false"/>
  </ItemGroup>
```

以上代码添加的 `Visible="false"` 用于让 `README.md` 文件不要在 VisualStudio 的解决方案窗格的项目里面显示

此属性为 .NET SDK 5.0.300 以及以上版本才能提供支持

<!-- 

licenseUrl

projectUrl

Description 描述信息

<RepositoryUrl>https://github.com/dotnet/wpf</RepositoryUrl>

<RepositoryType>git</RepositoryType>

<PackageProjectUrl>https://github.com/dotnet/wpf</PackageProjectUrl>

 -->

### PackageIcon

包的图标，详细请看 [NuGet 如何设置图标](https://blog.lindexi.com/post/NuGet-%E5%A6%82%E4%BD%95%E8%AE%BE%E7%BD%AE%E5%9B%BE%E6%A0%87.html )

现在推荐将图标作为文件放入到包里面，而不是使用外链图片下载地址，解决一些奇怪的地方无法拉到包或泄露隐私

大概的例子如下

```xml
<PropertyGroup>
    ...
    <PackageIcon>Icon.png</PackageIcon>
    ...
</PropertyGroup>

<ItemGroup>
    ...
    <None Include="..\Images\Icon.png" Pack="true" PackagePath="\"/>
    ...
</ItemGroup>
```

建议使用分辨率为 128x128 的图像，支持 JPEG 和 PNG 文件格式，图片大小限制 1MB 以内

## 打包控制

### GeneratePackageOnBuild

生成的时候，构建出 NuGet 包。没有开启此属性时，是需要有额外的打包过程，例如 `dotnet pack` 或者在 VisuslStudio 里右击打包。开启此属性之后，每次构建都会输出 NuGet 包。实际测试是开启此属性对生成的性能影响很小

```xml
  <PropertyGroup>
    <GeneratePackageOnBuild>true</GeneratePackageOnBuild>
  </PropertyGroup>
```

### IsPackable

用于设置项目是否可以被打包，默认是 true 表示项目可以打包，如果设置为 false 禁用则不打包 NuGet 包。可以用在如单元测试等项目，设置这些项目不要输出 NuGet 包

```xml
  <PropertyGroup>
    <IsPackable>false</IsPackable>
  </PropertyGroup>
```

注：对于 ASP.NET Core 应用项目，在 SDK 里面默认设置了 IsPackable 为 false 的值。也就是说在 ASP.NET Core 应用项目上默认 IsPackable 就是 false 的值

对于单元测试项目，还会额外配置 IsTestProject 属性

```xml
  <PropertyGroup>
    <IsPackable>false</IsPackable>
    <IsTestProject>true</IsTestProject>
  </PropertyGroup>
```

### GenerateDocumentationFile

设置是否在生成的时候，同时生成注释 XML 文件。此属性设置之后，将会自动将注释 XML 文件输出到 NuGet 里

```xml
  <PropertyGroup>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
  </PropertyGroup>
```

在 dotnet 里面，代码上的公开成员，如公开的方法公开的属性等，的注释是存放在一个和程序集同名后缀为 XML 的文件里面。开启 GenerateDocumentationFile 属性，即可在生成过程，生成注释 XML 文件。在拥有此 XML 文件，即可让 VisualStudio 等 IDE 可以自动提示引用库的代码注释，方便让开发者了解调用库的各个成员的含义。进行 NuGet 发布的时候，将注释的 XML 文件带到 NuGet 包里面，可以方便让引用此 NuGet 包的项目获取到库的代码注释


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

设置 EmbedAllSources 嵌入源代码到符号文件时，可能会遇到打本地包引用的时候，无法找到本地磁盘路径的代码文件，而是会显示进入嵌入到符号文件的代码文件，导致调试困难。可通过添加判断代码，仅在非 Debug 模式下才嵌入

```xml
    <!-- 不要在 debug 开启 EmbedAllSources 或 EmbedUntrackedSources：
         1. NuGet 包会提示包含未追踪的源，但实际列出的未追踪的源是空的（所以其实都已经追踪了？）
         2. 如果采用此属性将源嵌入，会导致 JetBrians Rider 调试时使用嵌入的源而不是仓库中的源，这会导致无法使用断点等一系列依赖于 pdb 源的功能。-->
    <EmbedAllSources Condition="'$(Configuration)' != 'Debug'">true</EmbedAllSources>
```

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

### IncludeSymbols

设置是否输出符号文件，用于制作符号包，通常和 SymbolPackageFormat 配合使用

```xml
  <PropertyGroup>
    <!-- 输出符号文件 -->
    <IncludeSymbols>true</IncludeSymbols>
    <SymbolPackageFormat>snupkg</SymbolPackageFormat>
  </PropertyGroup>
```

### SymbolPackageFormat

输出的符号文件的格式，符号文件有两个输出格式，文件名规范不相同

- `.symbols.nupkg` ： 默认的文件后缀。兼容性好，但是存在冲突。比如真有一个叫 `Xx.Symbols` 项目就凉凉。此格式已被淘汰
- `.snupkg` ： 专门定义的符号包格式，可以只包含符号 PDB 文件

```xml
  <PropertyGroup>
    <!-- 输出符号文件 -->
    <IncludeSymbols>true</IncludeSymbols>
    <SymbolPackageFormat>snupkg</SymbolPackageFormat>
  </PropertyGroup>
```

官方文档： [How to publish NuGet symbol packages using the new symbol package format '.snupkg' Microsoft Learn](https://learn.microsoft.com/en-us/nuget/create-packages/symbol-packages-snupkg )

使用 `.snupkg` 格式对应在 `.nuspec` 的配置是

```xml
<packageTypes>
   <packageType name="SymbolsPackage"/>
</packageTypes>
```

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

[dotnet 使用 SourceLink 将 NuGet 链接源代码到 GitHub 等仓库](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-SourceLink-%E5%B0%86-NuGet-%E9%93%BE%E6%8E%A5%E6%BA%90%E4%BB%A3%E7%A0%81%E5%88%B0-GitHub-%E7%AD%89%E4%BB%93%E5%BA%93.html )

[Producing Packages with Source Link - .NET Blog](https://devblogs.microsoft.com/dotnet/producing-packages-with-source-link/ )

[.NET 5 Deterministic Builds & Source Linking Mitchel Sellers](https://mitchelsellers.com/blog/article/net-5-deterministic-builds-source-linking )

[Deterministic Builds in C#](https://gist.github.com/aelij/b20271f4bd0ab1298e49068b388b54ae )

[dotnet/reproducible-builds: Contains the DotNet.ReproducibleBuilds package](https://github.com/dotnet/reproducible-builds )



### IncludeBuildOutput

默认是 true 的值，如果指定为 false 那么项目编译输出的 dll 文件将不会被打包到 NuGet 包中。可以用来配置将项目构建输出的 DLL 不要自动打入到 nupkg 的 lib 文件夹下

这个属性一般会用在分析器项目或者是工具 NuGet 包里

一般和 `<IsTool>true</IsTool>` 进行二选一使用。使用 `<IncludeBuildOutput>false</IncludeBuildOutput>` 能够实现更高的定制化，与 IsTool 不同之处在于 IsTool 属性设置为 true 的值将会让输出 NuGet 包的 `tools` 文件夹里面

设置 `<IncludeBuildOutput>false</IncludeBuildOutput>` 时，常会带上 `<NoPackageAnalysis>true</NoPackageAnalysis>` 设置，用于去掉 NU5128 警告。这是因为设置 IncludeBuildOutput 为 false 时，将不会在 NuGet 包的 lib 文件夹包含当前此项目的输出，如果没有额外设置 lib 文件夹内容，则会收到警告

设置此属性时，通常也会考虑再带上 SuppressDependenciesWhenPacking 属性

### DevelopmentDependency

这是一个仅开发阶段使用的 NuGet 包，默认是 false 的值。如果设置为 true 即可在安装此 NuGet 包后自动配置为不传递依赖。可用在工具类型的 NuGet 包上，让工具包只对当前安装的项目生效，不会传递给所引用的项目

详细请参阅 [帮助官方 NuGet 解掉 Bug，制作绝对不会传递依赖的 NuGet 包 - walterlv](https://blog.walterlv.com/post/prevent-nuget-package-been-depended )

### SuppressDependenciesWhenPacking

默认情况下，会将 TargetFramework 作为 NuGet 的框架依赖。有一些工具库之类的，想要适用范围更广，不想通过修改 TargetFrameworks 包含旧框架实现。可以设置 SuppressDependenciesWhenPacking 为 true 的值

```xml
    <!-- 
      配置为无依赖。即避免带上 TargetFramework=netstandard2.0 的限制
      配合 IncludeBuildOutput=false 即可让任意项目引用，无视目标框架
    -->
    <SuppressDependenciesWhenPacking>true</SuppressDependenciesWhenPacking>
```

一般而言，设置了 SuppressDependenciesWhenPacking 为 true 会搭配设置 IncludeBuildOutput 属性，反之不然

设置了 SuppressDependenciesWhenPacking 为 true 之后，可见此时的 NuGet 包的 Dependencies 是空白，在 NuGet Package Explorer 里面将显示 No Dependencies 表示无依赖。此时的 NuGet 包可被任意 TargetFramework 框架所引用

### EnablePackageValidation

启用基线包版本处理兼容性问题

用于判断当前版本是否能够兼容旧版本，对于大型项目来说，此功能能够更好地保持 API 兼容性

[dotnet 根据基线包版本实现库版本兼容](https://blog.lindexi.com/post/dotnet-%E6%A0%B9%E6%8D%AE%E5%9F%BA%E7%BA%BF%E5%8C%85%E7%89%88%E6%9C%AC%E5%AE%9E%E7%8E%B0%E5%BA%93%E7%89%88%E6%9C%AC%E5%85%BC%E5%AE%B9.html )

## 已知属性

以下内容是对 [项目文件中的已知属性（知道了这些，就不会随便在 csproj 中写死常量啦） - walterlv](https://blog.walterlv.com/post/known-properties-in-csproj.html ) 的更多补充

### NuGetPackageRoot

表示的是当前的 NuGet 的 Package 文件夹路径，可以用来拼接获取到对应的 NuGet 包文件路径。一般路径是 `C:\Users\【用户名】\.nuget\packages` 文件夹

拼接 NuGet 包路径的例子如下

```xml
    <None Include="$(NuGetPackageRoot)\sharpziplib\1.2.0\lib\netstandard2.0\ICSharpCode.SharpZipLib.dll" Link="ICSharpCode.SharpZipLib.dll">
      <Pack>true</Pack>
      <PackagePath>tools\netstandard2.0\</PackagePath>
    </None>
```

如果已经是明确知道有安装了某个包，还可以使用下文提及的 GeneratePathProperty 方法

### GeneratePathProperty

获取 NuGet 包还原到的本地路径，如以下示例代码

```xml
<PackageReference Include="Lindexi.Package" Version="1.2.3" GeneratePathProperty="true"/>

<Warning Text="Lindexi.Package Path=$(PkgLindexi_Package)" />
```

输出警告内容大概如下

```
Lindexi.Package Path=C:\Users\lindexi\.nuget\packages\lindexi.package\1.2.3
```

具体写法是在需要获取 NuGet 包路径的 PackageReference 标记 `GeneratePathProperty="true"` 属性

获取时的格式是 `$(Pkg包名)` 包名需要将 `.` 替换为下划线

详细请参阅 <https://learn.microsoft.com/en-us/nuget/consume-packages/package-references-in-project-files#generatepathproperty>

## 常写的代码片

### 输出和包含 targets 文件

```xml
  <ItemGroup>
    <None Include="Build\package.targets" Pack="True" PackagePath="\build\$(PackageId).targets" />
    <None Include="Build\package.props" Pack="True" PackagePath="\build\$(PackageId).props" />
  </ItemGroup>
```

详细请参阅 [Roslyn 打包自定义的文件到 NuGet 包](https://blog.lindexi.com/post/Roslyn-%E6%89%93%E5%8C%85%E8%87%AA%E5%AE%9A%E4%B9%89%E7%9A%84%E6%96%87%E4%BB%B6%E5%88%B0-NuGet-%E5%8C%85.html )

包含穿透依赖传递的 BuildTransitive 文件夹：

```xml
  <ItemGroup>
     <None Include="BuildTransitive\Package.targets" Pack="True" PackagePath="\buildTransitive\$(PackageId).targets" /> 
     <None Include="BuildTransitive\Package.props" Pack="True" PackagePath="\buildTransitive\$(PackageId).props" /> 
  </ItemGroup>
```

详细请参阅 [Roslyn 打包 NuGet 包 BuildTransitive 文件夹用于穿透依赖传递拷贝文件](https://blog.lindexi.com/post/Roslyn-%E6%89%93%E5%8C%85-NuGet-%E5%8C%85-BuildTransitive-%E6%96%87%E4%BB%B6%E5%A4%B9%E7%94%A8%E4%BA%8E%E7%A9%BF%E9%80%8F%E4%BE%9D%E8%B5%96%E4%BC%A0%E9%80%92%E6%8B%B7%E8%B4%9D%E6%96%87%E4%BB%B6.html )

### 判断框架和平台

```xml
  <PropertyGroup Condition="'$(Configuration)|$(TargetFramework)|$(Platform)'=='Debug|netstandard1.5|AnyCPU'">
    <PlatformTarget>AnyCPU</PlatformTarget>
  </PropertyGroup>
```

更多判断逻辑请参阅 [msbuild 项目文件常用判断条件](https://blog.lindexi.com/post/msbuild-%E9%A1%B9%E7%9B%AE%E6%96%87%E4%BB%B6%E5%B8%B8%E7%94%A8%E5%88%A4%E6%96%AD%E6%9D%A1%E4%BB%B6.html )

## Target 时机

### 动态加入打包到 NuGet 包的文件时机

可在 `_GetPackageFiles` 这个 Target 前执行，在此执行加入 Nuget 打包文件才是有效，在这个时机之后将会无效，如以下代码

```xml
  <ItemGroup>
    <None Include="build\package.targets" Pack="True" PackagePath="\build\$(PackageId).targets" />
  </ItemGroup>

  <Target Name="FooIncludeAllDependencies" BeforeTargets="_GetPackageFiles">
    <ItemGroup>
      <None Include="..\Foo\Foo.dll" Pack="True" PackagePath="analyzers\dotnet\cs" />
    </ItemGroup>
  </Target>
```

以上代码的两个加入打包的文件都会成功都被加入打包。更多请参阅 [Roslyn 打包自定义的文件到 NuGet 包](https://blog.lindexi.com/post/Roslyn-%E6%89%93%E5%8C%85%E8%87%AA%E5%AE%9A%E4%B9%89%E7%9A%84%E6%96%87%E4%BB%B6%E5%88%B0-NuGet-%E5%8C%85.html )

更多请看 [msbuild Roslyn 行为详解](https://blog.lindexi.com/post/msbuild-Roslyn-%E8%A1%8C%E4%B8%BA%E8%AF%A6%E8%A7%A3.html )

### 打包分析器进入 NuGet 包

在分析器项目里面将自身打入到 NuGet 的 `analyzers/dotnet/cs` 文件夹，作为分析器的存在

```xml
  <Target Name="AddOutputDllToNuGetAnalyzerFolder" BeforeTargets="_GetPackageFiles">
    <!-- 
      以下这句 ItemGroup 不能放在 Target 外面。否则首次构建之前 $(OutputPath)\$(AssemblyName).dll 是不存在的
      这里需要选用在 _GetPackageFiles 之前，确保在 NuGet 收集文件之前，标记将输出的 dll 放入到 NuGet 的 analyzers 文件夹下
    -->
    <ItemGroup>
      <None Include="$(OutputPath)\$(AssemblyName).dll"
            Pack="true"
            PackagePath="analyzers/dotnet/cs"
            Visible="false" />
    </ItemGroup>
  </Target>
```

常写的分析器 NuGet 包配置属性如下

```xml
    <!-- 
      配置为无依赖。即避免带上 TargetFramework=netstandard2.0 的限制
      配合 IncludeBuildOutput=false 即可让任意项目引用，无视目标框架
    -->
    <SuppressDependenciesWhenPacking>true</SuppressDependenciesWhenPacking>

    <!-- 不要将输出文件放入到 nuget 的 lib 文件夹下 -->
    <IncludeBuildOutput>false</IncludeBuildOutput>
    <!-- 不要警告 lib 下没内容 -->
    <NoPackageAnalysis>true</NoPackageAnalysis>
```

更多分析器打包相关，请参阅 [dotnet 源代码生成器分析器入门](https://blog.lindexi.com/post/dotnet-%E6%BA%90%E4%BB%A3%E7%A0%81%E7%94%9F%E6%88%90%E5%99%A8%E5%88%86%E6%9E%90%E5%99%A8%E5%85%A5%E9%97%A8.html )
<!-- [dotnet 源代码生成器分析器入门 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18786647 ) -->

### 进行 Publish 发布之后的时机

```xml
<Project>
  <Target Name="Fxxxxx" AfterTargets="Publish">
    <Warning Text="PublishFolder=$([MSBuild]::NormalizePath($(MSBuildProjectDirectory), $(PublishDir)))"/>
  </Target>
</Project>
```

以上的 `[MSBuild]::NormalizePath` 作用和 Path.Combine 或 Path.Join 相同

实测需要使用 `AfterTargets="Publish"` 而不能使用 DependsOnTargets 方式


## 相关文档

[msbuild Roslyn 行为详解](https://blog.lindexi.com/post/msbuild-Roslyn-%E8%A1%8C%E4%B8%BA%E8%AF%A6%E8%A7%A3.html )

[Roslyn 的确定性构建 - walterlv](https://blog.walterlv.com/post/deterministic-builds-in-roslyn.html )

[如何创建一个基于命令行工具的跨平台的 NuGet 工具包 - walterlv](https://blog.walterlv.com/post/create-a-cross-platform-command-based-nuget-tool.html )

[MSBuild properties for Microsoft.NET.Sdk - .NET Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/core/project-sdk/msbuild-props#nuget-metadata-properties )

[NuGet pack and restore as MSBuild targets - Microsoft Learn](https://learn.microsoft.com/en-us/nuget/reference/msbuild-targets )

[让你发布的nuget包支持源代码调试 - czd890 - 博客园](https://www.cnblogs.com/calvinK/p/14982676.html )

更多构建打包相关请看[手把手教你写 Roslyn 修改编译](https://blog.lindexi.com/post/roslyn.html )