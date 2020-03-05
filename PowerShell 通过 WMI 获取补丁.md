# PowerShell 通过 WMI 获取补丁

本文告诉大家如何通过 WMI 获取补丁

<!--more-->
<!-- CreateTime:2019/2/21 20:39:51 -->

<!-- csdn -->

<!-- 标签：PowerShell,WMI -->

通过 Win32_QuickFixEngineering 可以获取系统启动的服务

```csharp
Get-WmiObject Win32_QuickFixEngineering
```

运行代码

```csharp
Source        Description      HotFixID      InstalledBy          InstalledOn
------        -----------      --------      -----------          -----------
DESKTOP-KA... Update           KB4483452     NT AUTHORITY\SYSTEM  2019/2/14 0:00:00
DESKTOP-KA... Security Update  KB4465477     NT AUTHORITY\SYSTEM  2018/10/30 0:00:00
DESKTOP-KA... Security Update  KB4465664     NT AUTHORITY\SYSTEM  2018/11/16 0:00:00
DESKTOP-KA... Update           KB4469041     NT AUTHORITY\SYSTEM  2018/12/7 0:00:00
DESKTOP-KA... Update           KB4470502     NT AUTHORITY\SYSTEM  2018/12/18 0:00:00
DESKTOP-KA... Security Update  KB4470788     NT AUTHORITY\SYSTEM  2018/11/17 0:00:00
DESKTOP-KA... Security Update  KB4471331     NT AUTHORITY\SYSTEM  2018/12/7 0:00:00
DESKTOP-KA... Update           KB4480056     NT AUTHORITY\SYSTEM  2019/1/10 0:00:00
DESKTOP-KA... Security Update  KB4480979     NT AUTHORITY\SYSTEM  2019/1/10 0:00:00
DESKTOP-KA... Security Update  KB4487038     NT AUTHORITY\SYSTEM  2019/2/14 0:00:00
DESKTOP-KA... Update           KB4482887     NT AUTHORITY\SYSTEM  2019/2/21 0:00:00
```

[Win32_QuickFixEngineering class - Windows applications](https://docs.microsoft.com/en-us/windows/desktop/cimwin32prov/win32-quickfixengineering )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。