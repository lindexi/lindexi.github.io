---
title: WPF 已知问题 传入错误数据给到 WriteableBitmap 可能导致渲染线程锁住
description: 本文记录一个 WPF 已知问题，此问题已经被我修复。传入错误的数据给到 WriteableBitmap 对象，比如调用 WritePixels 时传入错误的 stride 数值，将可能导致渲染线程进入无限自旋锁

<!--more-->

tags: WPF WPF源代码
category: 
---

<!-- CreateTime:2023/9/11 8:50:10 -->

<!-- 标签：WPF，WPF源代码 -->
<!-- 发布 -->
<!-- 博客 -->

## 问题描述

应用程序停止渲染，或者是界面未响应。表现是在 渲染 线程卡住，从任务管理器看可以看到有一个 CPU 核在跑，但没跑满。进行本机代码调试可以看到卡在如下调用堆栈

```
 	wpfgfx_cor3.dll!CWGXBitmapLockState::LockRead() line 1086	C++
 	wpfgfx_cor3.dll!CWGXBitmap::HrLock(const tagRECT & rcLock={...}, MilPixelFormat::Enum pxlFormat=BGRA32bpp, unsigned int cbStride=1200, unsigned int cbBufferSize=1200, void * pvPixels=0x0000028eea8a0000, unsigned long dwFlags=1, IWGXBitmapLock * * ppILock=0x0000009a90b7b268, int) line 743	C++
 	wpfgfx_cor3.dll!CSystemMemoryBitmap::Lock(const WICRect * prcLock, unsigned long dwFlags=1, IWGXBitmapLock * * ppILock=0x0000009a90b7b268) line 176	C++
 	wpfgfx_cor3.dll!CWriteProtectedBitmap::Lock(const WICRect * prcLock, unsigned long dwFlags, IWGXBitmapLock * * ppILock) line 125	C++
 	wpfgfx_cor3.dll!CWGXBitmap::CopyPixels(const WICRect * prc=0x0000009a90b7b4a0, unsigned int cbStride=1200, unsigned int cbBufferSize=1200, unsigned char * pbPixels=0x0000028edfc56540) line 477	C++
 	wpfgfx_cor3.dll!CWGXWrapperBitmap::CopyPixels(const WICRect * prc, unsigned int cbStride, unsigned int cbBufferSize, unsigned char * pbPixels=0x0000028edfc56540) line 411	C++
 	WindowsCodecs.dll!00007ffcaa688759()	
 	WindowsCodecs.dll!00007ffcaa6ab5d3()	
 	WindowsCodecs.dll!00007ffcaa68f9c6()	
 	wpfgfx_cor3.dll!CWGXWrapperBitmap::CopyPixels(const WICRect * prc, unsigned int cbStride, unsigned int cbBufferSize, unsigned char * pbPixels=0x0000028eebe50ad0) line 411	C++
 	wpfgfx_cor3.dll!CWGXWrapperBitmap::CopyPixels(const WICRect * prc, unsigned int cbStride, unsigned int cbBufferSize, unsigned char * pbPixels=0x0000028eebe50ad0) line 411	C++
 	wpfgfx_cor3.dll!CWGXWrapperBitmap::CopyPixels(const WICRect * prc, unsigned int cbStride, unsigned int cbBufferSize, unsigned char * pbPixels=0x0000028eebe50ad0) line 411	C++
 	wpfgfx_cor3.dll!CSystemMemoryBitmap::UnsafeUpdateFromSource(IWGXBitmapSource * pISource=0x0000028edfb110d0, MilRectU & rcSrc, unsigned int uDstLeft, unsigned int uDstTop=0) line 263	C++
 	wpfgfx_cor3.dll!CSwBitmapColorSource::FillTextureWithTransformedSource(IWGXBitmapSource * pIBitmapSource=0x0000028edfb110d0) line 706	C++
 	wpfgfx_cor3.dll!CSwBitmapColorSource::FillTexture() line 584	C++
 	wpfgfx_cor3.dll!CSwBitmapColorSource::Realize() line 882	C++
 	wpfgfx_cor3.dll!CSwBitmapColorSource::DeriveFromBitmapAndContext(IWGXBitmapSource * pIBitmap=0x0000028edfc73650, CMatrix<CoordinateSpace::RealizationSampling,CoordinateSpace::DeviceHPC> * pmatBitmapToSampleSpace=0x0000009a90b7b9f0, const CColorSourceCreator * pCSCreator, bool fPrefilterEnabled, float rPrefilterThreshold=1.41421354, IMILResourceCache * pICacheAlternate=0x0000028eebd59cb0, IWGXBitmap * * ppBitmap=0x0000009a90b7b9e0) line 124	C++
 	wpfgfx_cor3.dll!CColorSourceCreator::GetCS_PrefilterAndResample(IWGXBitmapSource * pIBitmapSource, MilBitmapWrapMode::Enum wrapMode=Extend, const _D3DCOLORVALUE * pBorderColor=0x0000028eebd59d84, const CMatrix<CoordinateSpace::RealizationSampling,CoordinateSpace::DeviceHPC> * pmatTextureHPCToDeviceHPC=0x0000009a90b7bae0, MilBitmapInterpolationMode::Enum interpolationMode=Linear, bool prefilterEnable=false, float prefilterThreshold=1.41421354, IMILResourceCache * pICacheAlternate=0x0000028eebd59cb0, CColorSource * * ppColorSource=0x0000009a90b7bc30) line 1131	C++
 	wpfgfx_cor3.dll!CSoftwareRasterizer::GetCS_Brush(CMILBrush * pBrush=0x0000028eebd59ca8, const CMatrix<CoordinateSpace::BaseSampling,CoordinateSpace::DeviceHPC> & matWorldHPCToDeviceHPC={...}, const CContextState * pContextState=0x0000028edfacdae8, CColorSource * * ppColorSource=0x0000009a90b7bc30) line 866	C++
 	wpfgfx_cor3.dll!CSoftwareRasterizer::FillPath(CSpanSink * pSpanSink=0x0000028eebd7a440, CSpanClipper * pSpanClipper=0x0000009a90b7c278, const CContextState * pContextState=0x0000028edfacdae8, const CShapeBase * pShape=0x0000009a90b7c578, const CMatrix<CoordinateSpace::LocalRenderingHPC,CoordinateSpace::DeviceHPC> * pmatShapeToDevice=0x0000028edfacdc80, CMILBrush * pBrush=0x0000028eebd59ca8, const CMatrix<CoordinateSpace::BaseSampling,CoordinateSpace::DeviceHPC> & matWorldToDevice={...}, IMILEffectList * pIEffect=0x0000000000000000, float rComplementFactor=-1.00000000, const TMilRect_<int,tagRECT,MilPointAndSizeL,RectUniqueness::_CMILSurfaceRect_> * prcComplementBounds=0x0000000000000000) line 680	C++
 	wpfgfx_cor3.dll!CSoftwareRasterizer::FillPathUsingBrushRealizer(CSpanSink * pSpanSink=0x0000028eebd7a440, MilPixelFormat::Enum fmtTarget, DisplayId associatedDisplay, CSpanClipper * pSpanClipper=0x0000009a90b7c278, const CContextState * pContextState=0x0000028edfacdae8, BrushContext * pBrushContext=0x0000000000000000, const CShapeBase * pShape=0x0000009a90b7c578, const CMatrix<CoordinateSpace::LocalRenderingHPC,CoordinateSpace::DeviceHPC> * pmatShapeToDevice=0x0000028edfacdc80, CBrushRealizer * pBrushRealizer=0x0000009a90b7c480, const CMatrix<CoordinateSpace::BaseSampling,CoordinateSpace::DeviceHPC> & matWorldToDevice={...}) line 578	C++
 	wpfgfx_cor3.dll!CSwRenderTargetSurface::DrawPathInternal(CContextState * pContextState=0x0000028edfacdae8, BrushContext * pBrushContext=0x0000000000000000, const CShapeBase * pShape=0x0000009a90b7c578, const CPlainPen * pPen=0x0000000000000000, CBrushRealizer * pStrokeBrush=0x0000000000000000, CBrushRealizer * pFillBrush=0x0000009a90b7c480) line 841	C++
 	wpfgfx_cor3.dll!CDrawingContext::FillShapeWithBitmap(IWGXBitmapSource * pIWGXBitmapSource=0x0000028edfc73650, const CMatrix<CoordinateSpace::BaseSampling,CoordinateSpace::LocalRenderingHPC> * pTextureToLocalTransform=0x0000009a90b7c620, CShapeBase * pShape=0x0000009a90b7c578, IMILEffectList * pEffectList=0x0000000000000000, MilBitmapWrapMode::Enum wrapMode=Extend) line 1617	C++
 	wpfgfx_cor3.dll!CDrawingContext::DrawBitmap(IWGXBitmapSource * pIBitmapSource=0x0000028edfc73650, const MilRectF * prcSource=0x0000009a90b7c728, const MilRectF * prcDest=0x0000009a90b7c6f0, float opacity=1.00000000) line 1479	C++
 	wpfgfx_cor3.dll!CDrawingContext::DrawImage(CMilSlaveResource * pImage=0x0000028eebda14f0, const MilPointAndSizeD * prcDestinationBase, TMilSlaveValue<MilPointAndSizeD,MILCMD_RECTRESOURCE,52> * pDestRectAnimations=0x0000000000000000) line 1345	C++
 	wpfgfx_cor3.dll!CMilSlaveRenderData::Draw(IDrawingContext * pIDC=0x0000028edfacda30) line 766	C++
 	wpfgfx_cor3.dll!CMilVisual::RenderContent(CDrawingContext * pDrawingContext=0x0000028edfacda30) line 1204	C++
 	wpfgfx_cor3.dll!CDrawingContext::PreSubgraph(int * pfVisitChildren=0x0000009a90b7ca60) line 5076	C++
 	wpfgfx_cor3.dll!CGraphIterator::Walk(IGraphNode * pRoot, IGraphIteratorSink * pSink=0x0000028edfacda60) line 308	C++
 	wpfgfx_cor3.dll!CDrawingContext::DrawVisualTree(CMilVisual * pRoot=0x0000028eebd71be0, const _D3DCOLORVALUE * pClearColor=0x0000000000000000, const TMilRect_<float,MilRectF,MilPointAndSizeF,RectUniqueness::NotNeeded> & dirtyRect, bool fDrawingIntoVisualBrush=false) line 4424	C++
 	wpfgfx_cor3.dll!CDrawingContext::Render(CMilVisual * pRoot=0x0000028eebd71be0, IMILRenderTarget * pIRenderTarget, const _D3DCOLORVALUE * pClearColor=0x0000000000000000, const TMilRect_<float,MilRectF,MilPointAndSizeF,RectUniqueness::NotNeeded> & rcSurfaceBounds={...}, int fFullRender=1, unsigned int uNumInvalidTargetRegions=0, const MilRectF * rgInvalidTargetRegions=0x0000000000000000, bool fCanAccelerateScroll=false, int * pfNeedsFullPresent=0x0000009a90b7ccc8) line 6007	C++
 	wpfgfx_cor3.dll!CSlaveGenericRenderTarget::Render(bool * pfPresentNeeded=0x0000000000000000) line 88	C++
 	wpfgfx_cor3.dll!CRenderTargetManager::Render(bool * pfPresentNeeded=0x0000009a90b7ce60) line 339	C++
 	wpfgfx_cor3.dll!CComposition::Render(bool * pfPresentNeeded=0x0000009a90b7ce60) line 859	C++
 	wpfgfx_cor3.dll!CComposition::ProcessComposition(bool * pfPresentNeeded=0x0000009a90b7ce60) line 712	C++
 	wpfgfx_cor3.dll!CComposition::Compose(bool * pfPresentNeeded=0x0000009a90b7ce88) line 805	C++
 	wpfgfx_cor3.dll!CConnectionContext::PresentAllPartitions() line 506	C++
 	wpfgfx_cor3.dll!CMilConnection::PresentAllPartitions() line 485	C++
 	wpfgfx_cor3.dll!WgxConnection_SameThreadPresent(HMIL_CONNECTION__ * hConnection) line 135	C++
 	PresentationCore.dll!System.Windows.Media.Composition.DUCE.UnsafeNativeMethods.WgxConnection_SameThreadPresent(System.IntPtr pConnection)	
```

从以上代码可以看到是 WPF 的渲染线程的进入了 `CWGXBitmapLockState::LockRead` 自旋锁等待 WriteableBitmap 解锁

此问题已经报告给官方（嗯，其实就是我），且我已经修复：[https://github.com/dotnet/wpf/issues/8134](https://github.com/dotnet/wpf/issues/8134)

## 复现步骤

1. 创建一个 WriteableBitmap 对象且添加到 Image 控件里面
2. 调用 WriteableBitmap 的 WritePixels 方法，传入错误的 stride 数值
3. 立即调用渲染，如使用 RenderTargetBitmap 对 Image 控件进行截图

最小复现 Demo 代码：[https://github.com/lindexi/lindexi_gd/tree/20395cade5a79ed40bdd03acf73320994966c691/HawacearkecallLalarnowhallna](https://github.com/lindexi/lindexi_gd/tree/20395cade5a79ed40bdd03acf73320994966c691/HawacearkecallLalarnowhallna)

## 原因

这是因为在 WriteableBitmap 的代码实现没有关注到锁的安全性，导致了传入错误的 stride 数值，抛了异常，让 Unlock 没有被调用，从而让锁没有释放。于是渲染线程等待多久，都等不到锁的释放

问题的代码如下：

```csharp
        private void WritePixelsImpl(
            Int32Rect sourceRect,
            IntPtr    sourceBuffer,
            int       sourceBufferSize,
            int       sourceBufferStride,
            int       destinationX,
            int       destinationY,
            bool      backwardsCompat
            )
        {
            // 忽略其他代码
            
                unsafe
                {
                    // 忽略其他代码

                    Lock();

                    MILUtilities.MILCopyPixelBuffer(
                        pDest,
                        outputBufferSize,
                        (uint) _backBufferStride.Value,
                        destBufferBitOffset,
                        pSource,
                        inputBufferSize,
                        (uint) sourceBufferStride,
                        sourceBufferBitOffset,
                        (uint) sourceRect.Height,
                        copyWidthInBits);

                    AddDirtyRect(destinationRect);
                    Unlock();
                }
        }
```

当传入错误的 stride 数值，将会导致 `MILUtilities.MILCopyPixelBuffer` 抛出异常，从而导致 Unlock 函数没有被正确调用

渲染线程需要等待 WriteableBitmap 的锁释放，但是由于 WriteableBitmap 的 Unlock 因为异常而没有被正确调用，因此渲染线程进入无限等待

