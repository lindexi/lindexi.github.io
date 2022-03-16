
# 读 WPF 源代码的疑惑记录

本文记录读 WPF 源代码的疑惑

<!--more-->


<!-- CreateTime:2022/3/14 17:24:00 -->

<!-- 不发布 -->

## D3DImage 的 SetIsFrontBufferAvailable 为什么能对一个 object 强转为布尔

为什么在 D3DImage 的 Callback 方法里面，如下面代码，传入的第一个参数是 BooleanBoxes 

```
        private void Callback(bool isFrontBufferAvailable, uint version)
        {
            Dispatcher.BeginInvoke(
                DispatcherPriority.Normal,
                new DispatcherOperationCallback(SetIsFrontBufferAvailable),
                new Pair(BooleanBoxes.Box(isFrontBufferAvailable), version)
                );
        }
```

但是在 SetIsFrontBufferAvailable 方法里面，却使用的是 (bool) 转换

```
        private object SetIsFrontBufferAvailable(object isAvailableVersionPair)
        {
            Pair pair = (Pair)isAvailableVersionPair;
            uint version = (uint)pair.Second;

            if (version == _version)
            {
                bool isFrontBufferAvailable = (bool)pair.First;
                SetValue(IsFrontBufferAvailablePropertyKey, isFrontBufferAvailable);
            }

            // ...just because DispatcherOperationCallback requires returning an object
            return null;
        }
```

此时的转换，理论上应该出类型不相同

回答： [dotnet 读 WPF 源代码笔记 为什么加上 BooleanBoxes 类](https://blog.lindexi.com/post/dotnet-%E8%AF%BB-WPF-%E6%BA%90%E4%BB%A3%E7%A0%81%E7%AC%94%E8%AE%B0-%E4%B8%BA%E4%BB%80%E4%B9%88%E5%8A%A0%E4%B8%8A-BooleanBoxes-%E7%B1%BB.html)

## 拷贝 D3DImage 的性能

在 GFX 底层获取 FrontBufferUpdateMethod 方法的 GetUpdateMethod 代码里面，将进行评估传入的 BackBuffer 将如何加入到 WPF 的渲染上。将获取传入的 IDirect3DSurface9 的 D3DCAPS9 描述里面，在 D3DCAPS9 的 DevCaps2 是否包含 D3DDEVCAPS2_CAN_STRETCHRECT_FROM_TEXTURES 以及 Caps2 是否包含 D3DCAPS2_CANSHARERESOURCE 的功能

如果这两个都满足，就返回 SharedSurface 作为更新的方法。否则，默认将返回 Software 的方式。如果仅有 D3DDEVCAPS2_CAN_STRETCHRECT_FROM_TEXTURES 但是没有 D3DCAPS2_CANSHARERESOURCE 的，那就走 BitBlt 的方式

显然，走 Software 是最慢的，其次是走 BitBlt 的方式，最快是走 SharedSurface 的方法

```
CInteropDeviceBitmap::FrontBufferUpdateMethod 
CInteropDeviceBitmap::GetUpdateMethod(
    __in IDirect3DDevice9 *pID3DDevice,
    __in_opt const IDirect3DDevice9Ex *pID3DDeviceEx,
    __in IDirect3DSurface9 *pID3DSurface
    )
{   
    HRESULT hr = S_OK;
    FrontBufferUpdateMethod method = Software;

    D3DCAPS9 caps;
    IFC(pID3DDevice->GetDeviceCaps(&caps));

    if ((caps.DevCaps2 & D3DDEVCAPS2_CAN_STRETCHRECT_FROM_TEXTURES) == D3DDEVCAPS2_CAN_STRETCHRECT_FROM_TEXTURES)
    {
        if (pID3DDeviceEx && (caps.Caps2 & D3DCAPS2_CANSHARERESOURCE) == D3DCAPS2_CANSHARERESOURCE)
        {
            method = SharedSurface;
        }
        else if (!WPFUtils::OSVersionHelper::IsWindowsVistaOrGreater())
        {
            HDC hdc;
            IFC(pID3DSurface->GetDC(&hdc));
            IFC(pID3DSurface->ReleaseDC(hdc));

            // Failure of GetDC or ReleaseDC will skip this assignment
            method = BitBlt;
        }
    }

Cleanup:
    return method;
}
```

三个方法的枚举定义如下

```
    enum FrontBufferUpdateMethod
    {
        // Front buffer's handle will be opened on the back buffer's device and StretchRect'd
        SharedSurface,
        // Back buffer will be BitBlt to front buffer 
        BitBlt,
        // Back buffer will be copied to front buffer through software
        Software
    };
```

## 调用逻辑

将进入先进入 InteropDeviceBitmap_Create 方法，再进入 `CInteropDeviceBitmap::Create` 方法

## VisualBrush 的核心逻辑

获取输入的 Visual 的绘制树，接着调用底层的 DrawingContext 的 DrawVisualTree 方法。在底层，将让 CMilVisual 继承 IGraphNode 接口，实际逻辑就是走树遍历，让里层每一个进行绘制

也就是说在 Visual 层，收集的绘制指令，制作的 CMilVisual 数据，将会构建出 IGraphNode 用来实现绘制逻辑。也就是实际 VisualBrush 是需要在底层进行重复绘制





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。