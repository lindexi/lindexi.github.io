
# dotnet C# 使用 System.IO.Hashing 库进行 Crc32 计算

本文和大家介绍 dotnet 官方提供的 System.IO.Hashing 库进行 Crc32 计算

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

官方文档： <https://learn.microsoft.com/en-us/dotnet/api/system.io.hashing.crc32?view=net-9.0-pp>

在没有安装任何库的情况下，大家也能点出来一个 Crc32 类型，这个类型是放在 `System.Runtime.Intrinsics.Arm` 命名空间下的，这就意味着大部分情况下在 x86 或 x64 设备上，获取是否支持的 `System.Runtime.Intrinsics.Arm.Crc32.IsSupported` 属性都会是 false 值

而且 `System.Runtime.Intrinsics.Arm.Crc32` 也不能提供很好的 API 方法让咱进行文件的 Crc32 计算

根据微软官方文档可以了解到，这部分能力是额外通过 NuGet 包提供的，没有集成到默认的框架里面。但源代码都在 [dotnet 官方仓库](https://github.com/dotnet/runtime/blob/c81f403737c412942222d13b2753881e62d1e6f7/src/libraries/System.IO.Hashing/src/System/IO/Hashing/Crc32.cs) 里面，可以放心使用

按照 dotnet 的惯例，先通过 NuGet 管理器安装 `System.IO.Hashing` 库。当然，也可以修改 csproj 项目文件进行快速安装，修改之后的 csproj 项目文件代码大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="System.IO.Hashing" Version="9.0.4" />
  </ItemGroup>

</Project>
```

根据 System.IO.Hashing 库的说明，在当前 9.0.4 版本里面，最低支持为 .NET Standard 2.0 版本

以下是演示如何计算一个文件的 Crc32 值

```csharp
using System.IO.Hashing;

var file = @"C:\lindexi\Work\file";
var crc32 = new Crc32();
await using var fileStream = File.OpenRead(file);
await crc32.AppendAsync(fileStream);
var crcText = crc32.GetCurrentHashAsUInt32().ToString("X");
```

可以看到其方法非常简单

在 System.IO.Hashing 库里面，还提供了 CRC-64、xxHash3、 xxHash32、 xxHash64 和 xxHash128 的支持，其演示代码如下

```csharp
using System;
using System.IO.Hashing;

byte[] data = new byte[] { 1, 2, 3, 4 };

byte[] crc32Value = Crc32.Hash(data);
Console.WriteLine($"CRC-32 Hash: {BitConverter.ToString(crc32Value)}");
// CRC-32 Hash: CD-FB-3C-B6

byte[] crc64Value = Crc64.Hash(data);
Console.WriteLine($"CRC-64 Hash: {BitConverter.ToString(crc64Value)}");
// CRC-64 Hash: 58-8D-5A-D4-2A-70-1D-B2

byte[] xxHash3Value = XxHash3.Hash(data);
Console.WriteLine($"XxHash3 Hash: {BitConverter.ToString(xxHash3Value)}");
// XxHash3 Hash: 98-8B-7B-90-33-AC-46-22

byte[] xxHash32Value = XxHash32.Hash(data);
Console.WriteLine($"XxHash32 Hash: {BitConverter.ToString(xxHash32Value)}");
// XxHash32 Hash: FE-96-D1-9C

byte[] xxHash64Value = XxHash64.Hash(data);
Console.WriteLine($"XxHash64 Hash: {BitConverter.ToString(xxHash64Value)}");
// XxHash64 Hash: 54-26-20-E3-A2-A9-2E-D1

byte[] xxHash128Value = XxHash128.Hash(data);
Console.WriteLine($"XxHash128 Hash: {BitConverter.ToString(xxHash128Value)}");
// XxHash128 Hash: 49-A0-48-99-59-7A-35-67-53-76-53-A0-D9-95-5B-86
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/32dd2a2972a78abf4145a8a332ef10b57595e752/Workbench/WalwofurkaheyaceKenikila) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/32dd2a2972a78abf4145a8a332ef10b57595e752/Workbench/WalwofurkaheyaceKenikila) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 32dd2a2972a78abf4145a8a332ef10b57595e752
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 32dd2a2972a78abf4145a8a332ef10b57595e752
```

获取代码之后，进入 Workbench/WalwofurkaheyaceKenikila 文件夹，即可获取到源代码




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。