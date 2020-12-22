
# dotnet 读 WPF 源代码笔记 启动欢迎界面 SplashScreen 的原理

本文是我在读 WPF 源代码做的笔记。在 WPF 中的启动界面，为了能让 WPF 的启动界面显示足够快，需要在应用的 WPF 主机还没有启动完成之前就显示出启动图，此时的启动图需要自己解析图片同时也需要自己创建显示窗口

<!--more-->


<!-- CreateTime:2020/12/21 9:04:51 -->


<!-- 标签：WPF，WPF源代码 -->
<!-- 发布 -->

从 WPF 的 `src\Microsoft.DotNet.Wpf\src\WindowsBase\System\Windows\SplashScreen.cs` 文件可以看到 WPF 的 SplashScreen 的核心逻辑

在 SplashScreen 的构造函数会传入资源名，也就是启动图的资源名，或者加上指定程序集和图片资源名

```csharp
        public SplashScreen(string resourceName) : this(Assembly.GetEntryAssembly(), resourceName)
        {
        }

        public SplashScreen(Assembly resourceAssembly, string resourceName)
        {
        	// 忽略代码
        }
```

当然了，这个设计扩展性不够好哈，不支持指定任意的图片。如果想要指定本地路径的任意图片作为启动图的，可以使用 lsj 提供的 [kkwpsv/SplashImage: Fast splash Image with GDI+ in C#](https://github.com/kkwpsv/SplashImage) 库，当然了，这个库代码量特别少，我推荐大家可以抄抄代码。这个库提供的是高性能的版本，可以在另一个线程中执行，换句话说，就是使用 [kkwpsv/SplashImage](https://github.com/kkwpsv/SplashImage) 作为欢迎界面，是可以做到不占用 WPF 主线程时间的，性能比 WPF 提供的好

在 WPF 的 SplashScreen 的 Show 方法，就是启动图的核心逻辑

先调用 GetResourceStream 从自己的程序集里面读取图片资源的原始 Stream 对象，通过此方式的读取性能特别强，因此不是真的读取到内存里面，而是获取一个指针而已。但是有趣的是在这个方法上面有注释说比 Assembly.GetManifestResourceStream 慢 200-300 毫秒，也许是当年的设备才需要这么长的时间

```csharp
        // This is 200-300 ms slower than Assembly.GetManifestResourceStream() but works with localization.
        private UnmanagedMemoryStream GetResourceStream()
```

在获取到启动图片的 UnmanagedMemoryStream 之后，将使用下面代码转换为指针，用于后续传入给 WIC 层

```csharp
IntPtr pImageSrcBuffer;
unsafe
{
    pImageSrcBuffer = new IntPtr(umemStream.PositionPointer);
}
```

接下来就是调用 CreateLayeredWindowFromImgBuffer 创建一个窗口然后这个窗口显示图片内容

```csharp
if (CreateLayeredWindowFromImgBuffer(pImageSrcBuffer, umemStream.Length, topMost) && autoClose == true)
{
    Dispatcher.CurrentDispatcher.BeginInvoke(
        DispatcherPriority.Loaded,
        (DispatcherOperationCallback)ShowCallback,
        this);
}
```

可以看到在调用 CreateLayeredWindowFromImgBuffer 方法成功之后，就会调用 Dispatcher 插入 ShowCallback 函数，在 ShowCallback 里面用来自动关闭启动界面，如下面代码

```csharp
        private static object ShowCallback(object arg)
        {
            SplashScreen splashScreen = (SplashScreen)arg;
            splashScreen.Close(TimeSpan.FromSeconds(0.3));
            return null;
        }
```

从上面代码可以看到，在 WPF 中默认的启动图界面将会在 Loaded 完成之后延迟 0.3 秒执行，而具体是什么 Loaded 就不需要关注了。因为通过 BeginInvoke 插入的优先级是 DispatcherPriority.Loaded 优先级，也就是启动过程如果再没有什么比 DispatcherPriority.Loaded 更高的优先级，那就是启动完成了

在 WPF 里面的 SplashScreen 的核心逻辑里面包含以下三步

第一步是通过 WIC 层解码咱传入的图片，这样就支持不做任何优化的图片都能作为启动图

第二步就是将解码之后的图片编码为 BGRA 图片格式传给 GDI 图片对象，这样就能将咱的图片作为 GDI 图片对象能使用的资源

第三步是创建窗口显示这张 GDI 图片

回到创建窗口的核心方法 CreateLayeredWindowFromImgBuffer 上，这个方法里面大量调用 WIC 层的逻辑，用来处理图片的渲染，过程代码大概如下，下面代码为了方便说明，和 WPF 源代码有些不相同

```csharp
        private bool CreateLayeredWindowFromImgBuffer(IntPtr pImgBuffer, long cImgBufferLen, bool topMost)
        {
            bool bSuccess = false;
            IntPtr pImagingFactory = IntPtr.Zero;
            IntPtr pDecoder = IntPtr.Zero;
            IntPtr pIStream = IntPtr.Zero;
            IntPtr pDecodedFrame = IntPtr.Zero;
            IntPtr pBitmapSourceFormatConverter = IntPtr.Zero;
            IntPtr pBitmapFlipRotator = IntPtr.Zero;

            // 创建图片工厂，也就是获得 pImagingFactory 对象
            // 在 WPF 里面使用的 WINCODEC_SDK_VERSION 是 0x0236 一个比较古老的版本，在下文有告诉大家有哪些更新
            UnsafeNativeMethods.WIC.CreateImagingFactory(UnsafeNativeMethods.WIC.WINCODEC_SDK_VERSION,
                out pImagingFactory);

            // 使用 pImagingFactory 图片工厂创建出一个空的 Stream 返回指针给到 pIStream 变量
            // Use the WIC stream class to wrap the unmanaged pointer
            UnsafeNativeMethods.WIC.CreateStream(pImagingFactory, out pIStream);

            // 使用传进来的图片指针和长度，初始化图片工厂创建出来的 pIStream 对象
            UnsafeNativeMethods.WIC.InitializeStreamFromMemory(pIStream, pImgBuffer, (uint) cImgBufferLen);

            // Create an object that will decode the encoded image
            Guid vendor = Guid.Empty;
            // 拿到编解码器
            UnsafeNativeMethods.WIC.CreateDecoderFromStream(pImagingFactory, pIStream,
                ref vendor, 0, out pDecoder);

            // Get the frame from the decoder. Most image formats have only a single frame, in the case
            // of animated gifs we are ok with only displaying the first frame of the animation.
            // 从图片解码里面获取图片的第一帧，如果是 Gif 图片也只是显示第一帧
            UnsafeNativeMethods.WIC.GetFrame(pDecoder, 0, out pDecodedFrame);

            // 获取格式转换器
            UnsafeNativeMethods.WIC.CreateFormatConverter(pImagingFactory, out pBitmapSourceFormatConverter);

            // 定义了 32 位的 BGRA 图片格式，转换为此格式方便创建窗口使用 GDI 渲染
            // Convert the image from whatever format it is in to 32bpp premultiplied alpha BGRA
            Guid pixelFormat = UnsafeNativeMethods.WIC.WICPixelFormat32bppPBGRA;

            // 初始化转换器
            UnsafeNativeMethods.WIC.InitializeFormatConverter(pBitmapSourceFormatConverter, pDecodedFrame,
                ref pixelFormat, 0 /*DitherTypeNone*/, IntPtr.Zero,
                0, UnsafeNativeMethods.WIC.WICPaletteType.WICPaletteTypeCustom);
            // Reorient the image
            // 获取图片的裁剪和旋转
            UnsafeNativeMethods.WIC.CreateBitmapFlipRotator(pImagingFactory, out pBitmapFlipRotator);

            // 初始化图片的裁剪和旋转特效，此时的 pBitmapFlipRotator 就是最终叠加了特效的图片
            UnsafeNativeMethods.WIC.InitializeBitmapFlipRotator(pBitmapFlipRotator, pBitmapSourceFormatConverter,
                UnsafeNativeMethods.WIC.WICBitmapTransformOptions.WICBitmapTransformFlipVertical);

            // 获取图片的大小，用来在下面创建像素数组
            Int32 width, height;
            UnsafeNativeMethods.WIC.GetBitmapSize(pBitmapFlipRotator, out width, out height);

            // 因为一个像素由 BGRA 格式定义
            Int32 stride = width * 4;

            // 创建一个 GDI 对象，对象的大小通过上面的逻辑拿到
            // initialize the bitmap header
            MS.Win32.NativeMethods.BITMAPINFO bmInfo = new MS.Win32.NativeMethods.BITMAPINFO(width, height, 32 /*bpp*/);
            bmInfo.bmiHeader_biCompression = MS.Win32.NativeMethods.BI_RGB;
            bmInfo.bmiHeader_biSizeImage = (int) (stride * height);

            // 创建 GDI 图片对象的内存填充
            // Create a 32bpp DIB.  This DIB must have an alpha channel for UpdateLayeredWindow to succeed.
            IntPtr pBitmapBits = IntPtr.Zero;
            _hBitmap = UnsafeNativeMethods.CreateDIBSection(new HandleRef(), ref bmInfo, 0 /* DIB_RGB_COLORS*/,
                ref pBitmapBits, null, 0);

            // 从 WIC 解码器里面拷贝像素内容到 GDI 图片里面
            // Copy the decoded image to the new buffer which backs the HBITMAP
            Int32Rect rect = new Int32Rect(0, 0, width, height);
            UnsafeNativeMethods.WIC.CopyPixels(pBitmapFlipRotator, ref rect, stride, stride * height, pBitmapBits);

            // 使用 GDI 图片 _hBitmap 去创建一个窗口
            _hwnd = CreateWindow(_hBitmap, width, height, topMost);

            bSuccess = true;
            // 忽略一些清理资源的代码
            return bSuccess;
        }
```

上面代码中的 UnsafeNativeMethods.WIC 就是调用 WIC 层的逻辑，在 WPF 中的 WIC 层逻辑和其他 Win32 应用一样，通过 WindowsCodecs.dll 提供，只是在 `UnsafeNativeMethods.WIC.CreateImagingFactory(UnsafeNativeMethods.WIC.WINCODEC_SDK_VERSION, out pImagingFactory);` 可以看到 WPF 使用的版本是 0x236 比较古老

通过对比 6.2.9200.21830-Windows_7.0 和 6.3.9600.17415-Windows_8.1 版本的 windowscodecs.dll 可以看到有做了如下的更改

```diff
- #define WINCODEC_SDK_VERSION 0x0236
+ #define WINCODEC_SDK_VERSION1 0x0236
+ #define WINCODEC_SDK_VERSION2 0x0237
- DEFINE_GUID(CLSID_WICImagingFactory, 0xcacaf262, 0x9370, 0x4615, 0xa1, 0x3b, 0x9f, 0x55, 0x39, 0xda, 0x4c, 0xa);
+ DEFINE_GUID(CLSID_WICImagingFactory, 0xcacaf262, 0x9370, 0x4615, 0xa1, 0x3b, 0x9f, 0x55, 0x39, 0xda, 0x4c, 0xa);
+ DEFINE_GUID(CLSID_WICImagingFactory1, 0xcacaf262, 0x9370, 0x4615, 0xa1, 0x3b, 0x9f, 0x55, 0x39, 0xda, 0x4c, 0xa);
+ DEFINE_GUID(CLSID_WICImagingFactory2, 0x317d06e8, 0x5f24, 0x433d, 0xbd, 0xf7, 0x79, 0xce, 0x68, 0xd8, 0xab, 0xc2);
+ #if(_WIN32_WINNT >= _WIN32_WINNT_WIN8) || defined(_WIN7_PLATFORM_UPDATE)
+    #define WINCODEC_SDK_VERSION WINCODEC_SDK_VERSION2
+    #define CLSID_WICImagingFactory CLSID_WICImagingFactory2
+ #else
+    #define WINCODEC_SDK_VERSION WINCODEC_SDK_VERSION1
+ #endif
```

新版本的 WindowsCodecs.dll 更新请看 [What's New in WIC - Win32 apps](https://docs.microsoft.com/zh-cn/windows/win32/wic/what-s-new-in-wic-for-windows-8-1)

在调用到使用 GDI 图片创建窗口的逻辑就十分简单了，都是一些 Win32 的接口调用

```csharp
        private IntPtr CreateWindow(NativeMethods.BitmapHandle hBitmap, int width, int height, bool topMost)
        {
            if (_defWndProc == null)
            {
                _defWndProc = new MS.Win32.NativeMethods.WndProc(UnsafeNativeMethods.DefWindowProc);
            }

            // 基本的 Win32 窗口创建方法，没啥特别的
            MS.Win32.NativeMethods.WNDCLASSEX_D wndClass = new MS.Win32.NativeMethods.WNDCLASSEX_D();
            wndClass.cbSize = Marshal.SizeOf(typeof(MS.Win32.NativeMethods.WNDCLASSEX_D));
            wndClass.style = 3; /* CS_HREDRAW | CS_VREDRAW */
            wndClass.lpfnWndProc = null;
            wndClass.hInstance = _hInstance;
            wndClass.hCursor = IntPtr.Zero;
            wndClass.lpszClassName = CLASSNAME;
            wndClass.lpszMenuName = string.Empty;
            // 加上消息循环，不然会提示应用停止响应
            wndClass.lpfnWndProc = _defWndProc;

            // We chose to ignore re-registration errors in RegisterClassEx on the off chance that the user
            // wants to open multiple splash screens.
            _wndClass = MS.Win32.UnsafeNativeMethods.IntRegisterClassEx(wndClass);
            if (_wndClass == 0)
            {
            	// 下面代码不太合理，于是我就提了一个更改 https://github.com/dotnet/wpf/pull/3923
                if (Marshal.GetLastWin32Error() != 0x582) /* class already registered */
                    throw new Win32Exception();
            }

            // 决定启动窗口显示到哪
            int screenWidth = MS.Win32.UnsafeNativeMethods.GetSystemMetrics(SM.CXSCREEN);
            int screenHeight = MS.Win32.UnsafeNativeMethods.GetSystemMetrics(SM.CYSCREEN);
            int x = (screenWidth - width) / 2;
            int y = (screenHeight - height) / 2;

            // 窗口的样式，核心的就是 WS_EX_LAYERED 和 WS_EX_TOOLWINDOW 样式
            HandleRef nullHandle = new HandleRef(null, IntPtr.Zero);
            int windowCreateFlags =
                (int) NativeMethods.WS_EX_WINDOWEDGE |
                      NativeMethods.WS_EX_TOOLWINDOW |
                      NativeMethods.WS_EX_LAYERED |
                      // 是否显示到最前
                      (topMost ? NativeMethods.WS_EX_TOPMOST : 0);

            // 创建窗口
            // CreateWindowEx will either succeed or throw
            IntPtr hWnd =  MS.Win32.UnsafeNativeMethods.CreateWindowEx(
                windowCreateFlags,
                CLASSNAME, SR.Get(SRID.SplashScreenIsLoading),
                MS.Win32.NativeMethods.WS_POPUP | MS.Win32.NativeMethods.WS_VISIBLE,
                x, y, width, height,
                nullHandle, nullHandle, new HandleRef(null, _hInstance), IntPtr.Zero);

            // 将图片在窗口上显示出来
            // Display the image on the window
            IntPtr hScreenDC = UnsafeNativeMethods.GetDC(new HandleRef());
            IntPtr memDC = UnsafeNativeMethods.CreateCompatibleDC(new HandleRef(null, hScreenDC));
            IntPtr hOldBitmap = UnsafeNativeMethods.SelectObject(new HandleRef(null, memDC), hBitmap.MakeHandleRef(null).Handle);

            NativeMethods.POINT newSize = new NativeMethods.POINT(width, height);
            NativeMethods.POINT newLocation = new NativeMethods.POINT(x, y);
            NativeMethods.POINT sourceLocation = new NativeMethods.POINT(0, 0);
            _blendFunc = new NativeMethods.BLENDFUNCTION();
            _blendFunc.BlendOp = NativeMethods.AC_SRC_OVER;
            _blendFunc.BlendFlags = 0;
            _blendFunc.SourceConstantAlpha = 255;
            _blendFunc.AlphaFormat = 1; /*AC_SRC_ALPHA*/

            bool result = UnsafeNativeMethods.UpdateLayeredWindow(hWnd, hScreenDC, newLocation, newSize,
                memDC, sourceLocation, 0, ref _blendFunc, NativeMethods.ULW_ALPHA);

            UnsafeNativeMethods.SelectObject(new HandleRef(null, memDC), hOldBitmap);
            UnsafeNativeMethods.ReleaseDC(new HandleRef(), new HandleRef(null, memDC));
            UnsafeNativeMethods.ReleaseDC(new HandleRef(), new HandleRef(null, hScreenDC));

            if (result == false)
            {
                UnsafeNativeMethods.HRESULT.Check(Marshal.GetHRForLastWin32Error());
            }

            return hWnd;
        }

```

当然了，在 WPF 里面再快的启动图显示速度都不如 UWP 快，因此 UWP 是系统给的优化，通过 AppFrameHost 显示的，基本上点击应用立刻打开

当前的 WPF 在 [https://github.com/dotnet/wpf](https://github.com/dotnet/wpf) 完全开源，使用友好的 MIT 协议，意味着允许任何人任何组织和企业任意处置，包括使用，复制，修改，合并，发表，分发，再授权，或者销售。在仓库里面包含了完全的构建逻辑，只需要本地的网络足够好（因为需要下载一堆构建工具），即可进行本地构建





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。