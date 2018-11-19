# win10 uwp 使用 msbuild 命令行编译 UWP 程序

本文告诉大家如何使用 msbuild 命令行编译一个 UWP 程序

<!--more-->
<!-- 标签：UWP,VisualStudio,msbuild -->

在有一些时候，如使用持续集成的时候就不能通过 VisualStudio 的方式编译 UWP 程序，需要使用命令行的方式编译。

尝试在本地从开始菜单打开开发命令提示符，或者从使用命令行调用本机的 VisualStudio 编译命令行

```bash
cmd> "C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\Tools\LaunchDevCmd.bat"
```

在 VisualStudio 在 C 盘安装就可以在 `C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\Tools\` 找到 LunchDevCmd.bat 文件

运行之后可以看到下面界面

```bash
> "C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\Tools\LaunchDevCmd.bat"
**********************************************************************
** Visual Studio 2017 Developer Command Prompt v15.8.5
** Copyright (c) 2017 Microsoft Corporation
**********************************************************************

C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise>
```

先进入项目所在的文件夹，也就是 sln 或 csproj 文件所在的文件夹，通过 msbuild 可以编译 sln 或编译 csproj 项目，推荐是编译 sln 的方式

在命令行跳转盘符，如从原来的C盘到 D 盘可以通过 `盘符:` 的方式

```csharp
cmd> D:
```

这样就可以跳转到 D 盘，在进入指定的文件夹，可以输入 `cd 文件夹` 的方式

如果自己输入很容易就输入错误，推荐输入 `cd ` 然后在资源管理器打开文件夹，将地址栏的文件夹拖进命令行

在编译 UWP 之前，很重要的是清理原有的文件，假如文件都是通过 git 管理的，当前也不存在没有被跟踪的文件，可以使用下面的代码删除无关的文件，需要注意的是通过这个方式必须保证证书文件是被跟踪的

```bash
cmd> git clean -xdf
```

清理之后可以通过下面的代码还原 UWP 项目，还原这一步非常重要

```csharp
cmd> msbuild /t:restore
```

但是默认欢迎的 ARM 的项目，很多时候需要的是 x86 的项目，可以通过下面的方式还原

```csharp
cmd> msbuild /t:restore /p:Platform=x86
```

如果要还原x64的程序，可以使用下面代码

```csharp
cmd> msbuild /t:restore /p:Platform=x64

```

现在就可以进行编译了，通过下面的代码进行编译

```csharp
cmd> msbuild /p:Platform=x86
```

现在就可以编译 DEBUG 下的 x86 程序了

如果需要编译同时输出，可以尝试下面的代码。下面的 AppxPackageDir 是填写 AppxPackageDir 的文件夹路径，请将这个值修改为自己需要的。

```bash
/p:AppxBundlePlatforms="x86|x64|ARM" /p:AppxPackageDir="D:\lindexi\AppxPackages\\" /p:AppxBundle=Always /p:UapAppxPackageBuildMode=StoreUpload /p:platform="x86" /p:configuration="release" /p:VisualStudioVersion="15.0" 
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
