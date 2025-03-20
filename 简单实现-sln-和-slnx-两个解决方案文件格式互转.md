
# 简单实现 sln 和 slnx 两个解决方案文件格式互转

本文将告诉大家如何通过 Microsoft.VisualStudio.SolutionPersistence 实现 sln 和 slnx 格式文件的相互转换

<!--more-->


<!-- CreateTime:2025/03/20 07:25:55 -->

<!-- 发布 -->
<!-- 博客 -->

按照 dotnet 的惯例，使用 NuGet 安装 Microsoft.VisualStudio.SolutionPersistence 库，安装之后的 csproj 项目文件内容大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.VisualStudio.SolutionPersistence" Version="1.0.52" />
  </ItemGroup>

</Project>
```

互转的核心方法就是通过 `SolutionSerializers.SlnXml` 或 `SolutionSerializers.SlnFileV12` 的 OpenAsync 方法去读取文件，使用 SaveAsync 将其保存为另一个格式

如以下代码所示，从 slnx 转到 sln 格式

```csharp
using Microsoft.VisualStudio.SolutionPersistence.Model;
using Microsoft.VisualStudio.SolutionPersistence.Serializer;

var slnxFile = @"C:\lindexi\Work\Foo.slnx";
var slnFile = @"C:\lindexi\Work\Foo.sln";

var solutionModel = await SolutionSerializers.SlnXml.OpenAsync(slnxFile, CancellationToken.None);

await SolutionSerializers.SlnFileV12.SaveAsync(slnFile, solutionModel, CancellationToken.None);
```

如以上代码所示，调用 `SolutionSerializers.SlnXml.OpenAsync` 读取 slnx 文件，获取 SolutionModel 内存模型。调用 `SolutionSerializers.SlnFileV12.SaveAsync` 保存为 sln 格式

反过来从 sln 转换为 slnx 代码如下，只是将 SlnXml 和 SlnFileV12 进行调换而已

```csharp
var slnxFile = @"C:\lindexi\Work\Foo.slnx";
var slnFile = @"C:\lindexi\Work\Foo.sln";

SolutionModel solutionModel = await SolutionSerializers.SlnFileV12.OpenAsync(slnFile, CancellationToken.None);

await SolutionSerializers.SlnXml.SaveAsync(slnxFile, solutionModel, CancellationToken.None);
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/cf7cbded41b9a88fed7da3343664fc5a9b3396eb/Workbench/WhanenoheRarigagarnere) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/cf7cbded41b9a88fed7da3343664fc5a9b3396eb/Workbench/WhanenoheRarigagarnere) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin cf7cbded41b9a88fed7da3343664fc5a9b3396eb
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin cf7cbded41b9a88fed7da3343664fc5a9b3396eb
```

获取代码之后，进入 Workbench/WhanenoheRarigagarnere 文件夹，即可获取到源代码




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。