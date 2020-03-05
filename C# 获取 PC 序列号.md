# C# 获取 PC 序列号

在 C++ 需要使用 GetSystemFirmwareTable 的方法来获得 PC 的序列号，需要写的代码很多，但是在 C# 可以使用 WMI 来拿到序列号

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


<!-- 标签：dotnet,C#,WMI -->

首先是安装 System.Management ，安装了这个库，在 dotnet framework 和 dotnet core 都可以使用本文的方法获取 PC 的序列号

安装 System.Management 的方法是通过 Nuget 搜索`System.Management`然后安装，如果使用的是VisualStudio 2017项目格式就可以复制下面代码到项目文件

```csharp
    <PackageReference Include="System.Management" Version="4.5.0" />

```

我比较喜欢第二个方法来安装，因为这个速度比较快

安装完成使用 WMI 拿到序列号需要的代码很少，请看下面

```csharp
                var search = new ManagementObjectSearcher("SELECT * FROM Win32_BIOS");
                var mobos = search.Get();
                foreach (var temp in mobos)
                {
                    object serial = temp["SerialNumber"]; // ProcessorID if you use Win32_CPU
                    pcsn = serial.ToString();
                    Console.WriteLine(pcsn);
                }
```

这样就可以拿到序列号，但是可能有些设备是没有序列号，很多时候是返回下面代码

```csharp
To be filled by O.E.M
To be filled by O.E.M.
Default
Default string

```

上面代码最后一行就是空白，所以需要先做判断是不是存在序列号，所有代码请看下面

```csharp
       /// <summary>
        /// 获得pc号
        /// </summary>
        public static string GetPcsnString()
        {
            var pcsn = "";
            try
            {
                var search = new ManagementObjectSearcher("SELECT * FROM Win32_BIOS");
                var mobos = search.Get();
                foreach (var temp in mobos)
                {
                    object serial = temp["SerialNumber"]; // ProcessorID if you use Win32_CPU
                    pcsn = serial.ToString();
                    Console.WriteLine(pcsn);

                    if
                    (
                        !string.IsNullOrEmpty(pcsn)
                        && pcsn != "To be filled by O.E.M" //没有找到
                        && !pcsn.Contains("O.E.M")
                        && !pcsn.Contains("OEM")
                        && !pcsn.Contains("Default")
                    )
                    {
                        break;
                    }
                    else
                    {
                        Console.WriteLine("默认值");
                    }
                }
            }
            catch (Exception e)
            {
                Debug.WriteLine(e);
                // 无法处理
            }

            return pcsn;
        }
```

需要知道，使用这个方法获取 序列号是比较耗性能的，建议放在其他线程获取 

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
