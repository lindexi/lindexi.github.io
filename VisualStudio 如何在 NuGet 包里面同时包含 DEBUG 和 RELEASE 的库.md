# VisualStudio 如何在 NuGet 包里面同时包含 DEBUG 和 RELEASE 的库

我在开发的时候需要使用到一些 DEBUG 库进行调试，但是我的库是通过 NuGet 给用户的，如果在 NuGet 里面使用到了 DEBUG 的库那么会让代码的运行效率降低。于是我就找到一个方法，可以在 NuGet 同时打包调试和发布的包，这样在用户调试的时候就可以使用调试的代码

<!--more-->
<!-- CreateTime:2019/4/15 16:13:42 -->

<!-- csdn -->

<!-- 标签：VisualStudio，调试,nuget -->

我在一个库写代码，我需要做一点黑科技，让[吕毅](https://blog.walterlv.com/) 在调试的时候输出的是 林德熙是逗比，但是在他发布的时候却输出吕毅是逗比那么我需要如何做？

打开 [VisualStudio 2019](https://blog.lindexi.com/post/VisualStudio-2019-%E6%96%B0%E7%89%B9%E6%80%A7.html) 创建一个项目，然后添加下面一点代码

```csharp
using System;

namespace LerewararraNurfabeyo
{
    public class WijonakabaiBohallcallcem
    {
        public void Foo()
        {
#if DEBUG
            Console.WriteLine("林德熙是逗比");
#else
            Console.WriteLine("吕毅是逗比");
#endif

        }
    }
}

```

使用右击一键打包 NuGet 的方法创建一个 NuGet 包，这是 VisualStudio 2017 的 [VisualStudio 使用新项目格式快速打出 Nuget 包](https://blog.lindexi.com/post/VisualStudio-%E4%BD%BF%E7%94%A8%E6%96%B0%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F%E5%BF%AB%E9%80%9F%E6%89%93%E5%87%BA-Nuget-%E5%8C%85.html) 功能很好用

![](http://image.acmx.xyz/lindexi%2F2019415144444107)

再创建一个项目，为了直接引用上面测试项目的 NuGet 需要做一点黑科技，第一步是让测试项目的 NuGet 输出到一个文件夹

在 sln 所在的文件夹添加 [Directory.Build.props](https://blog.lindexi.com/post/Roslyn-%E4%BD%BF%E7%94%A8-Directory.Build.props-%E6%96%87%E4%BB%B6%E5%AE%9A%E4%B9%89%E7%BC%96%E8%AF%91.html) 文件，在这个文件可以设置全局的项目输出的文件夹

```csharp
<Project>
  <PropertyGroup>
    <PackageOutputPath>$(MSBuildThisFileDirectory)package</PackageOutputPath>
  </PropertyGroup>
</Project>
```

于是测试项目就会输出到这个文件夹，下面再创建一个控制台项目，在这个项目里面引用测试项目的代码

我是在相同的 sln 创建项目，所以写的文件夹都是相对的，文件夹请看 [代码](https://github.com/lindexi/lindexi_gd/tree/09626aa29c65d8efb00e71797e3b4e6b88c19eff/LerewararraNurfabeyo )

为了引用到测试项目的输出需要添加 nuget.config 文件，通过在命令行输入 `dotnet new nuget` 就可以成功创建，创建完成需要修改相对文件夹，这个科技请看[VisualStudio 给项目添加特殊的 Nuget 的链接](https://blog.lindexi.com/post/VisualStudio-%E7%BB%99%E9%A1%B9%E7%9B%AE%E6%B7%BB%E5%8A%A0%E7%89%B9%E6%AE%8A%E7%9A%84-Nuget-%E7%9A%84%E9%93%BE%E6%8E%A5.html) 通过这个文件可以修改本地的输出

```csharp
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <packageSources>
        <!--To inherit the global NuGet package sources remove the <clear/> line below -->
        <clear/>
        <add key="nuget" value="..\package" />
    </packageSources>
</configuration>

```

具体文件请看： [nuget.config](https://github.com/lindexi/lindexi_gd/blob/09626aa29c65d8efb00e71797e3b4e6b88c19eff/LerewararraNurfabeyo/WhawlalljeaceahearlurKudaiheko/nuget.config )

通过 dotnet add package 测试项目名 可以安装了测试项目的输出，具体请看 [代码](https://github.com/lindexi/lindexi_gd/blob/09626aa29c65d8efb00e71797e3b4e6b88c19eff/LerewararraNurfabeyo/WhawlalljeaceahearlurKudaiheko/WhawlalljeaceahearlurKudaiheko.csproj )

现在开始尝试编译，会发现无论是 DEBUG 下还是发布版本都是输出 林德熙是逗比，这样显然不是我需要的

现在简单的方法已经解决不了了，可以通过复杂的 NuGet 命令做到，先添加 NuGet 到环境变量，这样可以在任意的地方使用到命令

在 [https://www.nuget.org/downloads](https://www.nuget.org/downloads) 下载最新的 Nuget 软件

通过输入 Nuget 命令创建 [LerewararraNurfabeyo.nuspec](https://github.com/lindexi/lindexi_gd/blob/09626aa29c65d8efb00e71797e3b4e6b88c19eff/LerewararraNurfabeyo/LerewararraNurfabeyo/LerewararraNurfabeyo.nuspec) 文件，这个命令请看[NuGet CLI spec command](https://docs.microsoft.com/en-us/nuget/tools/cli-ref-spec?wt.mc_id=MVP )

```csharp
nuget spec LerewararraNurfabeyo.csproj
```

现在就创建了 LerewararraNurfabeyo.nuspec 文件，需要在这个文件里面替换很多代码

```csharp
<?xml version="1.0"?>
<package >
  <metadata>
    <id>LerewararraNurfabeyo</id>
    <version>1.0.2</version>
    <title>LerewararraNurfabeyo</title>
    <authors>lindexi</authors>
    <owners>lindexi</owners>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <description>测试代码</description>
    <copyright>Copyright 2019</copyright>
  </metadata>
    <files>
        
    </files>
</package>
```

我需要在 files 里面添加调试和发布的代码，也就是需要先编译了调试代码和发布的代码才可以打包。可以通过一个命令行执行编译，创建一个 cmd 文件，在这个文件里面添加编译代码，这个文件的代码放在 [github ](https://github.com/lindexi/lindexi_gd/blob/09626aa29c65d8efb00e71797e3b4e6b88c19eff/LerewararraNurfabeyo/LerewararraNurfabeyo/package.cmd) 这是里面的代码 

```csharp
@echo

dotnet build --configuration Release
dotnet build --configuration Debug
```

可以知道编译的文件输出在 `bin\Debug\netcoreapp3.0` 和 `bin\Release\netcoreapp3.0` 需要将文件放在 nuget 文件夹

```csharp
        <file src="bin\Debug\netcoreapp3.0\LerewararraNurfabeyo.dll" target="lib\debug\LerewararraNurfabeyo.dll" />
        <file src="bin\Debug\netcoreapp3.0\LerewararraNurfabeyo.pdb" target="lib\debug\LerewararraNurfabeyo.pdb" />
        <file src="bin\Release\netcoreapp3.0\LerewararraNurfabeyo.dll" target="lib\release\LerewararraNurfabeyo.dll" />
        <file src="bin\Release\netcoreapp3.0\LerewararraNurfabeyo.pdb" target="lib\release\LerewararraNurfabeyo.pdb" />
```

虽然添加了文件但是还不会自动选择在调试下使用调试的代码，在发布下使用发布的代码，需要添加一个 `LerewararraNurfabeyo.targets` 文件在调试的时候引用调试的代码

这里的 targets 文件的命名要求是 nuget 包对应的 id 同时放在 build 文件夹里面，也就是打开 nuget包可以看到 `build\id.targets` 里面的代码才可以运行，关于这个文件请看[代码](https://github.com/lindexi/lindexi_gd/blob/09626aa29c65d8efb00e71797e3b4e6b88c19eff/LerewararraNurfabeyo/LerewararraNurfabeyo/package/LerewararraNurfabeyo.targets)

```csharp
<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
    <ItemGroup Condition="'$(Configuration)' == 'DEBUG'">
        <Reference Include="LerewararraNurfabeyo">
            <HintPath>$(MSBuildThisFileDirectory)..\lib\debug\LerewararraNurfabeyo.dll</HintPath>
        </Reference>
    </ItemGroup>

    <ItemGroup Condition="'$(Configuration)' == 'RELEASE'">
        <Reference Include="LerewararraNurfabeyo">
            <HintPath>$(MSBuildThisFileDirectory)..\lib\release\LerewararraNurfabeyo.dll</HintPath>
        </Reference>
    </ItemGroup>
</Project>
```

通过 `$(Configuration)` 判断当前是调试还是发布，从而引用不同的代码，在 `$(MSBuildThisFileDirectory)` 将会拿到当前这个文件的路径，通过相对的文件引用就可以找到

这时需要将这个文件打包

```csharp
        <file src="package\LerewararraNurfabeyo.targets" target="build\LerewararraNurfabeyo.targets" />

```

此时打包文件就写好了，请看[代码](https://github.com/lindexi/lindexi_gd/blob/09626aa29c65d8efb00e71797e3b4e6b88c19eff/LerewararraNurfabeyo/LerewararraNurfabeyo/LerewararraNurfabeyo.nuspec)

通过 Nuget 命令打包

```csharp
nuget pack LerewararraNurfabeyo.nuspec
```

将这个命令放在 package 命令文件，请看[代码](https://github.com/lindexi/lindexi_gd/blob/09626aa29c65d8efb00e71797e3b4e6b88c19eff/LerewararraNurfabeyo/LerewararraNurfabeyo/package.cmd)

这时打包完成的包是放在相同的文件夹，因为刚才已经用到在上一层的文件夹，所以需要修改代码，请看[github](https://github.com/lindexi/lindexi_gd/commit/b82cc79d39c1238c68b2426e41b790b61344e7f7) 的修改，通过 `-OutputDirectory` 修改输出文件夹

现在尝试测试一下，更新一下测试项目的库然后在调试和发布下运行看输出

```csharp
// 在调试下运行
dotnet run 
// 输出林德熙是逗比

// 在发布运行
dotnet run --configuration release
// 输出吕毅是逗比
```

通过这个方法就可以在库同时包含调试的代码和发布的代码，因为在调试的代码可以添加很多影响性能的代码，所以通过这个方法可以方便调试也提高发布代码的效率，但是需要自己写一个[nuspec](https://github.com/lindexi/lindexi_gd/blob/b82cc79d39c1238c68b2426e41b790b61344e7f7/LerewararraNurfabeyo/LerewararraNurfabeyo/LerewararraNurfabeyo.nuspec ) 文件用来打包，因为需要连续编译两次。需要在 [targets](https://github.com/lindexi/lindexi_gd/blob/b82cc79d39c1238c68b2426e41b790b61344e7f7/LerewararraNurfabeyo/LerewararraNurfabeyo/package/LerewararraNurfabeyo.targets ) 文件在不同的版本使用不同的库

所有代码请看 [github](https://github.com/lindexi/lindexi_gd/tree/09626aa29c65d8efb00e71797e3b4e6b88c19eff/LerewararraNurfabeyo )

[Roslyn 使用 Directory.Build.props 文件定义编译](https://blog.lindexi.com/post/Roslyn-%E4%BD%BF%E7%94%A8-Directory.Build.props-%E6%96%87%E4%BB%B6%E5%AE%9A%E4%B9%89%E7%BC%96%E8%AF%91.html)

[Roslyn 使用 Directory.Build.props 管理多个项目配置](https://blog.lindexi.com/post/Roslyn-%E4%BD%BF%E7%94%A8-Directory.Build.props-%E7%AE%A1%E7%90%86%E5%A4%9A%E4%B8%AA%E9%A1%B9%E7%9B%AE%E9%85%8D%E7%BD%AE.html)

[VisualStudio 2019 新特性](https://blog.lindexi.com/post/VisualStudio-2019-%E6%96%B0%E7%89%B9%E6%80%A7.html)

[VisualStudio 给项目添加特殊的 Nuget 的链接](https://blog.lindexi.com/post/VisualStudio-%E7%BB%99%E9%A1%B9%E7%9B%AE%E6%B7%BB%E5%8A%A0%E7%89%B9%E6%AE%8A%E7%9A%84-Nuget-%E7%9A%84%E9%93%BE%E6%8E%A5.html)

VisualStudio 使用新项目格式快速打出 Nuget 包](https://blog.lindexi.com/post/VisualStudio-%E4%BD%BF%E7%94%A8%E6%96%B0%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F%E5%BF%AB%E9%80%9F%E6%89%93%E5%87%BA-Nuget-%E5%8C%85.html)

[How to create a nuget package with both release and debug dll's using nuget package explorer](https://stackoverflow.com/a/37676225/6116637)

[How to create a NuGet package](https://docs.microsoft.com/en-us/nuget/create-packages/creating-a-package?wt.mc_id=MVP )

[NuGet CLI spec command](https://docs.microsoft.com/en-us/nuget/tools/cli-ref-spec?wt.mc_id=MVP )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
