# PowerShell 设置 Windows 安全中心白名单排除项

通过管理员的 PowerShell 的 Add-MpPreference 和 Get-MpPreference 可分别设置和获取安全中心设置，进而添加进程或文件夹到 Windows 安全中心白名单

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

我的需求是将我的应用程序加入到 Windows 安全中心白名单，减少应用启动时被杀毒扫描的耗时，以及减少误报

在 Windows Defender（现称为 Microsoft Defender）安全中心里，可通过 Add-MpPreference 和 Get-MpPreference 可分别设置和获取安全中心设置

使用管理员权限的 PowerShell 获取当前有哪些白名单文件夹，在这些文件夹里面的内容将被加入到扫描排除项里，命令如下

```
PS C:\lindexi> Get-MpPreference | Select-Object -Property ExclusionPath -ExpandProperty ExclusionPath
C:\lindexi
D:\lindexi
```

使用管理员权限的 PowerShell 获取当前有哪些进程被加入到白名单，命令如下

```
PS C:\lindexi> Get-MpPreference | Select-Object -Property ExclusionProcess -ExpandProperty ExclusionProcess
C:\lindexi\lindexi.exe
D:\lindexi\lindexi.exe
```

设置某个文件夹加入到排除项，命令如下

```
Add-MpPreference -ExclusionPath "C:\Program Files\MyApp"
```

我在应用安装过程，就将自己的安装路径加入到白名单，如此即可提升启动性能

更多文档请参阅： [Add-MpPreference (Defender) - Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/defender/add-mppreference )