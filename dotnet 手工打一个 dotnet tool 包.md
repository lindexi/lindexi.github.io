# dotnet 手工打一个 dotnet tool 包

现在依靠 dotnet 平台，可以方便分发工具，利用 NuGet 服务进行分发和使用工具。打一个 dotnet tool 包，可以将这个包上传到 NuGet 上，小伙伴通过和安装 NuGet 相同方式就可以将工具安装在本机上。本文将告诉大家如何手工打一个 dotnet tool 包，方便小伙伴自己写工具用来创建代码

<!--more-->
<!-- CreateTime:2020/2/18 17:56:53 -->

<!-- 发布 -->
<!-- 标签: NuGet,dotnet,dotnettool -->

所有可执行项目可以打包为 dotnet tool 包，通过 `dotnet xx` 的命令就可以执行对应的软件。而 dotnet tool 包本身就是 NuGet 包，如果是在 dotnet 生成或 VisualStudio 中，只需要在 csporj 文件添加下面代码

```
        <PackAsTool>true</PackAsTool>
        <ToolCommandName>nugetfix</ToolCommandName>
```

如下面代码

```
    <PropertyGroup>
        <OutputType>WinExe</OutputType>
        <TargetFramework>netcoreapp3.1</TargetFramework>
        <UseWPF>true</UseWPF>
        <ApplicationIcon>Icon.ico</ApplicationIcon>
        <AssemblyName>NugetMergeFixTool</AssemblyName>
        <RootNamespace>dotnetCampus.NugetMergeFixTool</RootNamespace>
        <GeneratePackageOnBuild>true</GeneratePackageOnBuild>

        <PackAsTool>true</PackAsTool>
        <ToolCommandName>nugetfix</ToolCommandName>
    </PropertyGroup>
```

然后打包就可以了

代码请看[github](https://github.com/dotnet-campus/dotnetCampus.NugetMergeFixTool/blob/59916d4985a7ccb89bde81c3e4e8ff9962642cc8/dotnetCampus.NugetMergeFixTool/dotnetCampus.NugetMergeFixTool.csproj) 欢迎小伙伴访问

打包的 NuGet 包，可以通过下面命令安装

```
dotnet tool install --global --add-source .\bin\debug NugetMergeFixTool
```

接下来可以使用 `nugetfix` 启动这个应用，传入的命令行也可以传入应用

那么 `nugetfix` 这个参数是从哪里获取的？实际上在 `<ToolCommandName>nugetfix</ToolCommandName>` 设置的

如果我想要用 `dotnet nugetfix` 启动命令，那么请将 `nugetfix` 修改为 `dotnet-nugetfix` 就可以

这个 NuGet 包和其他的 NuGet 有什么不同

如果我需要手动打包，我先需要可执行文件，例如 Windows 下的 exe 文件，注意没有限制平台，也就是 Linux 也可以。这里说的可执行文件在 Windows 下可能是 dll 哦，只要通过 `dotnet` 命令可以启动这个 dll 就可以

我假设拿到可执行文件和他的所有依赖文件，放在 lindexi 文件夹里面

接下来就是手工打包了

创建准备打包文件夹，如 packing 文件夹，在 packing 文件夹里面创建 tools 文件夹，在 tools 文件夹创建对应框架文件夹，如 `netcoreapp3.1` 再创建 any 文件夹（AnyCPU) 请看下面路径

```
tools\netcoreapp3.1\any\
```

请将 lindexi 文件夹里面的所有文件放在 any 文件夹里面，需要确定 any 文件夹里面存在可执行文件，如 NugetMergeFixTool.dll 文件

接着在 any 文件夹里面创建 DotnetToolSettings.xml 文件，内容请看代码

```xml
<?xml version="1.0" encoding="utf-8"?>
<DotNetCliTool Version="1">
  <Commands>
    <Command Name="nugetfix" EntryPoint="NugetMergeFixTool.dll" Runner="dotnet" />
  </Commands>
</DotNetCliTool>
```

这里 Command 的 Name 就是 `nugetfix` 中的命令，而 EntryPoint 就是入口文件，请将代码修改为你需要的代码

返回 packing 文件夹，创建 nuspec 文件，我期望阅读本文的小伙伴都知道 nuspec 文件应该如何写，我不会告诉大家细节

```xml
<?xml version="1.0" encoding="utf-8"?>
<package xmlns="http://schemas.microsoft.com/packaging/2012/06/nuspec.xsd">
  <metadata>
    <id>NugetMergeFixTool</id>
    <version>0.1.19026-alpha</version>
    <authors>dotnet-campus</authors>
    <owners>dotnet-campus</owners>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <license type="expression">MIT</license>
    <licenseUrl>https://licenses.nuget.org/MIT</licenseUrl>
    <projectUrl>https://github.com/dotnet-campus/dotnetCampus.NugetMergeFixTool</projectUrl>
    <description>读写文件升级NuGet库，修复 NuGet 库引用</description>
    <copyright>Copyright (c) 2020 dotnet-campus</copyright>
    <tags>dotnet nuget msbuild</tags>
    <packageTypes>
      <packageType name="DotnetTool" />
    </packageTypes>
    <repository type="git" url="https://github.com/dotnet-campus/dotnetCampus.NugetMergeFixTool.git" />
    <frameworkReferences>
      <group targetFramework=".NETCoreApp3.1">
        <frameworkReference name="Microsoft.WindowsDesktop.App.WPF" />
      </group>
    </frameworkReferences>
  </metadata>
</package>
```

核心是 packageTypes 代码

```xml
    <packageTypes>
      <packageType name="DotnetTool" />
    </packageTypes>
```

然后将 packing 文件夹作为压缩包，注意修改压缩包名为 id.版本.nupkg 文件

这样就完成手工打包

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
