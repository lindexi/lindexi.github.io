
# PowerShell 设置 Windows 安全中心白名单排除项

通过管理员的 PowerShell 的 Add-MpPreference 和 Get-MpPreference 可分别设置和获取安全中心设置，进而添加进程或文件夹到 Windows 安全中心白名单

<!--more-->


<!-- CreateTime:2025/12/31 07:21:58 -->

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




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。