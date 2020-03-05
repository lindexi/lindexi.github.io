# dotnet 通过 WMI 拿到显卡信息

本文告诉大家如何通过 WMI 拿到显卡信息

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->

<!-- 标签：dotnet,C#,WMI -->


如果使用的是 dotnet core 请先引用 Microsoft.Windows.Compatibility 才可以使用 WMI 代码

通过下面的代码可以拿到所有的显卡信息

```csharp
ManagementObjectSearcher managementObject =
                    new ManagementObjectSearcher("select * from Win32_VideoController");
```

因为显卡是可能有很多个，通过这个代码就可以找到所有的显卡

创建一个字符串，收集显卡的指定信息

```csharp
                var str = new StringBuilder();

                foreach (ManagementObject m in managementObject.Get())
                {
                    foreach (var temp in new[]
                    {
                        "AcceleratorCapabilities", //AcceleratorCapabilities  --图形和视频控制器的三维阵列的能力
                        "AdapterCompatibility", //AdapterCompatibility  --用于此控制器与系统比较兼容性一般芯片组
                        "AdapterDACType", //AdapterDACType  --姓名或数字 - 模拟转换器（DAC）芯片的标识符
                        "AdapterRAM", //AdapterRAM  --视频适配器的内存大小
                        "Availability", //Availability  --可用性和设备的状态
                        "CapabilityDescriptions", //CapabilityDescriptions  --自由形式字符串提供更详细的解释中的任何加速器能力阵列所指示的视频加速器功能
                        "Caption", //Caption  --对象的简短描述
//ColorTableEntries  --尺寸系统的色表
//ConfigManagerErrorCode  --Win32的配置管理器错误代码
//ConfigManagerUserConfig  --如果为TRUE，该装置是使用用户定义的配置
//CreationClassName  --第一个具体类的名称出现在创建实例所使用的继承链
//CurrentBitsPerPixel  --使用的比特数以显示每个像素
//CurrentHorizontalResolution  --水平像素的当前数量
//CurrentNumberOfColors  --在当前的分辨率支持的色彩数目
//CurrentNumberOfColumns  --此视频控制器列（如果在字符模式下）编号
//CurrentNumberOfRows  --此视频控制器行（如果在字符模式下）编号
//CurrentRefreshRate  --频率在该视频控制器刷新监视器的图像
//CurrentScanMode  --当前扫描模式
//CurrentVerticalResolution  --当前垂直像素数量
//Description  --描述
                        "DeviceID", //DeviceID  --该视频控制器标识符（唯一的计算机系统）
//DeviceSpecificPens  --目前许多设备专用笔。值0xFFFF表示设备不支持笔。
//DitherType  --抖动型视频控制器
                        "DriverDate", //DriverDate  --当前已安装的视频驱动程序的最后修改日期和时间
                        "DriverVersion", //DriverVersion  --视频驱动程序的版本号
//ErrorCleared  --如果为真，报上一个错误代码属性中的错误现已清除
//ErrorDescription  --可能采取的纠正措施字符串提供有关记录在一个错误代码属性错误的详细信息
//ICMIntent  --应使用默认三种可能的配色方法或意图中的一个特定值
//ICMMethod  --处理ICM方法。对于非ICM的应用程序，这个属性决定了ICM是否已启用对于ICM的应用程序，系统将检查此属性来确定如何处理ICM支持
//InfFilename  --视频适配器的路径.inf文件
//InfSection  --Windows的视频信息所在的.inf文件
                        "InstallDate", //InstallDate  --安装的日期
                        "InstalledDisplayDrivers", //InstalledDisplayDrivers  --已安装的显示设备驱动程序的名称
//LastErrorCode  --报告的逻辑设备上一个错误代码 
//MaxMemorySupported  --以字节为单位支持的内存最高限额
//MaxNumberControlled  --可支持通过该控制器可直接寻址的实体的最大数量
//MaxRefreshRate  --在赫兹视频控制器的最大刷新率
//MinRefreshRate   --在赫兹视频控制器的最小刷新率
//Monochrome  --如果是TRUE，灰阶用于显示图像。
//Name  --标签由该对象是已知的。当子类，该属性可以被覆盖是一个关键属性。
//NumberOfColorPlanes  --当前一些颜色平面。如果该值不适用于当前视频的配置，输入0（零）
//NumberOfVideoPages  --当前的分辨率和可用内存支持视频页数
//PNPDeviceID  --即插即用逻辑设备的播放装置识别符
//PowerManagementCapabilities  --逻辑设备的特定功率相关的能力阵列
//PowerManagementSupported  --如果为TRUE，该装置可以是电源管理（可以投入挂起模式，等等）
//ProtocolSupported  --由控制器使用协议访问“控制”的设备
//ReservedSystemPaletteEntries  --系统调色板保留的条目数
//SpecificationVersion  --初始化数据规范的版本号（在其上的结构的基础）
//Status  --对象的当前状态
//StatusInfo  --对象的当前状态详细信息
//SystemCreationClassName  --该作用域计算机的创建类别名称属性的值
//SystemName  --系统的名称
//SystemPaletteEntries  --当前一些系统调色板颜色索引条目
//TimeOfLastReset  --该控制器是最后一次复位日期和时间，这可能意味着该控制器被断电或重新初始化
//VideoArchitecture  --视频体系结构的类型
                        //"VideoMemoryType", //VideoMemoryType  --显存类型
//VideoMode  --当前视频模式
//VideoModeDescription  --当前的分辨率，颜色和视频控制器的扫描模式设置
//VideoProcessor  --无格式的字符串描述视频处理器
                    })
                    {
                        str.Append(temp);
                        str.Append(" ");
                        str.Append(m[temp]?.ToString() ?? "");
                        str.Append("\n");
                    }
                }

                return str.ToString();
```

因为显卡的很多信息都是不需要的，所以就注释了，小伙伴可以拿到自己需要的信息

运行上面的代码大概可以拿到这些信息

```csharp
AcceleratorCapabilities :
AdapterCompatibility    : Intel Corporation
AdapterDACType          : Internal
AdapterRAM              : 1073741824
Availability            : 3
CapabilityDescriptions  :
Caption                 : Intel(R) HD Graphics 530
DeviceID                : VideoController1
DriverDate              : 20181119000000.000000-000
DriverVersion           : 23.20.16.4973
InstallDate             :
InstalledDisplayDrivers : C:\WINDOWS\System32\DriverStore\FileRepository\igdlh64.inf_amd64_2c92d70c30b8effe\igdumdim64.
                          dll,C:\WINDOWS\System32\DriverStore\FileRepository\igdlh64.inf_amd64_2c92d70c30b8effe\igd10iu
                          md64.dll,C:\WINDOWS\System32\DriverStore\FileRepository\igdlh64.inf_amd64_2c92d70c30b8effe\ig
                          d10iumd64.dll,C:\WINDOWS\System32\DriverStore\FileRepository\igdlh64.inf_amd64_2c92d70c30b8ef
                          fe\igd12umd64.dll
```

通过 Caption 和驱动安装时间就可以知道当前的显卡驱动

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
