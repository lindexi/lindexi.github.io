# PowerShell 通过 WMI 获取系统信息

本文告诉大家如何通过 WMI 使用 Win32_OperatingSystem 获取设备厂商

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


<!-- 标签：PowerShell,WMI -->

通过下面代码可以获取 系统版本和系统是专业版还是教育版

```csharp
Get-WmiObject Win32_OperatingSystem | Format-List BootDevice,BuildNumber,BuildType,Caption,CodeSet,CountryCode,CreationClassName,CSCreationClassName,CSDVersion,CSName,Description,Locale,Manufacturer,Name,Organization,OSArchitecture,OtherTypeDescription,PlusProductID,PlusVersionNumber,RegisteredUser,SerialNumber,Status,SystemDevice,SystemDirectory,SystemDrive,Version,WindowsDirectory
```

运行代码

```csharp
BootDevice           : \Device\HarddiskVolume2
BuildNumber          : 17763
BuildType            : Multiprocessor Free
Caption              : Microsoft Windows 10 专业版
CodeSet              : 936
CountryCode          : 86
CreationClassName    : Win32_OperatingSystem
CSCreationClassName  : Win32_ComputerSystem
CSDVersion           :
CSName               : DESKTOP-KA1CD6M
Description          :
Locale               : 0804
Manufacturer         : Microsoft Corporation
Name                 : Microsoft Windows 10 专业版|C:\WINDOWS|\Device\Harddisk0\Partition4
Organization         :
OSArchitecture       : 64 位
OtherTypeDescription :
PlusProductID        :
PlusVersionNumber    :
RegisteredUser       : lindexi_gd@outlook.com
SerialNumber         : 00331-10000-00001-AA523
Status               : OK
SystemDevice         : \Device\HarddiskVolume4
SystemDirectory      : C:\WINDOWS\system32
SystemDrive          : C:
Version              : 10.0.17763
WindowsDirectory     : C:\WINDOWS
```

[Win32_OperatingSystem class - Windows applications](https://docs.microsoft.com/en-us/windows/desktop/cimwin32prov/win32-operatingsystem )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

