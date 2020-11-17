
# 刚刚我从服务器回滚了 dotnet 5 的环境

今天是 2020.11.13 我在 CI 服务器上更新 dotnet 到 dotnet 5 以及 VS 到 16.8.1 最新版本，但是我在刚刚不得不回滚了环境…… 因为构建不通过

<!--more-->


<!-- CreateTime:2020/11/14 10:01:27 -->

<!-- 发布 -->

## 现象

使用经典的 NuGet 还原加上 msbuild 的构建，如以下代码将会构建失败

```
nuget restore
msbuild /p:Configuration=Release
```

构建失败核心提示如下

```
C:\Program Files\dotnet\sdk\5.0.100\Sdks\Microsoft.NET.Sdk\targets\Microsoft.PackageDependencyResolution.targets(241,5): error NETSDK1005: 资产文件“C:\gitlab\builds\SKH4KvNc\0\Lindexi\Lindexi-Doubi\demo\Lindexi.Doubi.DesktopModeDemo\obj\project.assets.json”没有“net45”的目标。确保已运行还原，且“net45”已包含在项目的 TargetFrameworks 中。 [C:\gitlab\builds\SKH4KvNc\0\Lindexi\Lindexi-Doubi\demo\Lindexi.Doubi.DesktopModeDemo\Lindexi.Doubi.DesktopModeDemo.csproj]
```

## 原因

本质原因是 NuGet 没有跟上步伐，因为通过 dotnet restore 是可以的

## 不完美解决方法

如果项目里面不存在旧项目格式，以及 VS 插件项目等等项目，试试用 dotnet 命令构建

```
dotnet build -c release
```

或者使用 dotnet 的还原配合 msbuild 的构建

```
dotnet restore
msbuild /p:Configuration=Release
```

但是以上方法都有缺点，在一些使用旧版本格式的 csproj 项目文件里面，以及一些 VSTO 项目，或者引用了 COM 的项目和 VS 插件项目等类型里面，将因为 dotnet restore 的不支持导致后续构建不通过。或者在 dotnet build 里面使用了 dotnet core 版本的 msbuild 丢失了旧版本 dotnet framework 版本的 msbuild 支持的旧版本功能，让一些旧版本项目构建不通过

我当前的 CI 服务器上依然承担了一定数量的旧版本项目的任务，因此 dotnet 5 环境暂时还不能支持，只能回滚了

## 彻底的解决方法

~~彻底的解决方法是： 再等几天~~

更新到 NuGet 5.8 就能解决

这是全网首个能支持 .NET 5 的 NuGet 版本，在 [https://dist.nuget.org/win-x86-commandline/v5.8.0/nuget.exe](https://dist.nuget.org/win-x86-commandline/v5.8.0/nuget.exe) 可以下载





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。