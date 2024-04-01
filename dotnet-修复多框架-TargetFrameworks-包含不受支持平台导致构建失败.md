
# dotnet 修复多框架 TargetFrameworks 包含不受支持平台导致构建失败

本文将告诉大家如何修复 dotnet 项目里的多框架 TargetFrameworks 如果包含了当前系统无法支持的平台时，如何进行跳过。解决在 Linux 平台构建时提示 Mac Catalyst 不受支持而构建失败

<!--more-->


<!-- CreateTime:2024/03/31 07:13:30 -->

<!-- 发布 -->
<!-- 博客 -->

故事的背景是我期望在 GitHub 的 Action 里面构建一个项目，我期望能够在 Windows 和 Linux 和 Mac 平台上进行构建，一开始 Windows 和 Mac 平台都十分顺利，只是到 Linux 平台时就不断构建失败了

核心构建失败的原因是在 GitHub 的 Action 里面的 Linux 不直接支持 Mac Catalyst 平台，而我在 GitHub 的 Action 里面也不想也不用构建 Mac Catalyst 平台。但可惜的是我的 csproj 里面的 TargetFrameworks 是这样写的

```xml
 <TargetFrameworks>net6.0;net6.0-windows;net6.0-maccatalyst</TargetFrameworks>
```

构建时的核心报错信息如下

```
MSBuild version 17.9.6+a4ecab324 for .NET
  Determining projects to restore...
/usr/share/dotnet/sdk/8.0.202/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.Sdk.ImportWorkloads.targets(38,5): error NETSDK1178: The project depends on the following workload packs that do not exist in any of the workloads available in this installation: Microsoft.maccatalyst.Sdk.net8
```

核心错误就是 error NETSDK1178: The project depends on the following workload packs that do not exist in any of the workloads available in this installation: Microsoft.maccatalyst.Sdk.net8

但是我在 Linux 平台只想构建 net6.0 的版本，即使我使用了如下命令行，也依然构建失败

```
dotnet build -c release -r linux-x64 -p:TargetFramework=net6.0
```

或者是如下命令也是无效的

```
dotnet build -c release -r linux-x64 -f net6.0
```

这可能是因为本质上挂的步骤是在 dotnet restore 还原的步骤里面，在还原步骤时做的是完全的还原，没有受到 dotnet build 的影响。这也就是为什么从 dotnet 的设计上，也提供了 dotnet restore 命令的原因

解决此问题可以使用 dotnet restore 命令，通过 dotnet restore 命令可以做到更细节的控制还原逻辑，避免在还原时读取 TargetFramework 找到不受支持的平台而失败

将以上的构建命令换成以下两句命令即可修复问题

```
dotnet restore -p:TargetFramework=net6.0
dotnet build -c release -p:TargetFramework=net6.0 --no-restore
```

以上的 dotnet build 命令里面需要带上 `--no-restore` 参数，用来表示只构建不还原，这是因为咱在上一句命令里面就自己还原了

通过以上的命令手动设置 TargetFramework 可以避免 dotnet restore 时对整体的框架进行还原，导致遇到不受支持的框架平台返回失败

更进一步可以添加上具体的运行时版本，如在 linux 下只构建 linux 的版本，方法是加上 `-r` 参数，修改之后的命令行如下

```
dotnet restore -p:TargetFramework=net6.0 -r linux-x64
dotnet build -c release -r linux-x64 -p:TargetFramework=net6.0 --no-restore
```

通过以上的构建命令可以更快的完成指定平台构建，且解决包含不受支持的平台构建失败




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。