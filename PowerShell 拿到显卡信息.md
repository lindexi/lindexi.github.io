# PowerShell 拿到显卡信息

本文告诉大家如何在 PowerShell 通过 WMI 拿到显卡信息

<!--more-->
<!-- CreateTime:2019/7/29 10:03:35 -->


在 PowerShell 可以使用下面代码拿到显卡的信息

```csharp
Get-WmiObject Win32_VideoController
```

打开 PowerShell 输入代码可以看到下面内容，当然因为小伙伴的显卡和我不相同，拿到的字符串是不相同

```csharp
__GENUS                      : 2
__CLASS                      : Win32_VideoController
__SUPERCLASS                 : CIM_PCVideoController
__DYNASTY                    : CIM_ManagedSystemElement
__RELPATH                    : Win32_VideoController.DeviceID="VideoController1"
__PROPERTY_COUNT             : 59
__DERIVATION                 : {CIM_PCVideoController, CIM_VideoController, CIM_Controller, CIM_LogicalDevice...}
__SERVER                     : lindexi
__NAMESPACE                  : root\cimv2
__PATH                       : \\lindexi\root\cimv2:Win32_VideoController.DeviceID="VideoController1"
AcceleratorCapabilities      :
AdapterCompatibility         : Intel Corporation
AdapterDACType               : Internal
AdapterRAM                   : 1073741824
Availability                 : 3
CapabilityDescriptions       :
Caption                      : Intel(R) HD Graphics 530
ColorTableEntries            :
ConfigManagerErrorCode       : 0
ConfigManagerUserConfig      : False
CreationClassName            : Win32_VideoController
CurrentBitsPerPixel          : 32
CurrentHorizontalResolution  : 1920
CurrentNumberOfColors        : 4294967296
CurrentNumberOfColumns       : 0
CurrentNumberOfRows          : 0
CurrentRefreshRate           : 60
CurrentScanMode              : 4
CurrentVerticalResolution    : 1080
Description                  : Intel(R) HD Graphics 530
DeviceID                     : VideoController1
DeviceSpecificPens           :
DitherType                   : 0
DriverDate                   : 20181119000000.000000-000
DriverVersion                : 23.20.16.4973
ErrorCleared                 :
ErrorDescription             :
ICMIntent                    :
ICMMethod                    :
InfFilename                  : oem16.inf
InfSection                   : iSKLD_w10_DS
InstallDate                  :
InstalledDisplayDrivers      : C:\WINDOWS\System32\DriverStore\FileRepository\igdlh64.inf_amd64_2c92d70c30b8effe\igdumd
                               im64.dll,C:\WINDOWS\System32\DriverStore\FileRepository\igdlh64.inf_amd64_2c92d70c30b8ef
                               fe\igd10iumd64.dll,C:\WINDOWS\System32\DriverStore\FileRepository\igdlh64.inf_amd64_2c92
                               d70c30b8effe\igd10iumd64.dll,C:\WINDOWS\System32\DriverStore\FileRepository\igdlh64.inf_
                               amd64_2c92d70c30b8effe\igd12umd64.dll
LastErrorCode                :
MaxMemorySupported           :
MaxNumberControlled          :
MaxRefreshRate               : 75
MinRefreshRate               : 59
Monochrome                   : False
Name                         : Intel(R) HD Graphics 530
NumberOfColorPlanes          :
NumberOfVideoPages           :
PNPDeviceID                  : PCI\VEN_8086&DEV_1912&SUBSYS_D0001458&REV_06\3&11583659&0&10
PowerManagementCapabilities  :
PowerManagementSupported     :
ProtocolSupported            :
ReservedSystemPaletteEntries :
SpecificationVersion         :
Status                       : OK
StatusInfo                   :
SystemCreationClassName      : Win32_ComputerSystem
SystemName                   : lindexi
SystemPaletteEntries         :
TimeOfLastReset              :
VideoArchitecture            : 5
VideoMemoryType              : 2
VideoMode                    :
VideoModeDescription         : 1920 x 1080 x 4294967296 种颜色
VideoProcessor               : Intel(R) HD Graphics Family
PSComputerName               : lindexi
```

如果只需要一些必要的数据，通过 Format-List 拿到特殊的属性

```csharp
Get-WmiObject Win32_VideoController | Format-List AcceleratorCapabilities,AdapterCompatibility,AdapterDACType,AdapterRAM,Availability,CapabilityDescriptions,Caption,DeviceID,DriverDate,DriverVersion,InstallDate,InstalledDisplayDrivers
```

运行上面代码会显示比上面少的信息

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

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
