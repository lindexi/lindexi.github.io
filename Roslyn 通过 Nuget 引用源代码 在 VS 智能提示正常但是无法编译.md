# Roslyn 通过 Nuget 引用源代码 在 VS 智能提示正常但是无法编译

本文告诉大家如果在 Nuget 引用源代码的方式引用源代码，在 VisualStudio 的智能提示和 Resharper 的智能提示都能找到对应的类，但是在 VisualStudio 编译或使用命令行 msbuild 编译时提示找不到类

<!--more-->
<!-- CreateTime:2018/9/29 12:58:16 -->

<!-- csdn -->

<!-- 标签：Roslyn,MSBuild,编译器 -->

这个问题是 msbuild 的一个坑，主要是没有主动引用 `.nuget.g.props` 和 `.nuget.g.targets` 文件，使用 Microsoft.NET.Sdk 作为 Sdk 的项目文件会自动在 obj 文件夹下生成 project.assets.json、$(ProjectName).csproj.nuget.cache、$(ProjectName).csproj.nuget.g.props 和 $(ProjectName).csproj.nuget.g.targets 文件；其中 .nuget.g.props 和 .nuget.g.targets 中生成了 Import 包中编译相关文件的代码。具体请看[MSBuild/Roslyn 和 NuGet 的 100 个坑 - walterlv](https://walterlv.com/post/problems-of-msbuild-and-nuget.html )

但是在使用 Nuget 引用源代码的时候，因为此时源代码还没加入到编译，在编译的时候 msbuild 找不到类，于是就没继续执行，只是就无法编译通过

在我的项目编译出现下面的提示

```csharp
“C:\lindexi\github\SopisatraJowje\SopisatraJowje\SopisatraJowje.csproj”(默认目标) (1) ->
“C:\lindexi\github\SopisatraJowje\SopisatraJowje\SopisatraJowje.csproj”(Build 目标) (1:2) ->
“C:\lindexi\github\SopisatraJowje\SopisatraJowje\SopisatraJowje_rb00pftp_wpftmp.csproj”(_CompileTemporaryAssembly 目标) (
2) ->
(CoreCompile 目标) ->
  MainWindow.xaml.cs(15,12): error CS0234: 命名空间“lindexi”中不存在类型或命名空间名“Doubi”(是否缺少程序集引用?) [C:\lindexi\github\SopisatraJowj
e\SopisatraJowje\SopisatraJowje_rb00pftp_wpftmp.csproj]
```

简单的解决方法是在 csproj 添加引用 `.nuget.g.props` 两个文件，引用的方式是在`<Project Sdk="Microsoft.NET.Sdk" ToolsVersion="15.0">` 的下一句引用 `.nuget.g.props` 例如我创建了项目是 `SopisatraJowje` 我可以使用下面的方式引用

```xml
<Project Sdk="Microsoft.NET.Sdk" ToolsVersion="15.0">
  <Import Project="obj\SopisatraJowje.csproj.nuget.g.props"></Import>

  <!-- 下面是原来的内容 -->
  <PropertyGroup>
    <LanguageTargets>$(MSBuildToolsPath)\Microsoft.CSharp.targets</LanguageTargets>
    <TargetFrameworks>net45;</TargetFrameworks>
    <OutputType>WinExe</OutputType>
  </PropertyGroup>
```

在文件的最后再添加引用 nuget.g.targets 文件的最后需要在 `</Project>` 上一个

```xml
  <Import Project="obj\SopisatraJowje.csproj.nuget.g.targets"></Import>
</Project>
```

也就是文件看起来是这样

```
<Project Sdk="Microsoft.NET.Sdk" ToolsVersion="15.0">
  <Import Project="obj\SopisatraJowje.csproj.nuget.g.props"></Import>
  
  <PropertyGroup>
    <LanguageTargets>$(MSBuildToolsPath)\Microsoft.CSharp.targets</LanguageTargets>
    <TargetFrameworks>net45;</TargetFrameworks>
    <OutputType>WinExe</OutputType>
  </PropertyGroup>
 

  <Import Project="obj\SopisatraJowje.csproj.nuget.g.targets"></Import>
</Project>
```

添加了这两个引用就可以解决源代码引用的时候出现了在 VisualStudio 可以跳转找到类，但是在编译的时候找不到类的问题

[MSBuild/Roslyn 和 NuGet 的 100 个坑 - walterlv](https://walterlv.com/post/problems-of-msbuild-and-nuget.html )

![](http://image.acmx.xyz/lindexi%2F2018927201059809)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
