---
title: dotnet 控制台调用 Windows.Graphics 实现屏幕截图功能
description: 本文将告诉大家如何在 .NET 控制台应用中，借助 Windows.Graphics.Capture 系列 WinRT API 实现高性能屏幕截图能力，最终将截图保存为 PNG 格式到本地临时目录
tags: dotnet
category: 
---

<!-- CreateTime:2026/04/24 07:29:48 -->

<!-- 发布 -->
<!-- 博客 -->

本文内容由 AI 辅助编写

## 前置准备

要使用 Windows.Graphics 系列 API 进行截图，我们需要先引入相关依赖：
- Microsoft.WindowsAppSDK：用于调用 WinRT 系列 API
- Vortice.Direct3D11 和 Vortice.Win32：用于操作 Direct3D11 设备，对接 Windows.Graphics API 的 Direct3D 依赖

对应的项目文件配置如下：

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0-windows10.0.26100</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Vortice.Direct3D11" Version="3.8.2" />
    <PackageReference Include="Vortice.Win32" Version="2.3.0" />
    <PackageReference Include="Microsoft.WindowsAppSDK" Version="1.8.260416003" />
  </ItemGroup>
</Project>
```

这里的 `<TargetFramework>net10.0-windows10.0.26100</TargetFramework>` 是为了适配 Windows App SDK 的版本要求，如果你不想加上 `windows10.0.26100` 后缀，可以参考 [WPF 不安装 WindowsAppSDK 使用 WinRT 功能的方法](https://blog.lindexi.com/post/WPF-%E4%B8%8D%E5%AE%89%E8%A3%85-WindowsAppSDK-%E4%BD%BF%E7%94%A8-WinRT-%E5%8A%9F%E8%83%BD%E7%9A%84%E6%96%B9%E6%B3%95.html ) 这篇博客实现兼容。

## 实现步骤

### 1. 创建 D3D11 设备

Windows.Graphics.Capture 的帧池需要绑定 Direct3D 设备才能工作，我们首先需要创建一个支持 BGRA 像素格式的 D3D11 硬件设备：

```csharp
var result = D3D11.D3D11CreateDevice(null, DriverType.Hardware, DeviceCreationFlags.BgraSupport, null, out ID3D11Device? device);
result.CheckError();

using var disposeDevice = device;
using var dxgiDevice = device!.QueryInterface<IDXGIDevice>();
```

这里没有传入显示适配器和特性等级，在本文这里，可以忽略这些参数，传入 null 即可

### 2. 转换为 WinRT 标准的 IDirect3DDevice

这里我踩了一个坑：一开始尝试使用 Vortice 提供的 `D3D11.CreateDirect3D11DeviceFromDXGIDevice` 方法转换，但是这个方法内部使用 `Marshal.GetObjectForIUnknown` 进行对象转换，不符合 WinRT 对象的转换规则，会导致后续调用抛出异常。

正确的做法是自己 P/Invoke 调用 d3d11.dll 提供的 `CreateDirect3D11DeviceFromDXGIDevice` 方法，再通过 WinRT 官方提供的转换方法转换：

```csharp
[DllImport
(
    "d3d11.dll",
    EntryPoint = "CreateDirect3D11DeviceFromDXGIDevice",
    SetLastError = true,
    CharSet = CharSet.Unicode
)]
static extern UInt32 CreateDirect3D11DeviceFromDXGIDevice(IntPtr dxgiDevice, out IntPtr graphicsDevice);
```

调用转换的代码如下：

```csharp
CreateDirect3D11DeviceFromDXGIDevice(dxgiDevice.NativePointer, out var graphicsDevice);
IDirect3DDevice? direct3DDevice = WinRT.MarshalInterface<IDirect3DDevice>.FromAbi(graphicsDevice);
```

### 3. 创建帧池和捕获会话

我们使用自由线程的帧池，不需要绑定UI线程，非常适合控制台应用场景：

```csharp
using Direct3D11CaptureFramePool direct3D11CaptureFramePool = Direct3D11CaptureFramePool.CreateFreeThreaded(
    direct3DDevice,
    Windows.Graphics.DirectX.DirectXPixelFormat.B8G8R8A8UIntNormalized,
    1, // 仅需要单张截图，缓存1帧足够
    new Windows.Graphics.SizeInt32 { Width = 1920, Height = 1080 }); // 这里可以自行替换为实际屏幕分辨率
```

接下来创建捕获项和捕获会话，示例中我们捕获第一个显示器的内容：

```csharp
// 如果有多个屏幕，可以枚举显示设备获取对应的DisplayId替换这里的参数
var captureItem = GraphicsCaptureItem.TryCreateFromDisplayId(new DisplayId(0));
using GraphicsCaptureSession graphicsCaptureSession = direct3D11CaptureFramePool.CreateCaptureSession(captureItem);
graphicsCaptureSession.StartCapture();
```

这里有两个可选操作：

- 可以提前调用 `WinRT.ComWrappersSupport.InitializeComWrappers()` 初始化 WinRT 封装，不调用也能正常运行，多调用也没有副作用
- 可以提前调用 `GraphicsCaptureAccess.RequestAccessAsync(GraphicsCaptureAccessKind.Programmatic)` 申请截图权限，不调用也能直接创建会话，多调用也没有问题

如果只需要单张截图，不需要监听 `FrameArrived` 事件，直接循环获取帧即可：

```csharp
while (true)
{
    var direct3D11CaptureFrame = direct3D11CaptureFramePool.TryGetNextFrame();
    if (direct3D11CaptureFrame is not null)
    {
        using (direct3D11CaptureFrame)
        {
            return await SaveFrameToFileAsync(direct3D11CaptureFrame);
        }
    }
    await Task.Delay(TimeSpan.FromSeconds(0.1));
}
```

如果你需要做持续录屏，才需要监听 `FrameArrived` 事件，这时候要注意将 `Direct3D11CaptureFramePool` 和 `GraphicsCaptureSession` 放在类的字段中，防止被GC回收导致事件不触发。

### 4. 保存截图到本地

拿到捕获的帧之后，我们将其转换为 SoftwareBitmap，再编码为 PNG 格式保存到临时目录：

```csharp
private static async Task<FileInfo> SaveFrameToFileAsync(Direct3D11CaptureFrame direct3D11CaptureFrame)
{
    ArgumentNullException.ThrowIfNull(direct3D11CaptureFrame);

    var fileName = $"{Path.GetFileNameWithoutExtension(Path.GetRandomFileName())}.png";
    var tempFolderPath = Path.GetTempPath();
    StorageFolder tempFolder = await StorageFolder.GetFolderFromPathAsync(tempFolderPath);
    StorageFile storageFile = await tempFolder.CreateFileAsync(fileName, CreationCollisionOption.GenerateUniqueName);

    using SoftwareBitmap softwareBitmap = await SoftwareBitmap.CreateCopyFromSurfaceAsync(
        direct3D11CaptureFrame.Surface,
        BitmapAlphaMode.Premultiplied);
    using SoftwareBitmap bitmapToSave = SoftwareBitmap.Convert(
        softwareBitmap,
        BitmapPixelFormat.Bgra8,
        BitmapAlphaMode.Premultiplied);
    using IRandomAccessStream stream = await storageFile.OpenAsync(FileAccessMode.ReadWrite);

    BitmapEncoder encoder = await BitmapEncoder.CreateAsync(BitmapEncoder.PngEncoderId, stream);
    encoder.SetSoftwareBitmap(bitmapToSave);
    await encoder.FlushAsync();
    
    return new FileInfo(storageFile.Path);
}
```

## 完整代码

以下是完整的可运行的截图实现代码：

```csharp
using System.Runtime.InteropServices;

using Vortice.Direct3D;
using Vortice.Direct3D11;
using Vortice.DXGI;

using Windows.Graphics;
using Windows.Graphics.Capture;
using Windows.Graphics.DirectX.Direct3D11;
using Windows.Graphics.Imaging;
using Windows.Storage;
using Windows.Storage.Streams;

namespace FellojeremgeraneJerlowewaju;

class ScreenSnapshotProvider
{
    public async Task<FileInfo> TakeSnapshotAsync()
    {
        // 可以不调用 InitializeComWrappers 方法。多调用也没啥坏处
        //global::WinRT.ComWrappersSupport.InitializeComWrappers();

        // 这里的权限请求也可以不调用，直接创建捕获会话。多调用也没啥坏处
        //if (GraphicsCaptureSession.IsSupported())
        //{
        //    var status = await GraphicsCaptureAccess.RequestAccessAsync(GraphicsCaptureAccessKind.Programmatic);
        //}

        var result = D3D11.D3D11CreateDevice(null, DriverType.Hardware, DeviceCreationFlags.BgraSupport, null, out ID3D11Device? device);
        result.CheckError();

        using var disposeDevice = device;

        using var dxgiDevice = device!.QueryInterface<IDXGIDevice>();

        // 不能这样直接转换，应该在 WinRT 里面应该调用 FromAbi 才能转换
        // 在这个方法里面用的是 Marshal.GetObjectForIUnknown 转换，这是不正确的
        //IDirect3DDevice direct3DDevice = Vortice.Direct3D11.D3D11.CreateDirect3D11DeviceFromDXGIDevice<IDirect3DDevice>(dxgiDevice);

        CreateDirect3D11DeviceFromDXGIDevice(dxgiDevice.NativePointer, out var graphicsDevice);
        IDirect3DDevice? direct3DDevice = WinRT.MarshalInterface<IDirect3DDevice>.FromAbi(graphicsDevice);

        // 这里的大小参数也是固定的
        using Direct3D11CaptureFramePool direct3D11CaptureFramePool = Direct3D11CaptureFramePool.CreateFreeThreaded(
            direct3DDevice,
            Windows.Graphics.DirectX.DirectXPixelFormat.B8G8R8A8UIntNormalized,
            1,
            // 正常来说，都是采用 GraphicsCaptureItem 的尺寸来决定的。如果给一个很小的尺寸，则会裁剪屏幕内容
            new Windows.Graphics.SizeInt32 { Width = 1920, Height = 1080 });

        // 这里的需求只是取一个截图界面，那就不需要监听事件。如果要监听事件，需要小心 Direct3D11CaptureFramePool 被释放，建议需要放在字段
        //direct3D11CaptureFramePool.FrameArrived += (sender, args) =>
        //{
        //    var direct3D11CaptureFrame = direct3D11CaptureFramePool.TryGetNextFrame();
        //};

        // 如果有多个屏幕，这里需要优化
        var captureItem = GraphicsCaptureItem.TryCreateFromDisplayId(new DisplayId(0));
        using GraphicsCaptureSession graphicsCaptureSession = direct3D11CaptureFramePool.CreateCaptureSession(captureItem);
        graphicsCaptureSession.StartCapture();

        while (true)
        {
            var direct3D11CaptureFrame = direct3D11CaptureFramePool.TryGetNextFrame();
            if (direct3D11CaptureFrame is not null)
            {
                using (direct3D11CaptureFrame)
                {
                    return await SaveFrameToFileAsync(direct3D11CaptureFrame);
                }
            }

            await Task.Delay(TimeSpan.FromSeconds(0.1));
        }
    }

    private static async Task<FileInfo> SaveFrameToFileAsync(Direct3D11CaptureFrame direct3D11CaptureFrame)
    {
        ArgumentNullException.ThrowIfNull(direct3D11CaptureFrame);

        var fileName = $"{Path.GetFileNameWithoutExtension(Path.GetRandomFileName())}.png";
        var tempFolderPath = Path.GetTempPath();
        StorageFolder tempFolder = await StorageFolder.GetFolderFromPathAsync(tempFolderPath);
        StorageFile storageFile = await tempFolder.CreateFileAsync(fileName, CreationCollisionOption.GenerateUniqueName);

        using SoftwareBitmap softwareBitmap = await SoftwareBitmap.CreateCopyFromSurfaceAsync(
            direct3D11CaptureFrame.Surface,
            BitmapAlphaMode.Premultiplied);
        using SoftwareBitmap bitmapToSave = SoftwareBitmap.Convert(
            softwareBitmap,
            BitmapPixelFormat.Bgra8,
            BitmapAlphaMode.Premultiplied);
        using IRandomAccessStream stream = await storageFile.OpenAsync(FileAccessMode.ReadWrite);

        BitmapEncoder encoder = await BitmapEncoder.CreateAsync(BitmapEncoder.PngEncoderId, stream);
        encoder.SetSoftwareBitmap(bitmapToSave);
        await encoder.FlushAsync();
        
        return new FileInfo(storageFile.Path);
    }

    /// <summary>
    /// 把自己创建的原生Win32 `IDXGIDevice`（从D3D11设备查询得到），封装转换成WinRT标准的 `Windows.Graphics.DirectX.Direct3D11.IDirect3DDevice` 对象
    /// </summary>
    /// <param name="dxgiDevice"></param>
    /// <param name="graphicsDevice"></param>
    /// <returns></returns>
    [DllImport
    (
        "d3d11.dll",
        EntryPoint = "CreateDirect3D11DeviceFromDXGIDevice",
        SetLastError = true,
        CharSet = CharSet.Unicode
    )]
    static extern UInt32 CreateDirect3D11DeviceFromDXGIDevice(IntPtr dxgiDevice, out IntPtr graphicsDevice);
}
```

## 使用示例

在 Main 方法中直接调用即可：

```csharp
internal class Program
{
    static async Task Main(string[] args)
    {
        var provider = new ScreenSnapshotProvider();
        var screenshotFile = await provider.TakeSnapshotAsync();
        Console.WriteLine($"截图已保存到：{screenshotFile.FullName}");
    }
}
```

## 注意事项

1. 该功能仅支持 Windows 10 1803 及以上版本的系统，低于该版本的系统无法使用 Windows.Graphics.Capture 系列 API
2. 如果需要捕获指定窗口而不是整个屏幕，只需要将 `GraphicsCaptureItem.TryCreateFromDisplayId` 替换为 `GraphicsCaptureItem.TryCreateFromWindowId` 传入对应窗口的 WindowId 即可
3. 示例中的分辨率是写死的1920*1080，实际使用时可以枚举显示设备获取对应屏幕的实际分辨率传入帧池创建参数

## 代码

本文以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/57b2035dea5fa9bcb0f12b06d23efcaff05ddb4a/DirectX/WindowsGraphics/FellojeremgeraneJerlowewaju) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/57b2035dea5fa9bcb0f12b06d23efcaff05ddb4a/DirectX/WindowsGraphics/FellojeremgeraneJerlowewaju) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 57b2035dea5fa9bcb0f12b06d23efcaff05ddb4a
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 57b2035dea5fa9bcb0f12b06d23efcaff05ddb4a
```

获取代码之后，进入 DirectX/WindowsGraphics/FellojeremgeraneJerlowewaju 文件夹，即可获取到源代码

## 参考博客

- [C# 调用WGC 实现桌面屏幕的捕获 - wuty007 - 博客园](https://www.cnblogs.com/wuty/p/19578990 )
- [.NET 窗口/屏幕录制 - 唐宋元明清2188 - 博客园](https://www.cnblogs.com/kybs0/p/18330811 )
- [屏幕捕获到视频 - UWP applications - Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/uwp/audio-video-camera/screen-capture-video )

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
