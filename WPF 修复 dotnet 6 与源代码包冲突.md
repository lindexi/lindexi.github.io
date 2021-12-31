# WPF 修复 dotnet 6 与源代码包冲突

在 dotnet 6 时，官方为了适配好 Source Generators 功能，于是默认就将 WPF 的 XAML 构建过程中，引入第三方库的 cs 文件，这个功能默认设置为开启。刚好源代码包为了修复在使用 dotnet 6 SDK 之前，在 WPF 的构建 XAML 过程中，不包含第三方库的代码文件，从而使用黑科技将源代码包加入到 WPF 构建 XAML 中。在 VisualStudio 升级到 2022 版本，或者是升级 dotnet sdk 到 dotnet 6 版本，将会更新构建调度，让源代码包里的代码文件被加入两次，从而构建失败

<!--more-->
<!-- CreateTime:2021/12/29 15:23:10 -->

<!-- 发布 -->

修复方法很简单，在不更改源代码包的前提下，可以在 csproj 项目文件里加入以下代码

```xml
    <IncludePackageReferencesDuringMarkupCompilation>False</IncludePackageReferencesDuringMarkupCompilation>
```

更改之后的 csproj 代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net6.0-windows</TargetFramework>
    <UseWPF>true</UseWPF>
    <IncludePackageReferencesDuringMarkupCompilation>False</IncludePackageReferencesDuringMarkupCompilation>
  </PropertyGroup>

</Project>
```

此影响不仅包含 TargetFramework 为 net6.0-windows 的 WPF 应用，而是任何使用 SDK 风格的 WPF 项目。其原因是 dotnet 6 此更改是在 dotnet sdk 更改构建调度过程，和具体应用的框架无关，只影响构建本身

<!-- ![](image/WPF 修复 dotnet 6 与源代码包冲突/WPF 修复 dotnet 6 与源代码包冲突0.png) -->

![](http://image.acmx.xyz/lindexi%2F202112291523193526.jpg)

更多关于 WPF 构建过程，请看 [WPF 程序的编译过程 - walterlv](https://blog.walterlv.com/post/how-wpf-assemblies-are-compiled.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
