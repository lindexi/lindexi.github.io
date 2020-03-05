# 影子系统让 C++ 程序无法运行

最近发现在一些设备上，我运行的 C++ 程序提示 VC++ runtime error r6025 无法运行

<!--more-->
<!-- CreateTime:2019/5/15 15:24:35 -->

<!-- csdn -->

运行程序提示下面代码

<!-- ![](image/影子系统让程序无法运行/影子系统让程序无法运行0.png) -->

![](http://image.acmx.xyz/lindexi%2F2019515151657277)

从系统日志可以看到下面代码

```csharp
错误应用程序名称: lindexi.exe，版本: 5.1.12.63002，时间戳: 0xedd2d687
错误模块名称: MSVCR100.dll，版本: 10.0.40219.325，时间戳: 0x4df2be1e
异常代码: 0x40000015
错误偏移量: 0x0008d6fd
错误进程 ID: 0x994
错误应用程序启动时间: 0x01d50ac3bd970061
错误应用程序路径: C:\Program Files\lindexi\lindexi.exe
错误模块路径: C:\Program Files\PowerShadow\App\MSVCR100.dll
报告 ID: a0c5c0b1-76b7-11e9-9d20-94c69123de40
```

也就是在读取 MSVCR100 实际读的是影子系统的 dll 但是他的 dll 无法使用

解决方法是先卸载影子系统，然后卸载系统的 VC++ 2005 然后重新安装 VC++ 2005 就可以

VC++ 2005 下载地址

- [官网](http://download.microsoft.com/download/5/2/1/5212066c-5f48-4b16-a059-ed84b505a65d/vcredist_x86.exe)
- [华军](http://www.onlinedown.net/soft/1093138.htm )
- [CSDN下载](https://download.csdn.net/download/zhh271075949/10830062?utm_source=bbsseo )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
