# WPF 判断当前应用被打包为 UWP 而运行

本文告诉大家如何在应用运行过程判断自己的 WPF 应用被转制为 UWP 应用运行

<!--more-->

通过 kernel32 的 GetCurrentPackageFullName 方法即可判断，此方法要求是在 Win10 或以上版本才能使用哦。当然了，如果在 Win10 以下的版本，如 Win7 那默认就跑不了 UWP 应用，也就不需要判断了

判断代码如下

```csharp
        const long APPMODEL_ERROR_NO_PACKAGE = 15700L;

        [DllImport("kernel32.dll", CharSet = CharSet.Unicode, SetLastError = true)]
        static extern int GetCurrentPackageFullName(ref int packageFullNameLength, StringBuilder packageFullName);

        public bool IsRunningAsUwp()
        {
                int length = 0;
                StringBuilder sb = new StringBuilder(0);
                int result = GetCurrentPackageFullName(ref length, sb);

                sb = new StringBuilder(length);
                result = GetCurrentPackageFullName(ref length, sb);

                return result != APPMODEL_ERROR_NO_PACKAGE;
        }
```

在开始判断之前，先判断系统版本，代码如下

```csharp
        public bool IsRunningAsUwp()
        {
            if (IsWindows7OrLower)
            {
                return false;
            }
            else
            {
                int length = 0;
                StringBuilder sb = new StringBuilder(0);
                int result = GetCurrentPackageFullName(ref length, sb);

                sb = new StringBuilder(length);
                result = GetCurrentPackageFullName(ref length, sb);

                return result != APPMODEL_ERROR_NO_PACKAGE;
            }
        }

        private bool IsWindows7OrLower
        {
            get
            {
                int versionMajor = Environment.OSVersion.Version.Major;
                int versionMinor = Environment.OSVersion.Version.Minor;
                double version = versionMajor + (double)versionMinor / 10;
                return version <= 6.1;
            }
        }
```

以上代码由微软提供，请看 [microsoft/Windows-AppConsult-Tools-DesktopBridgeHelpers: Simple libraryto detect if a desktop application is running as classic Win32 or packaged with the Desktop Bridge](https://github.com/microsoft/Windows-AppConsult-Tools-DesktopBridgeHelpers )

[Call Windows Runtime APIs in desktop apps - Windows apps](https://docs.microsoft.com/en-us/windows/apps/desktop/modernize/desktop-to-uwp-enhance#net-5-and-later-use-the-target-framework-moniker-option?WT.mc_id=WD-MVP-5003260 )

[Desktop Bridge – Identify the application’s context](https://docs.microsoft.com/en-us/archive/blogs/appconsult/desktop-bridge-identify-the-applications-context )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
