# dotnet 通过 WMI 获取系统信息

本文告诉大家如何通过 WMI 获取系统信息

<!--more-->
<!-- CreateTime:2019/8/31 16:55:59 -->


<!-- 标签：dotnet,C#,WMI -->

通过 Win32_OperatingSystem 可以获取系统信息

```csharp
            var mc = "Win32_OperatingSystem";
            var managementObject = new[]
            {
                    "BootDevice",
                    "BuildNumber",
                    "BuildType",
                    "Caption",
                    "CodeSet",
                    "CountryCode",
                    "CreationClassName",
                    "CSCreationClassName",
                    "CSDVersion",
                    "CSName",
                    "Description",
                    "Locale",
                    "Manufacturer",
                    "Name",
                    "Organization",
                    "OSArchitecture",
                    "OtherTypeDescription",
                    "PlusProductID",
                    "PlusVersionNumber",
                    "RegisteredUser",
                    "SerialNumber",
                    "Status",
                    "SystemDevice",
                    "SystemDirectory",
                    "SystemDrive",
                    "Version",
                    "WindowsDirectory",
            };
            ManagementClass managementClass = new ManagementClass(mc);
            ManagementObjectCollection managementObjectCollection = managementClass.GetInstances();
            var str = new StringBuilder();

            foreach (ManagementObject m in managementObjectCollection)
            {
                foreach (var temp in managementObject)
                {
                    try
                    {
                        str.Append(temp);
                        str.Append(" ");
                        str.Append(m[temp]?.ToString() ?? "");
                        str.Append("\n");
                    }
                    catch (Exception e)
                    {
                        Console.WriteLine(temp + " " + e);
                    }
                }

                str.Append("\n");
            }

            return str.ToString();
```

输出 str 的内容

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

这里的 Version 就是系统版本

[Win32_OperatingSystem class - Windows applications](https://docs.microsoft.com/en-us/windows/desktop/cimwin32prov/win32-operatingsystem )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
