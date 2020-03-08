# dotnet 通过 WMI 获取系统补丁

本文告诉大家如何通过 WMI 获取补丁

<!--more-->
<!-- CreateTime:2019/8/31 16:55:59 -->


<!-- 标签：dotnet,C#,WMI -->

通过 Win32_QuickFixEngineering 可以获取系统启动的服务

下面代码只是获取补丁的 kb 字符

```csharp
                const string query = "SELECT HotFixID FROM Win32_QuickFixEngineering";
                var search = new ManagementObjectSearcher(query);
                var collection = search.Get();

                var str = new StringBuilder();
                foreach (ManagementObject quickFix in collection)
                {
                    str.Append(quickFix["HotFixID"].ToString());
                    str.Append(";");
                }

                return str.ToString();
```

输出 str 的内容

```csharp
KB4465477,KB4465664
```

这个方法无法在 XP 系统获取补丁

[Win32_QuickFixEngineering class - Windows applications](https://docs.microsoft.com/en-us/windows/desktop/cimwin32prov/win32-quickfixengineering )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
