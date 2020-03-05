# NuGet 符号服务器

在新的 VisualStudio 支持使用 NuGet 符号服务器，可以支持新的 Portable PDB 调试符号的库，本文告诉大家如何打包上传带符号的库和使用符号服务器

<!--more-->
<!-- CreateTime:2019/5/21 11:34:40 -->

<!-- 标签：nuget -->

在 2018 的 11 月微软支持上传带符号的包到 NuGet.org 符号服务器，在使用 Visual Studio 2017 15.9 和以上版本可以支持打包 .NET Core 的 [Portable PDB](https://www.infoq.cn/article/2017/02/Portable-PDB ) 符号的库，或者使用 [nuget.exe 4.9.0](https://www.nuget.org/downloads) 或 dotnet 命令行打包

创建的带符号的库的后缀是 `.snupkg` 下面是一些打包方法

使用 dotnet 命令行方法，以打包 Foo 项目为例

```csharp
dotnet pack Foo.csproj --include-symbols -p:SymbolPackageFormat=snupkg
```

使用 nuget 命令行的方法，请看下面代码，请将 Foo.nuspec 替换为你自己需要的打包文件

```csharp
 nuget pack Foo.nuspec -Symbols -SymbolPackageFormat snupkg
```

使用 msbuild 命令行的方法，请替换为自己项目

```csharp
msbuild /t:pack Foo.csproj /p:IncludeSymbols=true /p:SymbolPackageFormat=snupkg
```

此时打包出来的是 `.snupkg` 文件，可以用来上传到符号服务器

上传的方法是打开[nuget.org](https://www.nuget.org/packages/manage/upload)点击上传，传入文件

![](http://image.acmx.xyz/lindexi%2F201958213818688)

在 VisualStudio 使用 NuGet 符号服务器的方法是添加符号服务器 `https://symbols.nuget.org/download/symbols` 到工具-选项-调试-符号

![](http://image.acmx.xyz/lindexi%2F201958214432905)

注意，当前的符号服务器支持的 `.snupkg` 文件需要使用最新的[portable pdb](https://github.com/dotnet/core/blob/master/Documentation/diagnostics/portable_pdb.md ) 格式，关于 [portable pdb](https://github.com/dotnet/core/blob/master/Documentation/diagnostics/portable_pdb.md ) 请看[介绍Portable PDB](https://www.infoq.cn/article/2017/02/Portable-PDB )

在 VisualStudio 2019 16.1 Preview 2 默认添加了符号服务器，可以点击勾选就可以使用

[介绍Portable PDB](https://www.infoq.cn/article/2017/02/Portable-PDB )

[portable pdb](https://github.com/dotnet/core/blob/master/Documentation/diagnostics/portable_pdb.md )

[Improved package debugging experience with the NuGet.org symbol server](https://blog.nuget.org/20181116/Improved-debugging-experience-with-the-NuGet-org-symbol-server-and-snupkg.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
