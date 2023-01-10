# WPF 修复引用库报错 所使用的 PresentationFramework 6.0.2 高于所引用的标识为 6.0.0 程序集

本文记录一个 WPF 构建的坑，这是 WPF 仓库的运维管理大意挖的坑。将会在大家使用低版本的 dotnet 6 如 6.0.1 版本时，引用其他人使用高 dotnet 6 版本，如 dotnet 6.0.12 版本构建出来的使用 WPF 的程序集时，将会提示引用存在版本差异从而构建失败。本文将告诉大家如何解决以及原因

<!--more-->
<!-- CreateTime:2023/1/9 8:36:12 -->


<!-- 发布 -->
<!-- 博客 -->

这个构建失败提示是因为 WPF 仓库的运维管理大佬，不小心在从 dotnet 6.0.1 更新到 dotnet 6.0.2 时，修改了 WPF 的程序集版本号。这就导致了在使用 dotnet 6.0.2 和以上的 sdk 版本构建出来的引用 WPF 的程序集时，将在构建出来的程序集自动设置要求使用的 WPF 的版本是 6.0.2 版本

如果自己的开发环境或者是构建 CI CD 环境依然是 6.0.1 版本，将会因为 6.0.1 版本的 WPF 的程序集是 6.0.0 版本，低于所引用的程序集声明的 6.0.2 的版本，从而出现提示类似于标识为 xx 的程序集 xx 所使用的“PresentationFramework, Version=6.0.2.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35”版本高于所引用的标识为“PresentationFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35”的程序集“PresentationFramework” 的错误

提示内容大概如下

```
CSC : error CS1705: 标识为“Lindexi.Lib, Version=1.0.0, Culture=neutral, PublicKeyToken=null”的程序集“Lindexi.Lib”所使用的“PresentationFramework, Version=6.0.2.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35”版本高于所引用的标识为“PresentationFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35”的程序集“PresentationFramework” 
CSC : error CS1705: 标识为“Lindexi.Lib, Version=1.0.0, Culture=neutral, PublicKeyToken=null”的程序集“Lindexi.Lib”所使用的“PresentationCore, Version=6.0.2.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35”版本高于所引用的标识为“PresentationCore, Version=6.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35”的程序集“PresentationCore”
CSC : error CS1705: 标识为“Lindexi.Lib, Version=1.0.0, Culture=neutral, PublicKeyToken=null”的程序集“Lindexi.Lib”所使用的“WindowsBase, Version=6.0.2.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35”版本高于所引用的标识为“WindowsBase, Version=6.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35”的程序集“WindowsBase” 
```

在 WPF 官方了解到此问题的时候，已经发布了 6.0.2 版本了，于是只能固定 WPF 的程序集版本为 6.0.2 版本，即使后面升级到 6.0.12 版本，也是固定为 6.0.2 版本。这里需要敲黑板的是程序集版本不等于发行版本，不等于运行时或 SDK 版本，只是不同的几条线

修复方法有两个：

1. 推荐的方法是更新一下自己的 dotnet sdk 版本
2. 如果是库的开发者，可以尝试在自己的项目的 csproj 上添加以下代码，保持 6.0.0 的 WPF 和 WindowsForms 引用

```xml
<FrameworkReference
          Update="Microsoft.WindowsDesktop.App;Microsoft.WindowsDesktop.App.WPF;Microsoft.WindowsDesktop.App.WindowsForms"
          TargetingPackVersion="6.0.0" />
```

