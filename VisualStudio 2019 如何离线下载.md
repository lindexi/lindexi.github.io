# VisualStudio 2019 如何离线下载

本文告诉大家如何离线下载 VisualStudio 2019 离线安装

<!--more-->
<!-- CreateTime:2019/11/29 8:38:13 -->

<!-- csdn -->

微软就给 VisualStudio 2019 一个在线安装工具，需要通过命令行输入参数才可以离线下载

先从[官网](https://docs.microsoft.com/en-us/visualstudio/install/create-an-offline-installation-of-visual-studio?view=vs-2019 )选择自己需要下载的 VisualStudio 版本，可以选择社区版、专业版和企业版。只有社区版是免费的

![](http://image.acmx.xyz/lindexi%2F20194394145961)
 
如我选择 Visual Studio Community 社区版，于是下载的 exe 就是 vs_community.exe 如果下载的是 Visual Studio Professional 对应的 exe 就是 vs_professional.exe 请替换本文的代码里面用到的程序为你下载的版本

## 存放的路径

离线下载需要指定一个文件夹，用于存放下载的文件，通过 `--layout` 加上绝对路径可以下载到输入的文件夹

如下面代码设置下载到 `C` 盘的文件夹，请将这个文件夹修改为一个普通的文件夹，注意这个文件夹需要在当前的 User 有写入权限

```csharp
--layout c:\vslayout
```

## 添加功能

可选的功能通过 `--add` 添加功能

- Microsoft.VisualStudio.Workload.Azure Azure development
- Microsoft.VisualStudio.Workload.ManagedDesktop 桌面端开发
- Microsoft.VisualStudio.Workload.NetWeb ASP.NET Core 开发
- Microsoft.VisualStudio.Workload.NetCoreTools .NET Core 开发
- Microsoft.VisualStudio.Workload.Universal UWP 开发

在这个页面可以找到更多的功能 [Visual Studio Community workload and component IDs](https://docs.microsoft.com/en-us/visualstudio/install/workload-component-id-vs-community?view=vs-2019#aspnet-and-web-development )

多个功能通过多个 `--add` 添加，如我需要选择 .NET Core 开发、桌面端开发和 ASP.NET Core 开发同时需要 UWP 开发等，我可以这样写

```csharp
--add Microsoft.VisualStudio.Workload.ManagedDesktop
--add Microsoft.VisualStudio.Workload.NetWeb
--add Component.GitHub.VisualStudio
--add Microsoft.VisualStudio.Workload.Office 
--add Microsoft.VisualStudio.Workload.NetCoreTools 
--add Microsoft.VisualStudio.Workload.Universal
--add Microsoft.VisualStudio.Workload.VisualStudioExtension
```

注意在命令行是不能有空格的，上面代码为了方便理解加了换行

如果还需要某个功能里面的选项，如我需要 UWP 里面的 17763 的 SDK 可以从[Visual Studio Community workload and component IDs](https://docs.microsoft.com/en-us/visualstudio/install/workload-component-id-vs-community?view=vs-2019#aspnet-and-web-development )找到对应的功能，也通过 `--add` 添加

```csharp
--add Microsoft.VisualStudio.Component.Windows10SDK.17763
```

## 多语言

通过 `--lang` 可以添加多语言

如添加中文是 `zh-CN` 英文是 `en-US` 可以通过 `--addProductLang` 多添加语言

下面代码就是添加中文作为主要语言，同时添加英文

```csharp
--lang zh-CN --addProductLang en-US
```

如果同时添加多个语言可以在 `--lang` 后面添加多个

```csharp
--lang zh-CN en-US
```

于是一个下载.NET Core 开发、桌面端开发和 ASP.NET Core 开发和 UWP 开发等，下载到 `F:\下载\vs\vslayout` 的命令可以这样写

```csharp
vs_community.exe --layout F:\下载\vs\vslayout --add Microsoft.VisualStudio.Workload.ManagedDesktop --add Microsoft.VisualStudio.Workload.NetWeb --add Component.GitHub.VisualStudio  --add Microsoft.VisualStudio.Workload.Office --add Microsoft.VisualStudio.Workload.NetCoreTools --add Microsoft.VisualStudio.Workload.Universal --add Microsoft.VisualStudio.Component.Windows10SDK.17763 --add Microsoft.VisualStudio.Workload.VisualStudioExtension --includeOptional --lang zh-CN --addProductLang en-US
```

其他语言请看 [Use command-line parameters to install Visual Studio](https://docs.microsoft.com/en-us/visualstudio/install/use-command-line-parameters-to-install-visual-studio?view=vs-2019#list-of-language-locales )

## 添加所有功能

如果想要添加所有的功能，那么去掉 `--includeOptional` 请看代码

```csharp
vs_community.exe --layout c:\vslayout --lang en-US
```

下载英文版的全功能的 VisualStudio 放在 `c:\vslayout` 这句命令需要等很久

## 下载恢复

如果下载了一半然后关机，需要恢复，可以尝试使用 `--fix` 修复，用法是输入下载离线的相同命令在命令后面添加 `--fix` 就可以

```csharp
vs_community.exe --layout F:\下载\vs\vslayout --add Microsoft.VisualStudio.Workload.ManagedDesktop --add Microsoft.VisualStudio.Workload.NetWeb --add Component.GitHub.VisualStudio  --add Microsoft.VisualStudio.Workload.Office --add Microsoft.VisualStudio.Workload.NetCoreTools --add Microsoft.VisualStudio.Workload.Universal --add Microsoft.VisualStudio.Component.Windows10SDK.17763 --add Microsoft.VisualStudio.Workload.VisualStudioExtension --includeOptional --lang zh-CN --addProductLang en-US --fix
```

## 安装离线下载

通过命令行打开离线下载的文件夹里面的对应的安装文件，如我使用的是社区版，下载到 `F:\下载\vs\vslayout` 可以通过下面命令安装

```csharp
F:\下载\vs\vslayout\vs_community.exe --add Microsoft.VisualStudio.Workload.ManagedDesktop --add Microsoft.VisualStudio.Workload.NetWeb --add Component.GitHub.VisualStudio  --add Microsoft.VisualStudio.Workload.Office --add Microsoft.VisualStudio.Workload.NetCoreTools --add Microsoft.VisualStudio.Workload.Universal --add Microsoft.VisualStudio.Component.Windows10SDK.17763 --add Microsoft.VisualStudio.Workload.VisualStudioExtension --includeOptional
```

注意这里添加的 `--add` 需要和上面下载的时候输入的相同

关于 VisualStudio 2019 新功能请看 [VisualStudio 2019 新特性](https://blog.lindexi.com/post/VisualStudio-2019-%E6%96%B0%E7%89%B9%E6%80%A7.html )

[Visual Studio 2019 发布活动 - Visual Studio](https://visualstudio.microsoft.com/zh-hans/vs2019-launch/?rr=https%3A%2F%2Fwww.baidu.com%2Flink%3Furl%3D54dGgZyPCZ0eQDyOjhrxFSjIslKmN5iGZ0EHDN672ecSriXESCrcVM9jCtpMBOCk%26wd%3D%26eqid%3D830caa27000393af000000065ca40ad5 )

[Use command-line parameters to install Visual Studio](https://docs.microsoft.com/en-us/visualstudio/install/use-command-line-parameters-to-install-visual-studio?view=vs-2019 )

[Visual Studio Community workload and component IDs](https://docs.microsoft.com/en-us/visualstudio/install/workload-component-id-vs-community?view=vs-2019#aspnet-and-web-development )

激活码，激活码只用于尝试 VisualStudio 的使用，请不要在商业环境使用

Visual Studio 2019 Enterprise

BF8Y8-GN2QH-T84XB-QVY3B-RC4DF

Visual Studio 2019 Professional

NYWVH-HT4XC-R2WYW-9Y3CM-X4V3Y

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
