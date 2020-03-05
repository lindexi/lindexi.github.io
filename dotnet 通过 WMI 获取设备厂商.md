# dotnet 通过 WMI 获取设备厂商

本文告诉大家如何通过 WMI 获取设备厂商

<!--more-->
<!-- CreateTime:2019/8/31 16:55:59 -->

<!-- 标签：dotnet,C#,WMI -->

通过 Win32_ComputerSystem 可以获取电脑系统信息

通过下面代码可以获取 机器型号 和 制造厂商

```csharp
            var mc = "Win32_ComputerSystem";
            var managementObject = new[]
                {
                    "Description",
                    "Manufacturer", //制造厂商
                    "model", //机器型号
                    "UserName", //用户
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
Description  : AT/AT COMPATIBLE
Manufacturer : Gigabyte Technology Co., Ltd.
model        : To be filled by O.E.M.
UserName     : DESKTOP-KA1CD6M\lindexi
```

这里的model就是 PC 序列号

[Win32_ComputerSystem class - Windows applications](https://docs.microsoft.com/en-us/windows/desktop/cimwin32prov/win32-computersystem )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
