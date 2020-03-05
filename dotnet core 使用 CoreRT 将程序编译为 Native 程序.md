# dotnet core 使用 CoreRT 将程序编译为 Native 程序

现在微软有一个开源项目 CoreRT 能通过将托管的 .NET Core 编译为单个无依赖的 Native 程序

这个项目现在还没发布，但是能尝试使用，可以带来很多的性能提升

<!--more-->
<!-- CreateTime:2019/11/29 8:31:17 -->

<!-- 标签：C#,dotnet,dotnetcore -->

使用 CoreRT 发布的优点：

1. 只有一个 exe 文件，是绿色没有依赖

1. 发布的文件的大小很小，对比 dotnet core 的独立发布 50M 的大小会小很多

1. 能在大多数的系统运行

1. 提高很多启动性能

不足是软件不是非常稳定，同时只能支持x64的程序


使用这个项目是比较难的，本文下面提供的包的版本，请大家按照我的安装的版本使用，因为新的版本可能有一些修改，同时没有更新文档，如果是第一次使用，可能会遇到很多坑。

首先打开 VisualStudio 2017 安装 C++ 依赖，虽然现在已经有了 VisualStudio 2019 了，但是 C++ 的依赖是需要和 VS 的版本关系，所以这里暂时不能使用 VisualStudio 2019 安装

点击 Nuget 源设置，在 VisualStudio 的工具->选项->nuget包管理器->nuget包源里面添加 myget 的使用，这个网站是微软的 CI 输出的，里面大量的库都是没有发布的，所以微软也无法保证这些库稳定

添加一个新的源，路径是 [https://dotnet.myget.org/F/dotnet-core/api/v3/index.json](https://dotnet.myget.org/F/dotnet-core/api/v3/index.json) 可以通过 `dotnet new nuget` 创建配置文件，在配置文件添加下面代码，这样就可以不在全局添加这个链接

```csharp
<?xml version="1.0" encoding="utf-8"?>
<configuration>
 <packageSources>
    <add key="dotnet-core" value="https://dotnet.myget.org/F/dotnet-core/api/v3/index.json" />
 </packageSources>
</configuration>
``` 

这个黑科技请看 [VisualStudio 给项目添加特殊的 Nuget 的链接](https://blog.lindexi.com/post/VisualStudio-%E7%BB%99%E9%A1%B9%E7%9B%AE%E6%B7%BB%E5%8A%A0%E7%89%B9%E6%AE%8A%E7%9A%84-Nuget-%E7%9A%84%E9%93%BE%E6%8E%A5.html )

通过 VisualStudio 2017 创建一个新的控制台项目，或者通过命令行使用 dotnet 命令行创建一个新的控制台项目



然后在项目里面添加 Microsoft.DotNet.ILCompiler 的引用，这里添加的版本是 `1.0.0-alpha-*` 版本

```csharp
    <ItemGroup>
        <PackageReference Include="Microsoft.DotNet.ILCompiler" Version="1.0.0-alpha-*" />
    </ItemGroup>
```

也就是在 1.0.0 的任意版本都会被添加，我实际使用的是 1.0.0-alpha-27401-01 版本

现在尝试写一个 Hellow 程序，使用命令行发布，注意创建的项目的 dotnet core sdk 版本暂时需要是 2.0 的版本

```csharp
dotnet publish -r win-x64 -c release
```

注意暂时只能发布 x64 的程序，对 x86 暂时没有支持

现在可以发现发布的文件夹里面有 native 文件夹，里面就只包含一个 exe 程序，同时这个文件也非常小

代码请看 <https://github.com/dotnet/corert/tree/master/samples/HelloWorld>

更详细的博客请看 [使用CoreRT将.NET Core发布为Native应用程序 - KAnts - 博客园](https://www.cnblogs.com/ants/p/8630332.html )

[简析 .NET Core 构成体系 - 帅虫哥 - 博客园](http://www.cnblogs.com/vipyoumay/p/5613373.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
