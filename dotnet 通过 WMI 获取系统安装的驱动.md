# dotnet 通过 WMI 获取系统安装的驱动

本文告诉大家如何通过 WMI 获取用户已经安装的驱动程序

<!--more-->
<!-- CreateTime:2019/8/31 16:55:59 -->


<!-- 标签：dotnet,C#,WMI -->

通过 Win32_SystemDriver 可以获取用户已经安装的驱动程序


```csharp
            var mc = "Win32_SystemDriver";
            var managementObject = new[]
            {
                    "Caption",
                    //"CreationClassName",
                    //"Description",
                    //"DisplayName",
                    //"InstallDate",
                    "Name",
                    //"PathName",
                    //"ServiceType",
                    //"StartMode",
                    "State",
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

对于驱动程序只需要知道驱动程序是哪个，是否运行就可以，至于安装日期和在安装在哪很多时候不需要知道

输出 str 的内容

```csharp
Caption : Windows Driver Foundation - User-mode Driver Framework Reflector
Name    : WUDFRd
State   : Running

Caption : WPD 文件系统驱动程序
Name    : WUDFWpdFs
State   : Running

Caption : XINPUT HID 筛选器驱动程序
Name    : xinputhid
State   : Stopped
```

驱动的内容很多，我就不全部放在代码

[Win32_SystemDriver class - Windows applications](https://docs.microsoft.com/en-us/windows/desktop/cimwin32prov/win32-systemdriver )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。


