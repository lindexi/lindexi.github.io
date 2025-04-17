
# WPF 模拟 WPFMediaKit 的 D3D 配置用来测试4k性能

本文告诉大家我在测试 WPFMediaKit 的 D3D 配置性能影响在 4k 分辨率设备下采用高清摄像头的性能

<!--more-->


<!-- CreateTime:2021/12/20 19:21:53 -->
<!-- 标签：WPF,D2D,DirectX,SharpDX,渲染 -->

<!-- 发布 -->

测试效果是 10 代 i3 带 4G 内存和集显 UHD 630 在 4k 下，跑满 36 Hz 不卡。以下是具体测试的逻辑

在 WPFMediaKit 定义渲染在 Vmr9Allocator 类里面，在 CreateDevice 方法上采用如下代码进行初始化 IDirect3DDevice9 设备

```csharp
        private void CreateDevice()
        {
            if (m_device != null)
                return;

            var param = new D3DPRESENT_PARAMETERS
            {
                Windowed = 1,
                Flags = ((short)D3DPRESENTFLAG.D3DPRESENTFLAG_VIDEO),
                BackBufferFormat = D3DFORMAT.D3DFMT_X8R8G8B8,
                SwapEffect = D3DSWAPEFFECT.D3DSWAPEFFECT_COPY
            };

            /* The COM pointer to our D3D Device */
            IntPtr dev;

            /* Windows Vista runs much more performant with the IDirect3DDevice9Ex */
            if (IsVistaOrBetter)
            {
                m_d3dEx.CreateDeviceEx(0, D3DDEVTYPE.D3DDEVTYPE_HAL, m_hWnd,
                  CreateFlags.D3DCREATE_HARDWARE_VERTEXPROCESSING | CreateFlags.D3DCREATE_MULTITHREADED,
                  ref param, IntPtr.Zero, out dev);
            }
            else/* Windows XP */
            {
                m_d3d.CreateDevice(0, D3DDEVTYPE.D3DDEVTYPE_HAL, m_hWnd,
                  CreateFlags.D3DCREATE_SOFTWARE_VERTEXPROCESSING | CreateFlags.D3DCREATE_MULTITHREADED,
                  ref param, out dev);
            }

            m_device = (IDirect3DDevice9)Marshal.GetObjectForIUnknown(dev);
            Marshal.Release(dev);
        }
```

在 InitializeDevice 使用如下代码初始化

```csharp
                            hr = m_device.CreateTexture(lpAllocInfo.dwWidth, 
                                                        lpAllocInfo.dwHeight, 
                                                        1, 
                                                        1,
                                                        D3DFORMAT.D3DFMT_X8R8G8B8, 
                                                        0, 
                                                        out m_privateTexture, 
                                                        IntPtr.Zero);

                            DsError.ThrowExceptionForHR(hr);

                            hr = m_privateTexture.GetSurfaceLevel(0, out m_privateSurface);
                            DsError.ThrowExceptionForHR(hr);
```

通过如上代码创建的 IDirect3DSurface9 类型的 `m_privateSurface` 可以作为 D3DImage 的使用参数

为了测试此方式的参数创建的 IDirect3DTexture9 在 WPF 里的性能，本文将扔掉摄像头部分，换 D2D 渲染，测试在 4k 的性能。因为加上摄像头还有解码部分的逻辑，这部分逻辑将让说明性能失败

创建一个空 WPF 应用，在 MainWindow_Loaded 添加初始化代码

使用 Direct3DCreate9Ex 函数创建 IDirect3D9Ex 对象

```csharp
            var hr = Direct3DCreate9Ex(D3D_SDK_VERSION, out var direct3D9Ex);
```

所使用的参数如下

```csharp
        [ComImport, SuppressUnmanagedCodeSecurity,
        Guid("81BDCBCA-64D4-426d-AE8D-AD0147F4275C"),
        InterfaceType(ComInterfaceType.InterfaceIsIUnknown), SuppressUnmanagedCodeSecurity]
        public interface IDirect3D9
        {
        	// 忽略代码
        }

        [ComImport, SuppressUnmanagedCodeSecurity,
        Guid("02177241-69FC-400C-8FF1-93A44DF6861D"),
        InterfaceType(ComInterfaceType.InterfaceIsIUnknown), SuppressUnmanagedCodeSecurity]
        public interface IDirect3D9Ex : IDirect3D9
        {
        	// 忽略代码
        }

        /// <summary>
        /// The SDK version of D3D we are using
        /// </summary>
        private const ushort D3D_SDK_VERSION = 32;

        [DllImport("d3d9.dll", EntryPoint = "Direct3DCreate9Ex", CallingConvention = CallingConvention.StdCall),
        SuppressUnmanagedCodeSecurity]
        public static extern int Direct3DCreate9Ex(ushort SDKVersion, [Out] out IDirect3D9Ex ex);
```

拷贝 Vmr9Allocator 的代码初始化，但是需要修改部分逻辑，如删掉 BackBufferFormat 的设置和更改 SwapEffect 的参数，加上 hDeviceWindow 和 PresentationInterval 定义

```csharp
            m_hWnd = GetDesktopWindow();

            var param = new D3DPRESENT_PARAMETERS
            {
                Windowed = 1,
                Flags = ((short) D3DPRESENTFLAG.D3DPRESENTFLAG_VIDEO),
                /*
                D3DFMT_R8G8B8：表示一个24位像素，从左开始，8位分配给红色，8位分配给绿色，8位分配给蓝色。

                D3DFMT_X8R8G8B8：表示一个32位像素，从左开始，8位不用，8位分配给红色，8位分配给绿色，8位分配给蓝色。

                D3DFMT_A8R8G8B8：表示一个32位像素，从左开始，8位为ALPHA通道，8位分配给红色，8位分配给绿色，8位分配给蓝色。

                D3DFMT_A16B16G16R16F：表示一个64位浮点像素，从左开始，16位为ALPHA通道，16位分配给蓝色，16位分配给绿色，16位分配给红色。

                D3DFMT_A32B32G32R32F：表示一个128位浮点像素，从左开始，32位为ALPHA通道，32位分配给蓝色，32位分配给绿色，32位分配给红色。
                 */
                //BackBufferFormat = D3DFORMAT.D3DFMT_X8R8G8B8,

                //SwapEffect = D3DSWAPEFFECT.D3DSWAPEFFECT_COPY
                SwapEffect = D3DSWAPEFFECT.D3DSWAPEFFECT_DISCARD,

                hDeviceWindow = GetDesktopWindow(), // 添加
                PresentationInterval = (int) D3D9.PresentInterval.Default,
            };
```

和 Vmr9Allocator 使用相同代码创建设备

```csharp
            /* The COM pointer to our D3D Device */
            IntPtr dev;
            m_d3dEx.CreateDeviceEx(0, D3DDEVTYPE.D3DDEVTYPE_HAL, m_hWnd,
                Direct3D.CreateFlags.D3DCREATE_HARDWARE_VERTEXPROCESSING | Direct3D.CreateFlags.D3DCREATE_MULTITHREADED
                          | Direct3D.CreateFlags.D3DCREATE_FPU_PRESERVE,
                ref param, IntPtr.Zero, out dev);

            m_device = (IDirect3DDevice9) Marshal.GetObjectForIUnknown(dev);
            // 只是减少引用计数而已，现在换成 m_device 了
            Marshal.Release(dev);
```

为了在 D3D9 里使用上 D2D 需要创建 D3D11 设备，这部分逻辑只是用来测试，为了方便代码，我加上 SharpDx 的引用。值得一说的是 SharpDx 当前官方不维护了，可以选择的代替请看 [SharpDx 的代替项目](https://blog.lindexi.com/post/SharpDx-%E7%9A%84%E4%BB%A3%E6%9B%BF%E9%A1%B9%E7%9B%AE.html )

```xml
  <ItemGroup>
    <PackageReference Include="SharpDX" Version="4.2.0" />
    <PackageReference Include="SharpDX.Direct2D1" Version="4.2.0" />
    <PackageReference Include="SharpDX.Direct3D11" Version="4.2.0" />
    <PackageReference Include="SharpDX.Direct3D9" Version="4.2.0" />
  </ItemGroup>
```

参考 [WPF 使用 SharpDX 在 D3DImage 显示](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-SharpDX-%E5%9C%A8-D3DImage-%E6%98%BE%E7%A4%BA.html ) 的定义逻辑，在 CreateRenderTarget 方法加上代码

```csharp
        private Texture2D CreateRenderTarget()
        {
            var width = ImageWidth;
            var height = ImageHeight;

            var renderDesc = new D3D11.Texture2DDescription
            {
                BindFlags = D3D11.BindFlags.RenderTarget | D3D11.BindFlags.ShaderResource,
                Format = DXGI.Format.B8G8R8A8_UNorm,
                Width = width,
                Height = height,
                MipLevels = 1,
                SampleDescription = new DXGI.SampleDescription(1, 0),
                Usage = D3D11.ResourceUsage.Default,
                OptionFlags = D3D11.ResourceOptionFlags.Shared,
                CpuAccessFlags = D3D11.CpuAccessFlags.None,
                ArraySize = 1
            };

            var device = new D3D11.Device(DriverType.Hardware, D3D11.DeviceCreationFlags.BgraSupport);
            var renderTarget = new D3D11.Texture2D(device, renderDesc);
            var surface = renderTarget.QueryInterface<DXGI.Surface>();

            var d2DFactory = new D2D.Factory();

            var supportRequired = FormatSupport.RenderTarget;
            var isSupported = device.CheckFormatSupport(DXGI.Format.B8G8R8A8_UNorm).HasFlag(supportRequired);
            if (isSupported)
            {

            }

            var renderTargetProperties =
                new D2D.RenderTargetProperties(new D2D.PixelFormat(DXGI.Format.B8G8R8A8_UNorm, D2D.AlphaMode.Premultiplied));
            _d2DRenderTarget = new D2D.RenderTarget(d2DFactory, surface, renderTargetProperties);

            device.ImmediateContext.Rasterizer.SetViewport(0, 0, ImageWidth, ImageHeight);

            return renderTarget;
        }
```

调用 CreateRenderTarget 可以拿到 D3D11.Texture2D 对象，可以使用此作为 SharedHandle 创建 IDirect3DTexture9 对象

```csharp
            D3D11.Texture2D d3d11Texture2D = CreateRenderTarget();

            var format = TranslateFormat(TranslateFormat(d3d11Texture2D));

            var dxgiResource = d3d11Texture2D.QueryInterface<DXGI.Resource>();
            var pSharedHandle = dxgiResource.SharedHandle;

            hr = m_device.CreateTexture(ImageWidth,
                ImageHeight,
                1,
                1,
                format,
                0,
                out m_privateTexture,
                ref pSharedHandle);
```

上面代码的 TranslateFormat 如下

```csharp
        private static D3DFORMAT TranslateFormat(D3D9.Format format)
         => format switch
         {
             D3D9.Format.A8R8G8B8 => D3DFORMAT.D3DFMT_A8R8G8B8,
             _ => throw new ArgumentException(),
         };

        private static D3D9.Format TranslateFormat(D3D11.Texture2D texture)
        {
            switch (texture.Description.Format)
            {
                case SharpDX.DXGI.Format.R10G10B10A2_UNorm:
                    return D3D9.Format.A2B10G10R10;
                case SharpDX.DXGI.Format.R16G16B16A16_Float:
                    return D3D9.Format.A16B16G16R16F;
                case SharpDX.DXGI.Format.B8G8R8A8_UNorm:
                    return D3D9.Format.A8R8G8B8;
                default:
                    return D3D9.Format.Unknown;
            }
        }
```

拿到 m_privateTexture 对象，即可使用 GetSurfaceLevel 方法获取到可以给 D3DImage 的 BackBuffer 的参数

```csharp
            hr = m_privateTexture.GetSurfaceLevel(0, out m_privateSurface);

            var backBuffer = Marshal.GetIUnknownForObject(m_privateSurface);

            D3DImage.Lock();
            D3DImage.SetBackBuffer(D3DResourceType.IDirect3DSurface9, backBuffer, true);
            D3DImage.Unlock();
```


以上就完成了初始化逻辑，参数和 WPFMediaKit 相同，接下来是通过 D2D 进行渲染

```csharp
        private async void Render()
        {
            float x = 0;
            float y = 0;
            const float dx = 1;
            const float dy = 1;

            while (Dispatcher.CheckAccess())
            {
                var renderTarget = _d2DRenderTarget;

                renderTarget.BeginDraw();

                renderTarget.Clear(new RawColor4(Random.Shared.NextSingle(), Random.Shared.NextSingle(), Random.Shared.NextSingle(), 1));
                var brush = new D2D.SolidColorBrush(_d2DRenderTarget, new RawColor4(1, 0, 0, 1));

                renderTarget.DrawRectangle(new RawRectangleF(x, y, x + 10, y + 10), brush);

                x += dx;
                y += dy;
                if (x >= ImageWidth)
                {
                    x = 0;
                }

                if (y >= ImageHeight)
                {
                    y = 0;
                }

                renderTarget.EndDraw();

                D3DImage.Lock();
                D3DImage.AddDirtyRect(new Int32Rect(0, 0, D3DImage.PixelWidth, D3DImage.PixelHeight));
                D3DImage.Unlock();

                Image.InvalidateVisual();

                await Task.Delay(16);
            }
        }
```

以上就是测试 WPFMediaKit 的代码

本文所有代码放在[github](https://github.com/lindexi/lindexi_gd/tree/01e1dadff23f9481fa5ab99a52f2c0b64ad96fac/NaferfairqeLidajekawnal) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/01e1dadff23f9481fa5ab99a52f2c0b64ad96fac/NaferfairqeLidajekawnal) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 01e1dadff23f9481fa5ab99a52f2c0b64ad96fac
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 NaferfairqeLidajekawnal 文件夹

测试效果如下：

测试机器配置如下

- CPU i3 10100
- 内存 4G 2667MHz
- GPU Intel(R) UHD Graphics 630

系统版本是 19041.1348 版本

运行效果为最大化窗口 4k 分辨率，刷新率 36 左右下跑满 GPU 但不卡。其中 GPU 有百分之20是 DWM 占用

更多 DX 相关请看 [WPF 使用 SharpDx 渲染博客导航](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-SharpDx-%E6%B8%B2%E6%9F%93%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

[Direct3D 9Ex improvements - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/direct3darticles/direct3d-9ex-improvements?WT.mc_id=WD-MVP-5003260 )

[Surface sharing between Windows graphics APIs - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/direct3darticles/surface-sharing-between-windows-graphics-apis?WT.mc_id=WD-MVP-5003260 )

[IDirect3DSurface9 (d3d9helper.h) - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/api/d3d9helper/nn-d3d9helper-idirect3dsurface9?WT.mc_id=WD-MVP-5003260 )

[IDirect3DSurface9::LockRect (d3d9helper.h) - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/api/d3d9helper/nf-d3d9helper-idirect3dsurface9-lockrect?WT.mc_id=WD-MVP-5003260 )

[IDirect3DSurface9::GetDC (d3d9helper.h) - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/api/d3d9helper/nf-d3d9helper-idirect3dsurface9-getdc?WT.mc_id=WD-MVP-5003260 )

[SuppressUnmanagedCodeSecurityAttribute 类 (System.Security)](https://docs.microsoft.com/zh-cn/dotnet/api/system.security.suppressunmanagedcodesecurityattribute?WT.mc_id=WD-MVP-5003260 )

[c# - SharpDX. Unsupported pixel format - Stack Overflow](https://stackoverflow.com/questions/16318305/sharpdx-unsupported-pixel-format )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。