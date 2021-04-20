# dotnet 读 WPF 源代码笔记 WriteableBitmap 的渲染和更新是如何实现

在 WPF 框架提供方便进行像素读写的 WriteableBitmap 类，本文来告诉大家在咱写下像素到 WriteableBitmap 渲染，底层的逻辑

<!--more-->
<!-- 标签：WPF，WPF源代码 -->
<!-- 发布 -->

之前我使用 WriteableBitmap 进行 CPU 高性能绘图时，在性能调试遇到一个问题，写入到 WriteableBitmap 的像素会经过两次拷贝。其中一次是我自己拷贝到 WriteableBitmap 而另一次拷贝就在 WriteableBitmap 里面。无论设置 WriteableBitmap 的脏区多大，渲染的时候是整个图片渲染 。本来按照我的阅读顺序，当前还没有阅读到 WriteableBitmap 的代码，但是有小伙伴和我报告了 WriteableBitmap 的坑，因此我就开始阅读 WriteableBitmap 详细请看 [dotnet 读 WPF 源代码笔记 了解 WPF 已知问题 后台线程创建 WriteableBitmap 锁住主线程](https://blog.lindexi.com/post/dotnet-%E8%AF%BB-WPF-%E6%BA%90%E4%BB%A3%E7%A0%81%E7%AC%94%E8%AE%B0-%E4%BA%86%E8%A7%A3-WPF-%E5%B7%B2%E7%9F%A5%E9%97%AE%E9%A2%98-%E5%90%8E%E5%8F%B0%E7%BA%BF%E7%A8%8B%E5%88%9B%E5%BB%BA-WriteableBitmap-%E9%94%81%E4%BD%8F%E4%B8%BB%E7%BA%BF%E7%A8%8B.html )

在开始之前，先聊聊 WriteableBitmap 是什么？在 WPF 和 UWP 中提供的 WriteableBitmap 是支持对像素写入而更改渲染的图片，当然，本文只聊 WPF 的源代码，关于 UWP 部分，咱只知道使用就可以。通过 WriteableBitmap 可以用来实现高性能的 CPU 渲染，以下是我的其他 WriteableBitmap 博客

- [WPF 使用 Skia 绘制 WriteableBitmap 图片](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Skia-%E7%BB%98%E5%88%B6-WriteableBitmap-%E5%9B%BE%E7%89%87.html )
- [WPF 如何在 WriteableBitmap 写文字](https://blog.lindexi.com/post/WPF-%E5%A6%82%E4%BD%95%E5%9C%A8-WriteableBitmap-%E5%86%99%E6%96%87%E5%AD%97.html )
- [WPF 使用不安全代码快速从数组转 WriteableBitmap](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8%E4%B8%8D%E5%AE%89%E5%85%A8%E4%BB%A3%E7%A0%81%E5%BF%AB%E9%80%9F%E4%BB%8E%E6%95%B0%E7%BB%84%E8%BD%AC-WriteableBitmap.html )

在 WriteableBitmap 进行绘制时，有一个重要的功能是设置 DirtyRect 来告诉 WPF 层，当前需要更新的是 WriteableBitmap 的哪个内容。在调试时，可以看到如果 DirtyRect 很小，那么 CPU 占用也将会很小，但渲染时依然是渲染整个图片。在聊到 WriteableBitmap 的渲染和更新，就一定需要先聊到 AddDirtyRect 方法，下面咱看一下 AddDirtyRect 方法的实现

```csharp
        public void AddDirtyRect(Int32Rect dirtyRect)
        {
            WritePreamble();

            if (_lockCount == 0)
            {
                throw new InvalidOperationException(SR.Get(SRID.Image_MustBeLocked));
            }

            //
            // Sanitize the dirty rect.
            //
            dirtyRect.ValidateForDirtyRect("dirtyRect", _pixelWidth, _pixelHeight);
            if (dirtyRect.HasArea)
            {
                MILSwDoubleBufferedBitmap.AddDirtyRect(
                    _pDoubleBufferedBitmap,
                    ref dirtyRect);

                _hasDirtyRects = true;
            }

            // Note: we do not call WritePostscript because we do not want to
            // raise change notifications until the writeable bitmap is unlocked.
        }
```

调用 AddDirtyRect 基本都会在 Lock 和 Unlock 方法里面，但无论是 Lock 还是 Unlock 和渲染触发其实都没有关系，咱继续回到 AddDirtyRect 方法。在这个方法里面实际的调用就是 MILSwDoubleBufferedBitmap.AddDirtyRect 方法，这是一个从 MIL 层拿到的方法

```csharp
        [DllImport(DllImport.MilCore, EntryPoint = "MILSwDoubleBufferedBitmapAddDirtyRect", PreserveSig = false)]
        internal static extern void AddDirtyRect(
            SafeMILHandle /* CSwDoubleBufferedBitmap */ THIS_PTR,
            ref Int32Rect dirtyRect
            );
```

从上面的注释可以看到，这里的 SafeMILHandle 的 `THIS_PTR` 就是 CSwDoubleBufferedBitmap 类型，这个类型定义在 MIL 层，代码在 `src\Microsoft.DotNet.Wpf\src\WpfGfx\core\sw\swlib\doublebufferedbitmap.cpp` 文件。通过上面代码可以看到，就是定义在字段的 `_pDoubleBufferedBitmap` 字段

```csharp
        private SafeMILHandle _pDoubleBufferedBitmap;   // CSwDoubleBufferedBitmap
```

先忽略 `_pDoubleBufferedBitmap` 的创建，咱进入 MILSwDoubleBufferedBitmapAddDirtyRect 方法的实现。这是定义在 exports.cpp 的方法

```c++
HRESULT
MILSwDoubleBufferedBitmapAddDirtyRect(
    __in CSwDoubleBufferedBitmap * THIS_PTR,
    __in const MILRect *pRect
    )
{
    HRESULT hr = S_OK;
    UINT x = 0;
    UINT y = 0;
    UINT width = 0;
    UINT height = 0;
    CMilRectU rcDirty;

    CHECKPTR(THIS_PTR);
    CHECKPTR(pRect);

    IFC(IntToUInt(pRect->X, &x));
    IFC(IntToUInt(pRect->Y, &y));
    IFC(IntToUInt(pRect->Width, &width));
    IFC(IntToUInt(pRect->Height, &height));

    // Since we converted x, y, width, and height from ints, we can add them
    // together and remain within a UINT.
    rcDirty = CMilRectU(x, y, width, height, XYWH_Parameters);

    IFC(THIS_PTR->AddDirtyRect(&rcDirty));

Cleanup:

    RRETURN(hr);
}
```

这里的逻辑是在 MIL 层了，这一层就是实际处理多媒体的逻辑，可以看到上面代码核心的方法就是 `THIS_PTR->AddDirtyRect(&rcDirty)` 调用 CSwDoubleBufferedBitmap 的 AddDirtyRect 方法。在 AddDirtyRect 方法里面实际上就是维护一个去掉重复范围的 Rect 列表而已，只是因为用了 C++ 编写，代码看起来有点杂

```csharp
HRESULT
CSwDoubleBufferedBitmap::AddDirtyRect(__in const CMilRectU *prcDirty)
{
    HRESULT hr = S_OK;
    CMilRectU rcBounds(0, 0, m_width, m_height, XYWH_Parameters);
    CMilRectU rcDirty = *prcDirty;

    if (!rcDirty.IsEmpty())
    {
        // Each dirty rect will eventually be treated as a RECT, so we must
        // ensure that the Left, Right, Top, and Bottom values never exceed
        // INT_MAX.  We already restrict our dimensions to INT_MAX, so as
        // long as the dirty rect is fully within the bounds of the bitmap,
        // we are safe.
        if (!rcBounds.DoesContain(rcDirty))
        {
            IFC(E_INVALIDARG);
        }

        // Adding a dirty rect that spans the entire bitmap will simply
        // replace all existing dirty rects.
        if (rcDirty.IsEquivalentTo(rcBounds))
        {
            m_pDirtyRects[0] = rcBounds;
            m_numDirtyRects = 1;
        }
        else
        {
            // Check to see if one of the existing dirty rects fully contains the
            // new dirty rect.  If so, there is no need to add it.
            for (UINT i = 0; i < m_numDirtyRects; i++)
            {
                if (m_pDirtyRects[i].DoesContain(rcDirty))
                {
                    // No dirty list change - new dirty rect is already included.
                    goto Cleanup;
                }
            }

            // Collapse existing dirty rects if we're about to exceed our maximum.
            if (m_numDirtyRects >= c_maxBitmapDirtyListSize)
            {
                // Collapse dirty list to a single large rect (including new rect)
                while (m_numDirtyRects > 1)
                {
                    m_pDirtyRects[0].Union(m_pDirtyRects[--m_numDirtyRects]);
                }
                m_pDirtyRects[0].Union(rcDirty);

                Assert(m_numDirtyRects == 1);
            }
            else
            {
                m_pDirtyRects[m_numDirtyRects++] = rcDirty;
            }
        }
    }

Cleanup:

    RRETURN(hr);
}
```

上面代码是将传入的参数，合入到 m_pDirtyRects 字段里面

可以看到在调用咱的 AddDirtyRect 方法时，其实就是更新 CSwDoubleBufferedBitmap 的 m_pDirtyRects 字段而已，而此时依然没有做渲染相关逻辑。从 `CSwDoubleBufferedBitmap` 这个命名可以看到，这是双缓存的做法。两个缓存，前面的缓存是用在实际显示的对象，后面的缓存是用的是一个数组用于给 WPF 上层使用访问

在 WPF 的渲染过程中，按照 DirectX 应用的渲染步骤，第一步就是收集过程，在收集过程中收集绘制信息。收集过程中将会调用到 CSwDoubleBufferedBitmap 的 CopyForwardDirtyRects 方法，这个方法的作用就是根据脏区从后面的缓存将像素复制到前面的缓存。虽然这个类的命名是双缓存，但实际上的做法不是在渲染的时候交换两个缓存的指针，而是在渲染收集过程中，从后面的缓存拷贝数据到前面的缓存

以下是 CopyForwardDirtyRects 方法的代码，我在代码里面添加了一些注释

```c++
HRESULT
CSwDoubleBufferedBitmap::CopyForwardDirtyRects()
{
    HRESULT hr = S_OK;

    IWGXBitmapSource *pIWGXBitmapSource = NULL;
    IWGXBitmapLock *pFrontBufferLock = NULL;
    UINT cbLockStride = 0;
    UINT cbBufferSize = 0;
    BYTE *pbSurface = NULL;

    Assert(m_pBackBuffer);

    // 根据调用 AddDirtyRect 方法加入的 DirtyRect 获取当前有哪些需要拷贝的像素
    // This locks only the rect specified as dirty for each copy. It would
    // be more efficient to just lock the entire rect once for all of the
    // copies, but then we need to manually compute offsets into the front
    // buffer specific to each pixel format.
    while (m_numDirtyRects > 0)
    {
        // We have to jump through a few RECT hoops here since
        // IWGXBitmapSource::Lock/CopyPixels take a WICRect and
        // IWGXBitmap::AddDirtyRect takes a GDI RECT, neither of which are
        // CMilRectU which we use in CSwDoubleBufferedBitmap for geometric operations.
        //
        // CMilRectU and RECT share the same memory alignment, but different
        // signs.  Since we restrict the size of our bitmap to MAX_INT, we can
        // safely cast.
        // 这里只是做一层转换而已，拿到当前的一个 DirtyRect 范围
        const RECT *rcDirty = reinterpret_cast<RECT const *>(&m_pDirtyRects[--m_numDirtyRects]);
        WICRect copyRegion = {
            static_cast<int>(rcDirty->left),
            static_cast<int>(rcDirty->top),
            static_cast<int>(rcDirty->right - rcDirty->left),
            static_cast<int>(rcDirty->bottom - rcDirty->top)
            };

        // 根据 IWICBitmapSource 的使用文档，在使用之前需要先加上锁
        // This adds copyRegion as a dirty rect to m_pFrontBuffer automatically.
        IFC(m_pFrontBuffer->Lock(
            &copyRegion,
            MilBitmapLock::Write,
            &pFrontBufferLock
            ));

        IFC(pFrontBufferLock->GetStride(&cbLockStride));
        IFC(pFrontBufferLock->GetDataPointer(&cbBufferSize, &pbSurface));

        // If a format converter has been allocated, it is necessary that we call copy
        // pixels through it rather than directly from the back buffer since its very
        // existence implies that a conversion is needed.
        GetPossiblyFormatConvertedBackBuffer(&pIWGXBitmapSource);

        // 这里的 IFC 是一个宏，表示的是如果返回值是 gg 的，那么 goto 到 Cleanup 标签
        /*
        * #ifndef IFC
          #define IFC(x) { hr = (x); if (FAILED(hr)) goto Cleanup; }
          #endif
        */
        // 下面代码就是核心逻辑，通过 CopyPixels 方法从后面的缓存也就是 WPF 层的数据拷贝到前面的缓存用于显示
        // 在这一层里面其实就丢失了 DirtyRect 信息
        IFC(pIWGXBitmapSource->CopyPixels(
            &copyRegion,
            cbLockStride,
            cbBufferSize,
            pbSurface
            ));

        // 释放掉锁
        // We need to release the lock and format converter here because we are in a loop.
        ReleaseInterface(pIWGXBitmapSource);
        ReleaseInterface(pFrontBufferLock);
    }

Cleanup:
    ReleaseInterfaceNoNULL(pIWGXBitmapSource);
    ReleaseInterfaceNoNULL(pFrontBufferLock);

    RRETURN(hr);
}
```

从上面代码可以看到，咱在使用 WriteableBitmap 的两次复制的第二次复制就是上面的代码，通过 `pIWGXBitmapSource->CopyPixels` 的过程就会依赖传入的 DirtyRect 决定拷贝的数据量。也就是说通过 DirtyRect 能优化的性能也只是更新前面的缓存用到的拷贝的性能，我没有在官方文档里面找到 CopyPixels 里面还会记录 DirtyRect 的功能，同时也没有在 WPF 自定义渲染管线里面找到只刷新图片某个范围的逻辑，因此可以认为使用 WriteableBitmap 的更新，设置 DirtyRect 只影响第二次复制数据的性能，而不会影响渲染性能，依然是整个图片进行渲染

在拷贝到前面的缓存之后，在 WPF 中是在自定义渲染管线里面将前面的缓存作为纹理绘制到形状上，在 WPF 上，可以将 WriteableBitmap 作为 BitmapSource 放入到不规则形状上，将图片作为纹理绘制到形状上能做到比较通用。关于 WPF 的从图片到渲染的步骤，就需要额外的文档来告诉大家

当前的 WPF 在 [https://github.com/dotnet/wpf](https://github.com/dotnet/wpf) 完全开源，使用友好的 MIT 协议，意味着允许任何人任何组织和企业任意处置，包括使用，复制，修改，合并，发表，分发，再授权，或者销售。在仓库里面包含了完全的构建逻辑，只需要本地的网络足够好（因为需要下载一堆构建工具），即可进行本地构建

详细请看 [IWICBitmapSource::CopyPixels (wincodec.h) - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/api/wincodec/nf-wincodec-iwicbitmapsource-copypixels?WT.mc_id=WD-MVP-5003260 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
