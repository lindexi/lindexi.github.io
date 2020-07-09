
# 推荐官方开源 PInvoke 库 包含大量 win32 封装

在调用 win32 库的时候，小伙伴会遇到的问题是不知道对应的 win32 函数应该如何写。或者在网上抄了的代码的实现都有些诡异，想要自己封装发现工作量太大。好消息是官方将 PInvoke 库在 dotnet 基金会完全开源，包含了大量的 Win32 库，如 gdi32.dll 和 kernel32.dll 和 user32.dll 等

<!--more-->


<!-- 发布 -->

使用官方的库的优势是什么呢？第一个就是减少从网上复制粘贴有趣的 PInvoke 调用实现，其次是质量上能保底。虽然官方的实现也不够完美，例如 User32 的 GetWindowLong 方法依然有坑。但是因为此项目是在 [github 开源](https://github.com/dotnet/pinvoke) 因此也会有大量的小伙伴入坑不断的修复，相对来说应该会比自己实现的好一些

现在官方已经将大量的 dll 进行了封装

已经实现的 dll 如下

Library      | Package name     | NuGet       | Description
-------------|------------------|-------------|-------------
advapi32.dll |`PInvoke.AdvApi32`| [![NuGet](https://buildstats.info/nuget/PInvoke.AdvApi32)](https://www.nuget.org/packages/PInvoke.AdvApi32)|Windows Advanced Services
bcrypt.dll   |`PInvoke.BCrypt`  | [![NuGet](https://buildstats.info/nuget/PInvoke.BCrypt)](https://www.nuget.org/packages/PInvoke.BCrypt)|[Windows Cryptography API: Next Generation][CNG]
cabinet.dll  |`PInvoke.Cabinet` | [![NuGet](https://buildstats.info/nuget/PInvoke.Cabinet)](https://www.nuget.org/packages/PInvoke.Cabinet)|[Cabinet API Functions][Cabinet]
cfgmgr32.dll |`PInvoke.CfgMgr32`| [![NuGet](https://buildstats.info/nuget/PInvoke.CfgMgr32)](https://www.nuget.org/packages/PInvoke.CfgMgr32)|[Device and Driver Installation][CfgMgr32]
crypt32.dll  |`PInvoke.Crypt32` | [![NuGet](https://buildstats.info/nuget/PInvoke.Crypt32)](https://www.nuget.org/packages/PInvoke.Crypt32)|[Windows Cryptography API][Crypt32]
DwmApi.dll   |`PInvoke.DwmApi`  | [![NuGet](https://buildstats.info/nuget/PInvoke.DwmApi)](https://www.nuget.org/packages/PInvoke.DwmApi)|[Desktop Window Manager][DwmApi]
fusion.dll   |`PInvoke.Fusion`  | [![NuGet](https://buildstats.info/nuget/PInvoke.Fusion)](https://www.nuget.org/packages/PInvoke.Fusion)|.NET Framework Fusion
gdi32.dll    |`PInvoke.Gdi32`   | [![NuGet](https://buildstats.info/nuget/PInvoke.Gdi32)](https://www.nuget.org/packages/PInvoke.Gdi32)|[Windows Graphics Device Interface][Gdi]
hid.dll      |`PInvoke.Hid`     | [![NuGet](https://buildstats.info/nuget/PInvoke.Hid)](https://www.nuget.org/packages/PInvoke.Hid)|[Windows Human Interface Devices][Hid]
iphlpapi.dll |`PInvoke.IPHlpApi`| [![NuGet](https://buildstats.info/nuget/PInvoke.IPHlpApi)](https://www.nuget.org/packages/PInvoke.IPHlpApi)|[IP Helper](IPHlpApi)
kernel32.dll |`PInvoke.Kernel32`| [![NuGet](https://buildstats.info/nuget/PInvoke.Kernel32)](https://www.nuget.org/packages/PInvoke.Kernel32)|Windows Kernel API
magnification.dll |`PInvoke.Magnification`| [![NuGet](https://buildstats.info/nuget/PInvoke.Magnification)](https://www.nuget.org/packages/PInvoke.Magnification)|[Windows Magnification API][Magnification]
mscoree.dll  |`PInvoke.MSCorEE` | [![NuGet](https://buildstats.info/nuget/PInvoke.MSCorEE)](https://www.nuget.org/packages/PInvoke.MSCorEE)|.NET Framework CLR host
msi.dll      |`PInvoke.Msi`     | [![NuGet](https://buildstats.info/nuget/PInvoke.Msi)](https://www.nuget.org/packages/PInvoke.Msi)|[Microsoft Installer][Msi]
ncrypt.dll   |`PInvoke.NCrypt`  | [![NuGet](https://buildstats.info/nuget/PInvoke.NCrypt)](https://www.nuget.org/packages/PInvoke.NCrypt)|[Windows Cryptography API: Next Generation][CNG]
netapi32.dll |`PInvoke.NetApi32`| [![NuGet](https://buildstats.info/nuget/PInvoke.NetApi32)](https://www.nuget.org/packages/PInvoke.NetApi32)|[Network Management][NetApi32]
newdev.dll   |`PInvoke.NewDev`  | [![NuGet](https://buildstats.info/nuget/PInvoke.NewDev)](https://www.nuget.org/packages/PInvoke.NewDev)|[Device and Driver Installation][NewDev]
ntdll.dll    |`PInvoke.NTDll`   | [![NuGet](https://buildstats.info/nuget/PInvoke.NTDll)](https://www.nuget.org/packages/PInvoke.NTDll)|Windows NTDll
psapi.dll    |`PInvoke.Psapi`   | [![NuGet](https://buildstats.info/nuget/PInvoke.Psapi)](https://www.nuget.org/packages/PInvoke.Psapi)|[Windows Process Status API][Psapi]
setupapi.dll |`PInvoke.SetupApi`| [![NuGet](https://buildstats.info/nuget/PInvoke.SetupApi)](https://www.nuget.org/packages/PInvoke.SetupApi)|[Windows setup API][SetupApi]
SHCore.dll   |`PInvoke.SHCore`  | [![NuGet](https://buildstats.info/nuget/PInvoke.SHCore)](https://www.nuget.org/packages/PInvoke.SHCore)|[Windows Shell][Shell32]
shell32.dll  |`PInvoke.Shell32` | [![NuGet](https://buildstats.info/nuget/PInvoke.Shell32)](https://www.nuget.org/packages/PInvoke.Shell32)|[Windows Shell][Shell32]
user32.dll   |`PInvoke.User32`  | [![NuGet](https://buildstats.info/nuget/PInvoke.User32)](https://www.nuget.org/packages/PInvoke.User32)|Windows User Interface
userenv.dll  |`PInvoke.Userenv` | [![NuGet](https://buildstats.info/nuget/PInvoke.Userenv)](https://www.nuget.org/packages/PInvoke.Userenv)|Windows User Environment
uxtheme.dll  |`PInvoke.UxTheme` | [![NuGet](https://buildstats.info/nuget/PInvoke.UxTheme)](https://www.nuget.org/packages/PInvoke.UxTheme)|[Windows Visual Styles][UxTheme]
winusb.dll   |`PInvoke.WinUsb`  | [![NuGet](https://buildstats.info/nuget/PInvoke.WinUsb)](https://www.nuget.org/packages/PInvoke.WinUsb)|[USB Driver][WinUsb]
WtsApi32.dll |`PInvoke.WtsApi32`| [![NuGet](https://buildstats.info/nuget/PInvoke.WtsApi32)](https://www.nuget.org/packages/PInvoke.WtsApi32)|[Windows Remote Desktop Services][WtsApi32]


那如何使用这个库？在 dotnet 里面使用库都是统一使用 NuGet 的方法，在 NuGet 里面按照自己的需要安装对应的库就可以了

如我想要调用 Kernel32 的 CreateProcess 方法，这个方法里面包含了很多结构体等的实现，如果要我自己去找这些结构体的实现，那么我也许会复制到坑代码。而在使用库的时候，我可以在 csproj 添加下面代码安装 NuGet 库

```csharp
    <ItemGroup>
        <PackageReference Include="PInvoke.Kernel32" Version="0.6.49" />
    </ItemGroup>
```

此时我就可以通过 Kernel32 类拿到对应的函数和结构体，请看代码

```csharp
using PInvoke;

            var startupInfo = new Kernel32.STARTUPINFO()
            {
                dwX = 300, // X
                dwY = 300, // Y
                dwXSize = 1000, // 宽度
                dwYSize = 300,  // 高度
                dwFlags = Kernel32.StartupInfoFlags.STARTF_USESHOWWINDOW,
            };
            
            Kernel32.PROCESS_INFORMATION processInformation;
            var creationFlag = Kernel32.CreateProcessFlags.NORMAL_PRIORITY_CLASS | Kernel32.CreateProcessFlags.CREATE_UNICODE_ENVIRONMENT;

            var processSecurityAttribute = Kernel32.SECURITY_ATTRIBUTES.Create();
            var threadAttribute = Kernel32.SECURITY_ATTRIBUTES.Create();

            Kernel32.CreateProcess
            (
                lpApplicationName: @"C:\windows\notepad.exe",
                lpCommandLine: " ",
                lpProcessAttributes: processSecurityAttribute,
                lpThreadAttributes: threadAttribute, 
                bInheritHandles: false,
                dwCreationFlags: creationFlag,
                lpEnvironment: IntPtr.Zero,
                lpCurrentDirectory: null,
                lpStartupInfo: ref startupInfo,
                lpProcessInformation: out processInformation
            );

            Console.WriteLine(Kernel32.GetLastError());

            var process = Process.GetProcessById(processInformation.dwProcessId);
            Console.WriteLine(process.Id);
```


[CfgMgr32]: https://docs.microsoft.com/en-us/windows/win32/api/cfgmgr32/
[CNG]: https://msdn.microsoft.com/en-us/library/windows/desktop/aa376210
[Crypt32]: https://msdn.microsoft.com/en-us/library/windows/desktop/aa380256
[DwmApi]: https://msdn.microsoft.com/en-us/library/windows/desktop/aa969540.aspx
[Hid]: https://msdn.microsoft.com/en-us/library/windows/hardware/ff538865
[IPHlpApi]: https://docs.microsoft.com/en-us/windows/win32/api/_iphlp/
[Magnification]: https://msdn.microsoft.com/en-us/library/windows/desktop/ms692162
[Msi]: https://msdn.microsoft.com/en-us/library/aa372860.aspx
[SetupApi]: https://msdn.microsoft.com/en-us/library/windows/hardware/ff550855
[Gdi]: https://msdn.microsoft.com/en-us/library/dd145203
[Psapi]: https://msdn.microsoft.com/en-us/library/windows/desktop/ms684884.aspx
[UxTheme]: https://msdn.microsoft.com/en-us/library/windows/desktop/bb773187.aspx
[NetApi32]: https://msdn.microsoft.com/en-us/library/windows/desktop/aa370680.aspx
[NewDev]: https://docs.microsoft.com/en-us/windows/win32/api/newdev/
[Shell32]: https://msdn.microsoft.com/en-us/library/windows/desktop/bb773177.aspx
[WinUsb]: https://docs.microsoft.com/en-us/windows/win32/api/winusb/
[WtsApi32]: https://msdn.microsoft.com/en-us/library/aa383468(v=vs.85).aspx
[Cabinet]: https://docs.microsoft.com/en-us/windows/win32/devnotes/cabinet-api-functions

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/19cdd72409ba1af6bb3792ce02118055a0948a15/HalwerewolokaichaKojerwhabal) 欢迎小伙伴访问





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。