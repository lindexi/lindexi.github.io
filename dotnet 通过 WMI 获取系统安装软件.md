# dotnet 通过 WMI 获取系统安装软件

本文告诉大家如何通过 WMI 获取系统安装的软件，这个方法不能获取全部的软件

<!--more-->
<!-- CreateTime:2019/4/29 12:18:59 -->


<!-- 标签：dotnet,C#,WMI -->

通过 Win32_Product 可以获取系统安装的软件


```csharp
            var mc = "Win32_Product";
            var managementObject = new[]
            {
                    "Caption",
                    "Description",
                    "IdentifyingNumber",
                    "InstallDate",
                    "InstallLocation",
                    "HelpLink",
                    "HelpTelephone",
                    "InstallSource",
                    "Language",
                    "LocalPackage",
                    "Name",
                    "PackageCache",
                    "PackageCode",
                    "PackageName",
                    "ProductID",
                    "RegOwner",
                    "RegCompany",
                    "SKUNumber",
                    "Transforms",
                    "URLInfoAbout",
                    "URLUpdateInfo",
                    "Vendor",
                    "WordCount",
                    "Version",
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
Caption           : 坚果云
Description       : 坚果云
IdentifyingNumber : {FEA8B01C-3F43-470A-BB28-679B1AEEC6E8}
InstallDate       : 20180305
InstallLocation   : C:\Program Files\Nutstore\
HelpLink          : http://help.jianguoyun.com
HelpTelephone     :
InstallSource     : C:\Users\linde\AppData\Roaming\NutstoreClient\install\AEEC6E8\
Language          : 2052
LocalPackage      : C:\WINDOWS\Installer\4acb3a9.msi
Name              : 坚果云
PackageCache      : C:\WINDOWS\Installer\4acb3a9.msi
PackageCode       : {3802EFD2-0953-4527-835E-E4C459062CD5}
PackageName       : Nutstore.x64.msi
ProductID         :
RegOwner          :
RegCompany        :
SKUNumber         :
Transforms        : C:\WINDOWS\Installer\{FEA8B01C-3F43-470A-BB28-679B1AEEC6E8}\Nutstore.mst
URLInfoAbout      : https://www.jianguoyun.com/
URLUpdateInfo     :
Vendor            : 上海亦存网络科技有限公司
WordCount         : 0
Version           : 4.0.8


Caption           : Apple 应用程序支持 (32 位)
Description       : Apple 应用程序支持 (32 位)
IdentifyingNumber : {5A659BE5-849B-484E-A83B-DCB78407F3A4}
InstallDate       : 20190221
InstallLocation   : C:\Program Files (x86)\Common Files\Apple\Apple Application Support
HelpLink          : http://www.apple.com/cn/support/
HelpTelephone     : (86) 800 810 2323
InstallSource     : C:\Users\linde\AppData\Local\Temp\IXP246.TMP\
Language          : 2052
LocalPackage      : C:\WINDOWS\Installer\1a1ef7b.msi
Name              : Apple 应用程序支持 (32 位)
PackageCache      : C:\WINDOWS\Installer\1a1ef7b.msi
PackageCode       : {F3D0B996-B6DB-4283-9565-004518A6610B}
PackageName       : AppleApplicationSupport.msi
ProductID         :
RegOwner          :
RegCompany        :
SKUNumber         :
Transforms        :
URLInfoAbout      : http://www.apple.com/cn/
URLUpdateInfo     : http://www.apple.com/cn/
Vendor            : Apple Inc.
WordCount         : 0
Version           : 7.3
```

[Win32_Product class (Windows)](https://msdn.microsoft.com/en-us/library/aa394378(v=vs.85).aspx )

更多 WMI 博客请看

- [dotnet 通过 WMI 获取指定进程的输入命令行](https://blog.lindexi.com/post/dotnet-%E9%80%9A%E8%BF%87-WMI-%E8%8E%B7%E5%8F%96%E6%8C%87%E5%AE%9A%E8%BF%9B%E7%A8%8B%E7%9A%84%E8%BE%93%E5%85%A5%E5%91%BD%E4%BB%A4%E8%A1%8C.html)

- [dotnet 通过 WMI 拿到显卡信息](https://blog.lindexi.com/post/dotnet-%E9%80%9A%E8%BF%87-WMI-%E6%8B%BF%E5%88%B0%E6%98%BE%E5%8D%A1%E4%BF%A1%E6%81%AF.html)

- [dotnet 通过 WMI 获取系统安装软件](https://blog.lindexi.com/post/dotnet-%E9%80%9A%E8%BF%87-WMI-%E8%8E%B7%E5%8F%96%E7%B3%BB%E7%BB%9F%E5%AE%89%E8%A3%85%E8%BD%AF%E4%BB%B6.html)

- [dotnet 通过 WMI 获取系统补丁](https://blog.lindexi.com/post/dotnet-%E9%80%9A%E8%BF%87-WMI-%E8%8E%B7%E5%8F%96%E7%B3%BB%E7%BB%9F%E8%A1%A5%E4%B8%81.html)

- [dotnet 通过 WMI 获取系统启动的服务](https://blog.lindexi.com/post/dotnet-%E9%80%9A%E8%BF%87-WMI-%E8%8E%B7%E5%8F%96%E7%B3%BB%E7%BB%9F%E5%90%AF%E5%8A%A8%E7%9A%84%E6%9C%8D%E5%8A%A1.html)

- [dotnet 通过 WMI 获取系统安装的驱动](https://blog.lindexi.com/post/dotnet-%E9%80%9A%E8%BF%87-WMI-%E8%8E%B7%E5%8F%96%E7%B3%BB%E7%BB%9F%E5%AE%89%E8%A3%85%E7%9A%84%E9%A9%B1%E5%8A%A8.html)

- [dotnet 通过 WMI 获取系统信息](https://blog.lindexi.com/post/dotnet-%E9%80%9A%E8%BF%87-WMI-%E8%8E%B7%E5%8F%96%E7%B3%BB%E7%BB%9F%E4%BF%A1%E6%81%AF.html)

- [dotnet 通过 WMI 获取设备厂商](https://blog.lindexi.com/post/dotnet-%E9%80%9A%E8%BF%87-WMI-%E8%8E%B7%E5%8F%96%E8%AE%BE%E5%A4%87%E5%8E%82%E5%95%86.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。


