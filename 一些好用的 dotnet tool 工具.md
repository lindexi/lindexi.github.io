# 一些好用的 dotnet tool 工具

本文收藏一些好用的 dotnet tool 工具

<!--more-->
<!-- CreateTime:2020/3/5 21:24:30 -->

如何寻找到更多工具，请参阅 [dotnet 在 NuGet 上搜寻好用的 dotnet tool 工具](https://blog.lindexi.com/post/dotnet-%E5%9C%A8-NuGet-%E4%B8%8A%E6%90%9C%E5%AF%BB%E5%A5%BD%E7%94%A8%E7%9A%84-dotnet-tool-%E5%B7%A5%E5%85%B7.html )

## 收藏夹

以下这些是大佬们收藏的工具：

- [natemcmaster/dotnet-tools: A list of tools to extend the .NET Core command line (dotnet)](https://github.com/natemcmaster/dotnet-tools)
- [RehanSaeed/awesome-dotnet-core: A collection of awesome .NET core libraries, tools, frameworks and software](https://github.com/RehanSaeed/awesome-dotnet-core#tools)
- [spectresystems/snitch: A tool that help you find duplicate transitive package references.](https://github.com/spectresystems/snitch )

更多工具可以在 [ToolGet](https://www.toolget.net) 里面进行搜寻

## UpdateAllDotNetTools

一句命令更新所有 dotnet tool 到最新版本

安装方法：

```
dotnet tool install -g dotnetCampus.UpdateAllDotNetTools
```

## NugetMergeFixTool

修复大型项目的 NuGet 合并，也可以用来快速升级 NuGet 库

[dotnet-campus/dotnetCampus.NugetMergeFixTool: 传说博哥的工具 可以用来修复 git 合并的时候将 csproj 合并坏了的问题，也可以用来快速升级 NuGet 库](https://github.com/dotnet-campus/dotnetCampus.NugetMergeFixTool )


## snitch

参阅 [自动找到项目里面重复的 NuGet 依赖项](https://blog.lindexi.com/post/dotnet-tool-%E8%87%AA%E5%8A%A8%E6%89%BE%E5%88%B0%E9%A1%B9%E7%9B%AE%E9%87%8C%E9%9D%A2%E9%87%8D%E5%A4%8D%E7%9A%84-NuGet-%E4%BE%9D%E8%B5%96%E9%A1%B9.html )


## dotnet-serve

一句命令开启文件服务器

可以方便在多个设备之间分享文件，例如将电脑的文件传给手机

参阅 [dotnet serve 一句话开启文件服务器 通过 HTTP 将文件共享给其他设备](https://blog.lindexi.com/post/dotnet-serve-%E4%B8%80%E5%8F%A5%E8%AF%9D%E5%BC%80%E5%90%AF%E6%96%87%E4%BB%B6%E6%9C%8D%E5%8A%A1%E5%99%A8-%E9%80%9A%E8%BF%87-HTTP-%E5%B0%86%E6%96%87%E4%BB%B6%E5%85%B1%E4%BA%AB%E7%BB%99%E5%85%B6%E4%BB%96%E8%AE%BE%E5%A4%87.html )

## dotnet-exec

[javadparvaresh/dotnet-exec: Execute shell script as dotnet tasks](https://github.com/javadparvaresh/dotnet-exec )

## CSharpier

功能和 dotnet format 工具差不多，是一个代码格式化工具

详细请看 [CSharpier](https://csharpier.com/) 官网

[https://github.com/belav/csharpier](https://github.com/belav/csharpier)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
