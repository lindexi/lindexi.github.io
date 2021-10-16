# dotnet 5 让 WPF 调用 WindowsRuntime 方法

本文告诉大家在 dotnet 5 里，如何使用 WinRT 加上 Microsoft.Windows.SDK 的辅助来调用 WindowsRuntime 方法。当前是 2021.10 此时的 Windows App SDK 还没准备好，也因此构建起来等逻辑都有点锅。本文和大家演示如何在 WPF 应用里面用上 UWP 提供的 FolderPicker 类型

<!--more-->
<!-- CreateTime:2021/10/12 8:47:29 -->

<!-- 发布 -->

需要将原有的基于 .NET 5 的 WPF 应用的 csproj 替换 `net5.0-windows` 为 `net5.0-windows10.0.19041.0` 代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk.WindowsDesktop">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <UseWPF>true</UseWPF>
    <TargetFramework>net5.0-windows10.0.19041.0</TargetFramework>
  </PropertyGroup>

</Project>
```

以上表示采用 10.0.19041.0 的 SDK 版本，更多可用的版本请参阅 [CsWinRT/authoring.md at master · microsoft/CsWinRT](https://github.com/microsoft/CsWinRT/blob/master/docs/authoring.md )

当前有以下的 SDK 可用

- net5.0-windows10.0.17763.0
- net5.0-windows10.0.18362.0
- net5.0-windows10.0.19041.0
- net5.0-windows10.0.20348.0
- net6.0-windows10.0.17763.0
- net6.0-windows10.0.18362.0
- net6.0-windows10.0.19041.0
- net6.0-windows10.0.20348.0

我当前也理不清 CsWinRT 和 Windows App SDK 等仓库和技术线的关系，请不要问我为什么会碰到这些仓库

接下来需要在 cs 代码里面定义一个 COM 接口，通过此接口进行初始化。尽管官方文档说可以使用 `WinRT.Interop.InitializeWithWindow.Initialize` 方法进行初始化，然而实际上我没有找到此 InitializeWithWindow 类型。好在这些都是 COM 接口，可以自己定义，代码如下

```csharp
        [ComImport]
        [Guid("3E68D4BD-7135-4D10-8018-9FB6D9F33FA1")]
        [InterfaceType(ComInterfaceType.InterfaceIsIUnknown)]
        public interface IInitializeWithWindow
        {
            void Initialize(IntPtr hwnd);
        }
```

以上接口是通过 [Pin secondary tiles from desktop apps - Windows apps](https://docs.microsoft.com/en-us/windows/apps/design/shell/tiles-and-notifications/secondary-tiles-desktop-pinning?WT.mc_id=WD-MVP-5003260 ) 文档了解的

先创建 FolderPicker 对象

```csharp
            var folderPicker = new Windows.Storage.Pickers.FolderPicker();
            folderPicker.FileTypeFilter.Add("*");
```

接着将当前的窗口给到 FolderPicker 对象，让这个对象可以作为窗口的模态

```csharp
            var hwnd = System.Diagnostics.Process.GetCurrentProcess().MainWindowHandle; //WinRT.Interop.WindowNative.GetWindowHandle(this);

            //WinRT.Interop.InitializeWithWindow.Initialize(folderPicker, hwnd);
            //IInitializeWithCoreWindow initializeWithCoreWindow;// 这个不能使用
            var initializeWithCoreWindow = folderPicker.As<IInitializeWithWindow>();
            initializeWithCoreWindow.Initialize(hwnd);
```

接下来就可以和 UWP 一样使用 FolderPicker 让用户选择文件夹

```csharp
            var folder = await folderPicker.PickSingleFolderAsync();
            Debug.WriteLine(folder.Path);
```

使用本文的方法，应用是不需要进行打包为 MSIX 包在用户端进行部署就可以使用，双击 exe 就可以使用，和之前的相同。~~因为调用 WindowsRuntime 的原理是 COM 调用，不需要进行打包~~

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/26b7b6eec1f8e734bb9dbd49447f62fe2e116a9c/WelhearyalluneaceKujalwhekiraqi) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/26b7b6eec1f8e734bb9dbd49447f62fe2e116a9c/WelhearyalluneaceKujalwhekiraqi) 欢迎访问

可以通过如下方式获取本文代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 26b7b6eec1f8e734bb9dbd49447f62fe2e116a9c
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 WelhearyalluneaceKujalwhekiraqi 文件夹

大概可以从 [Breaking change: Built-in support for WinRT is removed from .NET](https://docs.microsoft.com/en-us/dotnet/core/compatibility/interop/5.0/built-in-support-for-winrt-removed ) 文档了解到为什么在 .NET 5 需要用到 [CsWinRT](https://github.com/microsoft/CsWinRT ) 的原因

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
