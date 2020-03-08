# 获取 Nuget 版本号

本文告诉大家通过命令行获取 Nuget 的版本号

<!--more-->
<!-- CreateTime:2019/3/1 9:27:06 -->

<!-- 标签：nuget -->

在 Nuget 中没有 `-version` 和 `-v` 和 `--version` 等写法，只需要直接输入 nuget 在第一行就会显示版本号

```csharp
nuget
NuGet Version: 4.9.3.5776
usage: NuGet <command> [args] [options]
Type 'NuGet help <command>' for help on a specific command.

Available commands:

 add         Adds the given package to a hierarchical source. http sources are not supported. For more info, goto https
             ://docs.nuget.org/consume/command-line-reference#add-command.

 config      获取或设置 NuGet 配置值。

 delete      从服务器中删除程序包。

 help (?)    显示一般帮助信息，以及有关其他命令的帮助信息。

 init        Adds all the packages from the <srcPackageSourcePath> to the hierarchical <destPackageSourcePath>. http fe
             eds are not supported. For more info, goto https://docs.nuget.org/consume/command-line-reference#init-comm
             and.

 install     使用指定的源安装程序包。如果未指定源，则将使用 NuGet 配置文件中定义的所有源。如果配置文件未指定源，则使用默认的 NuGet 源。

 list        显示给定源中的程序包列表。如果未指定源，则使用 %AppData%\NuGet\NuGet.config 中定义的所有源。如果 NuGet.config 未指定源，则使用默认 NuGet 源。

 locals      Clears or lists local NuGet resources such as http requests cache, temp cache or machine-wide global packa
             ges folder.

 pack        基于指定的 nuspec 或项目文件创建 NuGet 程序包。

 push        将程序包推送到服务器并进行发布。
             通过加载 %AppData%\NuGet\NuGet.config，然后加载从驱动器的根目录开始到当前目录为止的任何 nuget.config 或 .nuget\nuget.config 来获取 NuGet 的默
             认配置。

 restore     还原 NuGet 程序包。

 setApiKey   保存给定服务器 URL 所对应的 API 密钥。如果未提供 URL，则保存 NuGet 库的 API 密钥。

 sign        Signs a NuGet package with the specified certificate.

 sources     可以管理位于 %AppData%\NuGet\NuGet.config 的源列表

 spec        为新程序包生成 nuspec。如果此命令在项目文件(.csproj、.vbproj、.fsproj)所在的文件夹中运行，则它将创建已标记 化的 nuspec 文件。

 update      将程序包更新到最新的可用版本。此命令还更新 NuGet.exe 本身。

 verify      Verifies a signed NuGet package.

有关详细信息，请访问 http://docs.nuget.org/docs/reference/command-line-reference
```

可以已通过 `nuget update -self` 更新 Nuget 自己的版本

也可以从[官网](https://www.nuget.org/downloads) 下载最新版本

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
