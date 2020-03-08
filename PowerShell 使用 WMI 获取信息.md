# PowerShell 使用 WMI 获取信息

在 PowerShell 可以很容易使用 WMI 拿到系统的信息，如果有关注我的网站，就会发现我写了很多通过 WMI 拿到系统的显卡，系统安装的软件等方法，本文告诉大家如果通过 PowerShell 拿到 WMI 类里面的属性

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


<!-- 标签：PowerShell,WMI -->

在 Windows 系统通过 Windows Management Instrumentation (WMI) 统一管理系统的配置，在 PowerShell 能使用 WMI 的功能进行获取系统

很少有人知道 WMI 里面包含了多少可以使用的类，包括我之前写的很多博客，实际上也只是里面的很少，通过下面的例子告诉大家如何获取设备里面包含的类

## 获取 WMI 类

在使用 WMI 之前需要知道 WMI 是能做什么的，这个方法能做的就是描述系统能被管理的资源，在系统里面包含了几百个类，一个类里面包含很多属性

通过 Get-WmiObject 可以找到设备里面所有可以被找到的 WMI 类

```csharp
Get-WmiObject -List
```

在 Windows 10 设备，右击开始菜单，打开 PowerShell 输入上面代码，就可以看到输出

在 Get-WmiObject 的参数可以加上计算机是哪个，支持访问局域网可以访问的计算机的信息

```csharp
Get-WmiObject -List -ComputerName 192.168.1.29
```

尝试在自己的系统输入一下，可以看到很多代码

```csharp
PS> Get-WmiObject -List

Name                                Methods              Properties
----                                -------              ----------
__thisNAMESPACE                     {}                   {SECURITY_DESCRIPTOR}
__Provider                          {}                   {Name}
__Win32Provider                     {}                   {ClientLoadableCLSID, CLSID, Concurrency, DefaultMachineNam...
__ProviderRegistration              {}                   {provider}
__EventProviderRegistration         {}                   {EventQueryList, provider}
__ObjectProviderRegistration        {}                   {InteractionType, provider, QuerySupportLevels, SupportsBat...
__ClassProviderRegistration         {}                   {CacheRefreshInterval, InteractionType, PerUserSchema, prov...
__InstanceProviderRegistration      {}                   {InteractionType, provider, QuerySupportLevels, SupportsBat...
__MethodProviderRegistration        {}                   {provider}
__PropertyProviderRegistration      {}                   {provider, SupportsGet, SupportsPut}
__EventConsumerProviderRegistration {}                   {ConsumerClassNames, provider}
__NAMESPACE                         {}                   {Name}
__EventFilter                       {}                   {CreatorSID, EventAccess, EventNamespace, Name...}
__EventConsumer                     {}                   {CreatorSID, MachineName, MaximumQueueSize}
__FilterToConsumerBinding           {}                   {Consumer, CreatorSID, DeliverSynchronously, DeliveryQoS...}
__AggregateEvent                    {}                   {NumberOfEvents, Representative}
__TimerNextFiring                   {}                   {NextEvent64BitTime, TimerId}
__Event                             {}                   {SECURITY_DESCRIPTOR, TIME_CREATED}
__ExtrinsicEvent                    {}                   {SECURITY_DESCRIPTOR, TIME_CREATED}
Win32_DeviceChangeEvent             {}                   {EventType, SECURITY_DESCRIPTOR, TIME_CREATED}
Win32_SystemConfigurationChangeE... {}                   {EventType, SECURITY_DESCRIPTOR, TIME_CREATED}

// 后面还有很多
```

## 显示 WMI 类的信息

从上面列出的任意一个 WMI 类，可以使用下面代码显示这个类里面的属性

```csharp
PS> Get-WmiObject -Class Win32_OperatingSystem


SystemDirectory : C:\WINDOWS\system32
Organization    :
BuildNumber     : 17763
RegisteredUser  : lindexi_gd@outlook.com
SerialNumber    : 00331-10000-00001-AA523
Version         : 10.0.17763
```

这里的输出只是简要的信息，没有包含所有的属性，如果想输出所有的属性，可以使用下面代码

```csharp
PS> Get-WmiObject -Class Win32_OperatingSystem | Get-Member -MemberType Property

Name                                      MemberType Definition
----                                      ---------- ----------
BootDevice                                Property   string BootDevice {get;set;}
BuildNumber                               Property   string BuildNumber {get;set;}
BuildType                                 Property   string BuildType {get;set;}
Caption                                   Property   string Caption {get;set;}
CodeSet                                   Property   string CodeSet {get;set;}
CountryCode                               Property   string CountryCode {get;set;}
CreationClassName                         Property   string CreationClassName {get;set;}
CSCreationClassName                       Property   string CSCreationClassName {get;set;}
CSDVersion                                Property   string CSDVersion {get;set;}
CSName                                    Property   string CSName {get;set;}
// 还有很多属性
```

总结一下，获取一个 WMI  类的简洁属性，可以通过这个格式

```csharp
 Get-WmiObject -Class 某个类
```

具体的类可以通过 `Get-WmiObject -List` 找到

获取某个类里面包含的所有属性，通过这个格式

```csharp
Get-WmiObject -Class 某个类  | Get-Member -MemberType Property
```

如果需要获取某个类的某一些属性的值，可以通过下面的代码

```csharp
Get-WmiObject -Class 某个类 | Format-Table -Property 属性1,属性2
```

如获取 Win32_OperatingSystem 的 TotalVirtualMemorySize 和 RegisteredUser 因为小伙伴的设备和我不相同，可以看到不一样的

```csharp
PS> Get-WmiObject -Class Win32_OperatingSystem | Format-Table -Property TotalVirtualMemorySize,RegisteredUser

TotalVirtualMemorySize RegisteredUser
---------------------- --------------
              36052888 lindexi_gd@outlook.com
```

如果有很多属性，可以通过列表的方法输出，将 `Format-Table` 修改为 `Format-List` 请看下面

```csharp
PS> Get-WmiObject -Class Win32_OperatingSystem -Namespace root/cimv2 -ComputerName . | Format-List TotalVirtualMemorySize,TotalVisibleMemorySize,FreePhysicalMemory,FreeVirtualMemory,FreeSpaceInPagingFiles

TotalVirtualMemorySize : 36052888
TotalVisibleMemorySize : 25042840
FreePhysicalMemory     : 8510920
FreeVirtualMemory      : 9954748
FreeSpaceInPagingFiles : 10482656
```

通过 WMI 可以拿很多属性

- [PowerShell 拿到显卡信息](https://blog.lindexi.com/post/PowerShell-%E6%8B%BF%E5%88%B0%E6%98%BE%E5%8D%A1%E4%BF%A1%E6%81%AF.html )

- [PowerShell 通过 WMI 获取设备厂商](https://blog.lindexi.com/post/PowerShell-%E9%80%9A%E8%BF%87-WMI-%E8%8E%B7%E5%8F%96%E8%AE%BE%E5%A4%87%E5%8E%82%E5%95%86.html )

- [PowerShell 通过 WMI 获取系统信息](https://blog.lindexi.com/post/PowerShell-%E9%80%9A%E8%BF%87-WMI-%E8%8E%B7%E5%8F%96%E7%B3%BB%E7%BB%9F%E4%BF%A1%E6%81%AF.html )

- [PowerShell 通过 WMI 获取系统安装的驱动](https://blog.lindexi.com/post/PowerShell-%E9%80%9A%E8%BF%87-WMI-%E8%8E%B7%E5%8F%96%E7%B3%BB%E7%BB%9F%E5%AE%89%E8%A3%85%E7%9A%84%E9%A9%B1%E5%8A%A8.html )

- [PowerShell 通过 WMI 获取系统服务](https://blog.lindexi.com/post/PowerShell-%E9%80%9A%E8%BF%87-WMI-%E8%8E%B7%E5%8F%96%E7%B3%BB%E7%BB%9F%E6%9C%8D%E5%8A%A1.html )

- [PowerShell 通过 WMI 获取补丁](https://blog.lindexi.com/post/PowerShell-%E9%80%9A%E8%BF%87-WMI-%E8%8E%B7%E5%8F%96%E8%A1%A5%E4%B8%81.html )

- [PowerShell 通过 WMI 获取系统安装软件](https://blog.lindexi.com/post/PowerShell-%E9%80%9A%E8%BF%87-WMI-%E8%8E%B7%E5%8F%96%E7%B3%BB%E7%BB%9F%E5%AE%89%E8%A3%85%E8%BD%AF%E4%BB%B6.html )

[Getting WMI Objects Get WmiObject](https://docs.microsoft.com/en-us/powershell/scripting/samples/getting-wmi-objects--get-wmiobject-?view=powershell-6 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
