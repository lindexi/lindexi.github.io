# 安装visualStudio 出现 cant install Microsoft.TeamFoundation.OfficeIntegration.Resources

本文告诉大家在安装 VisualStudio 时出现`cant install Microsoft.TeamFoundation.OfficeIntegration.Resources`如何安装

<!-- csdn -->
<!--more-->

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

