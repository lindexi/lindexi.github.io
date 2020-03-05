# 安装visualStudio 出现 cant install Microsoft.TeamFoundation.OfficeIntegration.Resources

本文告诉大家在安装 VisualStudio 时出现`cant install Microsoft.TeamFoundation.OfficeIntegration.Resources`如何安装

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

如果在安装之前卸载了以前的VisualStudio 或者之前有安装VisualStudio 2015 现在安装 VisualStudio 2017 ，那么本文可能解决。

首先下载VisualStudio专门的卸载，工具在[https://github.com/Microsoft/VisualStudioUninstaller](https://github.com/Microsoft/VisualStudioUninstaller)

如果无法下载可以联系我，让我发给你。

打开管理员 cmd ，然后解压下载的工具在一个文件夹，例如 `C:\software` 然后使用命令进入文件夹，执行程序 ForcedUninstall

```

cd c:\

cd software

Setup.ForcedUninstall.exe
```

然后程序就会开始，需要按`y`，接着他就会卸载，完成就可以安装新的vs

如果以前安装的是 VisualStudio 2017 ，可以使用`C:\Program Files (x86)\Microsoft Visual Studio\Installer\resources\app\layout\InstallCleanup.exe` 卸载，需要使用管理员的命令行打开

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。