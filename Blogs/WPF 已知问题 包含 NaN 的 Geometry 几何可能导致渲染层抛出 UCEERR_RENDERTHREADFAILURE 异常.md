本文记录一个 WPF 已知问题，当传入到渲染的 Geometry 几何里面包含了 NaN 数值，将可能让应用程序收到从渲染层抛上来的 UCEERR_RENDERTHREADFAILURE 异常，且此异常缺乏必要信息，比较难定位到具体错误逻辑

<!--more-->


<!-- CreateTime:2023/12/12 20:28:53 -->

<!-- 发布 -->
<!-- 博客 -->

此问题是小伙伴报告给我的，详细请看 [https://github.com/dotnet/wpf/issues/7421](https://github.com/dotnet/wpf/issues/7421)

此问题仅仅只发生在比较特殊的情况，其他情况下都能比较及时收到异常或者是 WPF 层会吞没异常，忽略 Geometry 几何的行为，就当成此 Geometry 几何不存在。由于在所有逻辑里面提前判断参数的合法将降低通用逻辑性能，因此我决定了此问题不做修复，仅仅只是调查问题的原因

我将此问题的原因记录到问题的 Issues 上，同步也写了本文内容

复现步骤稍微复杂，复现代码如下

```csharp
using System;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;
using System.Windows.Shapes;

namespace NaN_Crash
{
  internal class Class1 : Control
  {
    protected override void OnRender(DrawingContext dc)
    {
      var rc = new Rect(0, 0, ActualWidth, ActualHeight);

      // bad rc
      rc = new Rect(0, double.NaN, 36, 144);
      Geometry.DrawRoundedRect(dc, rc, 18);
    }
  }

  internal static class Geometry
  {
    static void Curve(PathFigure pf, double x, double y, double cr, bool clockwise)
    {
      pf.Segments.Add(
        new ArcSegment(
          new Point(x, y),
          new Size(cr, cr),
          0,
          false,
          clockwise ? SweepDirection.Clockwise : SweepDirection.Counterclockwise,
          true));
    }

    static void Line(PathFigure pf, double x, double y)
    {
      pf.Segments.Add(new LineSegment(new Point(x, y), true));
    }

    public static PathGeometry GetRoundedRect(Rect rect, double cr)
    {
      PathGeometry pg = new PathGeometry();
      PathFigure pf = new PathFigure();
      pf.StartPoint = new Point(rect.Left + cr, rect.Top);
      Line(pf, rect.Right - cr, rect.Top);
      Curve(pf, rect.Right, rect.Top + cr, cr, true);
      Line(pf, rect.Right, rect.Bottom - cr);
      Curve(pf, rect.Right - cr, rect.Bottom, cr, true);
      Line(pf, rect.Left + cr, rect.Bottom);
      Curve(pf, rect.Left, rect.Bottom - cr, cr, true);
      Line(pf, rect.Left, rect.Top + cr);
      Curve(pf, rect.Left + cr, rect.Top, cr, true);
      pf.IsClosed = true;
      pf.Freeze();
      pg.Figures.Add(pf);
      pg.Freeze();
      return pg;
    }

    public struct InnerGeometryInfo
    {
      public System.Windows.Media.Geometry Geo { get; set; }
      public double InnerBr { get; set; }
      public Rect InnerRect { get; set; }
    }

    public static InnerGeometryInfo GetInnerGeo(Rect rect, double br)
    {
      var th = new Thickness(1, 1, 1, 1);
      var innerRect = Adjust(rect, th);
      double innerBr = Math.Max(0, br - 1.0);

      return new InnerGeometryInfo()
      {
        Geo = GetRoundedRect(innerRect, innerBr),
        InnerBr = innerBr,
        InnerRect = innerRect
      };
    }

    public static void DrawRoundedRect(DrawingContext dc, Rect rect, double br)
    {
      var geo1 = GetRoundedRect(rect, br);
      var innerInfo = GetInnerGeo(rect, br);
      var innerBr = innerInfo.InnerBr;
      var innerRect = Adjust(rect, new Thickness(1.0));
      var geo = new CombinedGeometry(GeometryCombineMode.Exclude, geo1,innerInfo.Geo);
      dc.PushClip(geo);
      dc.Pop();
    }

    public static Rect SafeRect(double x, double y, double w, double h)
    {
      return new Rect(x, y, Math.Max(0, w), Math.Max(0, h));
    }

    public static Rect Adjust(Rect rc, Thickness? th)
    {
      if (th.HasValue)
      {
        return SafeRect(rc.Left + th.Value.Left, rc.Top + th.Value.Top, rc.Width - th.Value.Left - th.Value.Right, rc.Height - th.Value.Top - th.Value.Bottom);
      }
      return rc;
    }
  }
}
```

将此 Class1 放入到 MainWindow.xaml 里面，运行项目即可复现问题

可以看到抛出的异常如下

```
An unhandled exception of type 'System.Runtime.InteropServices.COMException' occurred in PresentationCore.dll
UCEERR_RENDERTHREADFAILURE (0x88980406)

 	PresentationCore.dll!System.Windows.Media.Composition.DUCE.Channel.SyncFlush()	Unknown
 	PresentationCore.dll!System.Windows.Interop.HwndTarget.UpdateWindowSettings(bool enableRenderTarget, System.Windows.Media.Composition.DUCE.ChannelSet? channelSet)	Unknown
 	PresentationCore.dll!System.Windows.Interop.HwndTarget.UpdateWindowPos(System.IntPtr lParam)	Unknown
 	PresentationCore.dll!System.Windows.Interop.HwndTarget.HandleMessage(MS.Internal.Interop.WindowMessage msg, System.IntPtr wparam, System.IntPtr lparam)	Unknown
 	PresentationCore.dll!System.Windows.Interop.HwndSource.HwndTargetFilterMessage(System.IntPtr hwnd, int msg, System.IntPtr wParam, System.IntPtr lParam, ref bool handled)	Unknown
 	WindowsBase.dll!MS.Win32.HwndWrapper.WndProc(System.IntPtr hwnd, int msg, System.IntPtr wParam, System.IntPtr lParam, ref bool handled)	Unknown
 	WindowsBase.dll!MS.Win32.HwndSubclass.DispatcherCallbackOperation(object o)	Unknown
 	WindowsBase.dll!System.Windows.Threading.ExceptionWrapper.InternalRealCall(System.Delegate callback, object args, int numArgs)	Unknown
 	WindowsBase.dll!System.Windows.Threading.ExceptionWrapper.TryCatchWhen(object source, System.Delegate callback, object args, int numArgs, System.Delegate catchHandler)	Unknown
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.LegacyInvokeImpl(System.Windows.Threading.DispatcherPriority priority, System.TimeSpan timeout, System.Delegate method, object args, int numArgs)	Unknown
 	WindowsBase.dll!MS.Win32.HwndSubclass.SubclassWndProc(System.IntPtr hwnd, int msg, System.IntPtr wParam, System.IntPtr lParam)	Unknown
 	[Native to Managed Transition]	
 	[Managed to Native Transition]	
 	PresentationFramework.dll!System.Windows.Window.Flush()	Unknown
 	PresentationFramework.dll!System.Windows.Window.HwndStyleManager.System.IDisposable.Dispose()	Unknown
 	PresentationFramework.dll!System.Windows.Window.CreateSourceWindow(bool duringShow)	Unknown
 	PresentationFramework.dll!System.Windows.Window.CreateSourceWindowDuringShow()	Unknown
 	PresentationFramework.dll!System.Windows.Window.SafeCreateWindowDuringShow()	Unknown
 	PresentationFramework.dll!System.Windows.Window.ShowHelper(object booleanBox)	Unknown
 	PresentationFramework.dll!System.Windows.Window.Show()	Unknown
 	Microsoft.VisualStudio.DesignTools.WpfTap.dll!Microsoft.VisualStudio.DesignTools.WpfTap.WpfVisualTreeService.Adorners.AdornerWindow.SafeShow()	Unknown
 	Microsoft.VisualStudio.DesignTools.WpfTap.dll!Microsoft.VisualStudio.DesignTools.WpfTap.WpfVisualTreeService.Adorners.AdornerWindow.UpdatePlacement()	Unknown
 	Microsoft.VisualStudio.DesignTools.WpfTap.dll!Microsoft.VisualStudio.DesignTools.WpfTap.Utility.DispatcherUtility.SafeInvokeAsync.AnonymousMethod__0()	Unknown
 	WindowsBase.dll!System.Windows.Threading.DispatcherOperation.InvokeDelegateCore()	Unknown
 	WindowsBase.dll!System.Windows.Threading.DispatcherOperation.InvokeImpl()	Unknown
 	WindowsBase.dll!System.Windows.Threading.DispatcherOperation.InvokeInSecurityContext(object state)	Unknown
 	WindowsBase.dll!MS.Internal.CulturePreservingExecutionContext.CallbackWrapper(object obj)	Unknown
 	System.Private.CoreLib.dll!System.Threading.ExecutionContext.RunInternal(System.Threading.ExecutionContext executionContext, System.Threading.ContextCallback callback, object state)	Unknown
 	System.Private.CoreLib.dll!System.Threading.ExecutionContext.Run(System.Threading.ExecutionContext executionContext, System.Threading.ContextCallback callback, object state)	Unknown
 	WindowsBase.dll!MS.Internal.CulturePreservingExecutionContext.Run(MS.Internal.CulturePreservingExecutionContext executionContext, System.Threading.ContextCallback callback, object state)	Unknown
 	WindowsBase.dll!System.Windows.Threading.DispatcherOperation.Invoke()	Unknown
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.ProcessQueue()	Unknown
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.WndProcHook(System.IntPtr hwnd, int msg, System.IntPtr wParam, System.IntPtr lParam, ref bool handled)	Unknown
 	WindowsBase.dll!MS.Win32.HwndWrapper.WndProc(System.IntPtr hwnd, int msg, System.IntPtr wParam, System.IntPtr lParam, ref bool handled)	Unknown
 	WindowsBase.dll!MS.Win32.HwndSubclass.DispatcherCallbackOperation(object o)	Unknown
 	WindowsBase.dll!System.Windows.Threading.ExceptionWrapper.InternalRealCall(System.Delegate callback, object args, int numArgs)	Unknown
 	WindowsBase.dll!System.Windows.Threading.ExceptionWrapper.TryCatchWhen(object source, System.Delegate callback, object args, int numArgs, System.Delegate catchHandler)	Unknown
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.LegacyInvokeImpl(System.Windows.Threading.DispatcherPriority priority, System.TimeSpan timeout, System.Delegate method, object args, int numArgs)	Unknown
 	WindowsBase.dll!MS.Win32.HwndSubclass.SubclassWndProc(System.IntPtr hwnd, int msg, System.IntPtr wParam, System.IntPtr lParam)	Unknown
 	[Native to Managed Transition]	
 	[Managed to Native Transition]	
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.PushFrameImpl(System.Windows.Threading.DispatcherFrame frame)	Unknown
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.PushFrame(System.Windows.Threading.DispatcherFrame frame)	Unknown
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.Run()	Unknown
 	PresentationFramework.dll!System.Windows.Application.RunDispatcher(object ignore)	Unknown
 	PresentationFramework.dll!System.Windows.Application.RunInternal(System.Windows.Window window)	Unknown
 	PresentationFramework.dll!System.Windows.Application.Run()	Unknown
 	NaN-Crash.dll!NaN_Crash.App.Main()	Unknown
```

这个异常存在的问题是缺乏足够的提示信息，导致难以定位具体问题。尽管可以在 CombinedGeometry 进行拦截，且当前的 CombinedGeometry 已经做了一定的拦截措施，但如果想要进行更进一步的拦截本问题，则需要修改到通用逻辑。修改通用逻辑将会降低通用逻辑性能。由于此问题比较难以复现，即使出现问题了，慢慢调试也能找到坑。于是我就决定此问题不修复，但是我将会记录下来出现此问题的原因

我通过调试 WPF 框架，调试 WPF 的 GFX 层调试到问题的原因。核心原因是在 `CShapeBase::GetCachedBounds` 对 Geometry 的 Bounds 进行校验失败，因此导致此方法返回了 `WGXERR_BADNUMBER` 错误。此 `WGXERR_BADNUMBER` 错误对应 0x8898000a 错误码，也就是在调试下可以看到 MIL 层的如下输出

```
MIL FAILURE: Unexpected HRESULT 0x8898000a in caller: The render thread failed unexpectedly.
```

根据的是在 `CShapeBase::GetCachedBounds` 的 `if (!rect.HasValidValues())` 判断不通过，详细请看[以下代码](https://github.com/dotnet/wpf/blob/3774bcb6001b7e6b0b84223a406d6ed033a4c02c/src/Microsoft.DotNet.Wpf/src/WpfGfx/core/geometry/shapebase.cpp#L1524-L1546)

```csharp
 HRESULT 
 CShapeBase::GetCachedBounds( 
     __out_ecount(1) CMilRectF &rect) const     // The data's cached bounds 
 { 
     HRESULT hr = S_OK; 
  
     if (!GetCachedBoundsCore(rect)) 
     { 
         // Compute the bounds and update the cache 
         CMilRectF box; 
         IFC(GetFillBounds(OUT box, false /* including non-fillable figures */)); 
         SetCachedBounds(box); 
         rect = box; 
     } 
  
     if (!rect.HasValidValues()) 
     { 
         IFC(WGXERR_BADNUMBER); 
     } 
  
 Cleanup: 
     RRETURN(hr); 
 } 
```

为什么以上代码的 HasValidValues 方法将无法通过？原因是这个 rect 包含了 NaN 的内容。那为什么这个 rect 包含了 NaN 的内容，是在哪一层投毒的

如上面代码，在 `CShapeBase::GetFillBounds` 方法里面就获取到了不合法的 box 值。接下来我将细细告诉大家这个调用链是如何一步步使用错误的上层业务代码传入的 Geometry 数据，获取到错误的参数。在 `CShapeBase::GetFillBounds` 方法里将会调用到 `CShapeBase::UpdateBounds` 方法进行更新 Bounds 范围，而 UpdateBounds 方法将返回包含 NaN 的 Bounds 范围

在 WPF 底层里面，许多代码都使用 Shape 来表示 Geometry 几何

以下是 `CShapeBase::UpdateBounds` 方法的代码

```csharp
 CShapeBase::UpdateBounds( 
     __inout_ecount(1) CBounds &bounds, 
         // Bounds, updated here 
     __in bool fFillOnly, 
         // Skip non-fillable figures if true 
     __in_ecount_opt(1) const CMILMatrix *pMatrix 
         // Transformation (NULL OK) 
     ) const 
 { 
     HRESULT hr = S_OK; 
  
     for (UINT i = 0;  i < GetFigureCount();  i++) 
     { 
         const IFigureData & figure = GetFigure(i); 
         if (!fFillOnly || figure.IsFillable()) 
         { 
             IFC(CFigureBase(figure).UpdateBounds(OUT bounds, pMatrix)); 
         } 
     } 
  
 Cleanup: 
     RRETURN(hr); 
 } 
```

在 `CShapeBase::UpdateBounds` 里面，将会使用每个 Figure 进行更新 Bounds 范围内容，代码如下

```csharp
HRESULT
CFigureBase::UpdateBounds(
    __inout_ecount(1) CBounds &bounds,
        // In/out: Bounds, updated here
    __in_ecount_opt(1) const CMILMatrix *pMatrix
        // In: Transformation (NULL OK)
    ) const
{
    HRESULT hr = S_OK;

    if (!m_refData.IsEmpty())
    {
        if (pMatrix == NULL &&
            m_refData.IsAxisAlignedRectangle()
            )
        {
            MilPoint2F ptCorners[2];
            m_refData.GetRectangleCorners(ptCorners);

            bounds.UpdateWithPoint(ptCorners[0]);
            bounds.UpdateWithPoint(ptCorners[1]);
        }
        else
        {
            CBoundsTask task(bounds, m_refData.GetStartPoint(), pMatrix);
            IFC(task.TraverseForward(m_refData));
        }
    }

Cleanup:
    RRETURN(hr);
}
```

在本文的例子代码里面，将会进入 `IFC(task.TraverseForward(m_refData))` 分支

在 `task.TraverseForward(m_refData)` 里面，将会获取每段的内容更新 Bounds 范围

```csharp
HRESULT
CFigureTask::TraverseForward(
    __in_ecount(1) const IFigureData &figure
        // The traversed figure
    )
{
    HRESULT hr = S_OK;
    const MilPoint2F *pPt;
    BYTE bType;
    m_fAborted = false;

    if (!figure.SetToFirstSegment())
        goto Cleanup;
    
    do 
    {
        figure.GetCurrentSegment(bType, pPt);
        if (MilCoreSeg::TypeLine == bType)
        {          
            IFC(DoLine(*pPt));
        }
        else
        {
            Assert(MilCoreSeg::TypeBezier == bType);
            IFC(DoBezier(pPt));
        }
    } 
    while (!m_fAborted  &&  figure.SetToNextSegment());

Cleanup:
    RRETURN(hr);
}
```

本文这里将进入 `DoLine(*pPt)` 分支，在进入 DoLine 时拿到的 `pPt` 已经是错误的值了。在 `figure.GetCurrentSegment` 里面将会调用到 GetStartPoint 方法，这就是本文代码例子里面最开始投毒的地方，后续也会在 `DoBezier` 里面继续投毒。先看一下最开始投毒的逻辑，在 PathGeometryWrapper.cpp 定义的 GetStartPoint 方法，返回了本文使用的代码里面传入的包含 NaN 的点的值，如[以下代码](https://github.com/dotnet/wpf/blob/3774bcb6001b7e6b0b84223a406d6ed033a4c02c/src/Microsoft.DotNet.Wpf/src/WpfGfx/core/resources/PathGeometryWrapper.cpp#L827-L839)，拿到的 `m_pFigure->StartPoint` 的值是不符合预期的 `{X=18.000000000000000 Y=-nan(ind) }` 值

```csharp
 const MilPoint2F &PathFigureData::GetStartPoint() const 
 { 
     Assert(m_pFigure != NULL); 
  
     m_ptStartPoint = ConvertToSingle(m_pFigure->StartPoint); 
  
     if (m_pMatrix != NULL) 
     { 
         TransformPoint(*m_pMatrix, m_ptStartPoint); 
     } 
  
     return m_ptStartPoint; 
 } 
```

这也就导致 `CFigureTask::TraverseForward` 通过 `m_oBounds.UpdateWithPoint(m_ptCurrent)` 这行代码从 StartPoint 里面将 NaN 带入到 bounds 里面

在 UpdateWithPoint 方法里面将会调用到 `CBounds::UpdateNaN` 设置 NaN 到 `m_fEncounteredNaN` 字段，如以下代码

```csharp
 void UpdateNaN(GpPointR pt) 
 { 
     m_fEncounteredNaN = m_fEncounteredNaN || _isnan(pt.X) || _isnan(pt.Y); 
 } 
```

于是这个 Bounds 范围给到 rect 变量，将让 HasValidValues 方法不通过，返回异常。进入到以上的 UpdateNaN 方法的堆栈如下

```
 	wpfgfx_cor3.dll!CBounds::UpdateNaN(GpPointR pt={...}) Line 526	C++
 	wpfgfx_cor3.dll!CBounds::UpdateWithPoint(const GpPointR & pt={...}) Line 761	C++
 	wpfgfx_cor3.dll!CBoundsTask::DoLineNoHRESULT(const MilPoint2F & ptEnd={...}) Line 58	C++
 	wpfgfx_cor3.dll!CBoundsTask::DoLine(const MilPoint2F & ptEnd={...}) Line 211	C++
 	wpfgfx_cor3.dll!CFigureTask::TraverseForward(const IFigureData & figure={...}) Line 531	C++
 	wpfgfx_cor3.dll!CFigureBase::UpdateBounds(CBounds & bounds={...}, const CMILMatrix * pMatrix=0x00000000) Line 62	C++
 	wpfgfx_cor3.dll!CShapeBase::UpdateBounds(CBounds & bounds={...}, bool fFillOnly=false, const CMILMatrix * pMatrix=0x00000000) Line 1440	C++
 	wpfgfx_cor3.dll!CShapeBase::GetFillBounds(TMilRect_<float,MilRectF,MilPointAndSizeF,RectUniqueness::NotNeeded> & rect={...}, bool fFillOnly=false, const CMILMatrix * pMatrix=0x00000000) Line 1407	C++
 	wpfgfx_cor3.dll!CShapeBase::GetCachedBounds(TMilRect_<float,MilRectF,MilPointAndSizeF,RectUniqueness::NotNeeded> & rect={...}) Line 1534	C++
 	wpfgfx_cor3.dll!CShapeBase::GetTightBounds(TMilRect_<float,MilRectF,MilPointAndSizeF,RectUniqueness::NotNeeded> & rect={...}, const CPlainPen * pPen=0x00000000, const CMILMatrix * pMatrix=0x00000000, double rTolerance=0.0000000000000000, bool fRelative=false, bool fSkipHollows=true) Line 1356	C++
 	wpfgfx_cor3.dll!CShapeBase::Combine(const CShapeBase * pFirst=0x0e4b4510, const CShapeBase * pSecond=0x0e4b4ab0, MilCombineMode::Enum eOperation=Exclude, bool fRetrieveCurves=true, CShape * pResult=0x129b2cec, const CMILMatrix * pFirstTransform=0x00000000, const CMILMatrix * pSecondTransform=0x00000000, double rTolerance=0.25000000000000000, bool fRelative=false) Line 700	C++
 	wpfgfx_cor3.dll!CMilCombinedGeometryDuce::GetShapeDataCore(CShapeBase * * ppShapeData=0x129b2cdc) Line 106	C++
 	wpfgfx_cor3.dll!CMilGeometryDuce::GetShapeData(CShapeBase * * ppShapeData=0x0f37f454) Line 98	C++
 	wpfgfx_cor3.dll!CDrawingContext::PushEffects(const double & rOpacity=1.0000000000000000, CMilGeometryDuce * pGeometryMask=0x129b2cc8, CMilBrushDuce * pOpacityMaskBrush=0x00000000, CMilEffectDuce * pEffect=0x00000000, const CRectF<CoordinateSpace::LocalRenderingHPC> * pSurfaceBoundsLocalSpace=0x00000000) Line 2689	C++
 	wpfgfx_cor3.dll!CDrawingContext::PushClip(CMilGeometryDuce * pClipGeometry=0x129b2cc8) Line 2076	C++
 	wpfgfx_cor3.dll!CMilSlaveRenderData::Draw(IDrawingContext * pIDC=0x129f9bc8) Line 1025	C++
 	wpfgfx_cor3.dll!CContentBounder::GetContentBounds(CMilSlaveResource * pContent=0x0e5d7ed0, TMilRect_<float,MilRectF,MilPointAndSizeF,RectUniqueness::NotNeeded> * prcBounds=0x129f7f90) Line 196	C++
 	wpfgfx_cor3.dll!CMilVisual::GetContentBounds(CContentBounder * pContentBounder=0x12883d58, TMilRect_<float,MilRectF,MilPointAndSizeF,RectUniqueness::NotNeeded> * prcBounds=0x129f7f90) Line 1176	C++
 	wpfgfx_cor3.dll!CPreComputeContext::PreSubgraph(int * pfVisitChildren=0x0f37f828) Line 319	C++
 	wpfgfx_cor3.dll!CGraphIterator::Walk(IGraphNode * pRoot=0x0e6d96d8, IGraphIteratorSink * pSink=0x129b6208) Line 308	C++
 	wpfgfx_cor3.dll!CPreComputeContext::PreCompute(CMilVisual * pRoot=0x0e6d96d8, const TMilRect_<float,MilRectF,MilPointAndSizeF,RectUniqueness::NotNeeded> * prcSurfaceBounds=0x0f37f9f4, unsigned int uNumInvalidTargetRegions=0, const MilRectF * rgInvalidTargetRegions=0x0501e750, float allowedDirtyRegionOverhead=50000.0000, MilBitmapInterpolationMode::Enum defaultInterpolationMode=Linear, ScrollAreaStruct * pScrollArea=0x00000000, int fDisableDirtyRegionOptimization=1) Line 136	C++
 	wpfgfx_cor3.dll!CDrawingContext::PreCompute(CMilVisual * pRoot=0x0e6d96d8, const TMilRect_<float,MilRectF,MilPointAndSizeF,RectUniqueness::NotNeeded> * prcSurfaceBounds=0x0f37f9f4, unsigned int uNumInvalidTargetRegions=0, const MilRectF * rgInvalidTargetRegions=0x0501e750, float allowedDirtyRegionOverhead=50000.0000, int fFullRender=1, ScrollAreaStruct * pScrollArea=0x00000000) Line 5681	C++
 	wpfgfx_cor3.dll!CDrawingContext::Render(CMilVisual * pRoot=0x0e6d96d8, IMILRenderTarget * pIRenderTarget=0x0501e64c, const _D3DCOLORVALUE * pClearColor=0x0d122818, const TMilRect_<float,MilRectF,MilPointAndSizeF,RectUniqueness::NotNeeded> & rcSurfaceBounds={...}, int fFullRender=1, unsigned int uNumInvalidTargetRegions=0, const MilRectF * rgInvalidTargetRegions=0x0501e750, bool fCanAccelerateScroll=false, int * pfNeedsFullPresent=0x0f37fa08) Line 5944	C++
 	wpfgfx_cor3.dll!CSlaveHWndRenderTarget::Render(bool * pfNeedsPresent=0x0f37fa4f) Line 185	C++
 	wpfgfx_cor3.dll!CRenderTargetManager::Render(bool * pfPresentNeeded=0x0f37fb23) Line 339	C++
 	wpfgfx_cor3.dll!CComposition::Render(bool * pfPresentNeeded=0x0f37fb23) Line 859	C++
 	wpfgfx_cor3.dll!CComposition::ProcessComposition(bool * pfPresentNeeded=0x0f37fb23) Line 712	C++
 	wpfgfx_cor3.dll!CComposition::Compose(bool * pfPresentNeeded=0x0f37fb3f) Line 805	C++
 	wpfgfx_cor3.dll!CPartitionThread::RenderPartition(Partition * pPartition=0x0d0b8ac0) Line 134	C++
 	wpfgfx_cor3.dll!CPartitionThread::Run() Line 235	C++
 	wpfgfx_cor3.dll!CPartitionThread::ThreadMain(void * pv=0x0b99b5d0) Line 46	C++
 	kernel32.dll!@BaseThreadInitThunk@12()
 	ntdll.dll!___RtlUserThreadStart@8()
 	ntdll.dll!__RtlUserThreadStart@8()
```

其他的投毒逻辑也差不多，只需要在 figure 拿到的点包含 NaN 即可更新到 Bounds 导致拿到不符合预期的内容

那为什么上层收到的是 RENDERTHREADFAILURE 异常而不是这里的 WGXERR_BADNUMBER 错误码？这是因为在 `wpfgfx_cor3.dll!CPartitionThread::RenderPartition(Partition * pPartition=0x0d0b8ac0) Line 134	C++ ` 这行代码里面，也就是以上堆栈的倒数第六行，将会统一处理异常

在 `CPartitionThread::RenderPartition` 处于 GFX 的上层了，在这里将会进入 `CPartitionManager::ZombifyPartitionAndCompleteProcessing` 方法进行处理异常，调用堆栈如下

```
 	wpfgfx_cor3.dll!CPartitionManager::ZombifyPartitionAndCompleteProcessing(Partition * pPartition=0x0ce18aa0, HRESULT hrFailureCode=0x8898000a) Line 540	C++
 	wpfgfx_cor3.dll!CPartitionThread::RenderPartition(Partition * pPartition=0x0ce18aa0) Line 144	C++
 	wpfgfx_cor3.dll!CPartitionThread::Run() Line 235	C++
 	wpfgfx_cor3.dll!CPartitionThread::ThreadMain(void * pv=0x0bdab2b8) Line 46	C++
 	kernel32.dll!@BaseThreadInitThunk@12()
 	ntdll.dll!___RtlUserThreadStart@8()
 	ntdll.dll!__RtlUserThreadStart@8()
```

在 `CPartitionManager::ZombifyPartitionAndCompleteProcessing` 里面，只是判断 hrFailureCode 是不是 D3DERR_OUTOFVIDEOMEMORY 这个 D3D 设备的内存不足或者是其他内存不足问题，如果不是这两个问题，则替换通用渲染异常

```csharp
void
CPartitionManager::ZombifyPartitionAndCompleteProcessing(
    __in_ecount(1) Partition *pPartition,
    HRESULT hrFailureCode
    )
{
    // This routine should be called from worker thread.
    Assert(CurrentThreadIsWorkerThread());

    //
    // We only return OOM or OOVM back to through the back channel.
    // For all other failures, just return a generic render thread failure
    // failure since we don't want to give out the details
    // for security reasons.
    //
    if (hrFailureCode == D3DERR_OUTOFVIDEOMEMORY)
    {
        pPartition->m_hrZombieNotificationFailureReason = D3DERR_OUTOFVIDEOMEMORY;
    }
    else if (IsOOM(hrFailureCode))
    {
        pPartition->m_hrZombieNotificationFailureReason = E_OUTOFMEMORY;
    }
    else
    {
        //
        // Note: This failure is not as a result in a problem in this code, something
        //       happened in the render thread that resulted in us zombifying the 
        //       partition. Look at stack backtrace capture to determine the root
        //       cause of the failure.
        // 
        MilUnexpectedError(hrFailureCode, TEXT("The render thread failed unexpectedly."));

        pPartition->m_hrZombieNotificationFailureReason = WGXERR_UCE_RENDERTHREADFAILURE;
    }

    SetPartitionState(
        pPartition,
        PartitionZombifyClearFlags,
        PartitionZombifySetFlags
        );
}
```

至于为什么要替换为通用异常，如以上注释 `For all other failures, just return a generic render thread failure since we don't want to give out the details for security reasons.` 这是为了安全，虽然我也不知道为什么将具体的错误信息返回上层就不安全了

通用处理返回的 `WGXERR_UCE_RENDERTHREADFAILURE` 错误，将会在 C# 层抛出 `UCEERR_RENDERTHREADFAILURE (0x88980406)` 异常

这看起来在 GFX 层的处理也是合理的

在 WPF 的底层里面，已经对此做了许多兼容逻辑，比如另一个会进入到 `WGXERR_BADNUMBER` 错误码的地方是在 `PathGeometry.InternalCombine` 堆栈里面，如以下代码

```csharp
 if (hr == (int)MILErrors.WGXERR_BADNUMBER) 
 { 
     // When we encounter NaNs in the renderer, we absorb the error and draw 
     // nothing. To be consistent, we return an empty geometry. 
     resultGeometry = new PathGeometry(); 
 } 
```

也就是说在 PathGeometry 底层已经判断了此情况，如果有 NaN 的情况就替换为空的 PathGeometry 对象。在本文的例子代码里，其实也进入了这个分支，调用堆栈如下

```
 	wpfgfx_cor3.dll!CShapeBase::GetCachedBounds(TMilRect_<float,MilRectF,MilPointAndSizeF,RectUniqueness::NotNeeded> & rect={...}) Line 1541	C++
 	wpfgfx_cor3.dll!CShapeBase::GetTightBounds(TMilRect_<float,MilRectF,MilPointAndSizeF,RectUniqueness::NotNeeded> & rect={...}, const CPlainPen * pPen=0x00000000, const CMILMatrix * pMatrix=0x00000000, double rTolerance=0.0000000000000000, bool fRelative=false, bool fSkipHollows=true) Line 1356	C++
 	wpfgfx_cor3.dll!CShapeBase::Combine(const CShapeBase * pFirst=0x006bae44, const CShapeBase * pSecond=0x006baf00, MilCombineMode::Enum eOperation=Exclude, bool fRetrieveCurves=true, CShape * pResult=0x006bafbc, const CMILMatrix * pFirstTransform=0x00000000, const CMILMatrix * pSecondTransform=0x00000000, double rTolerance=0.25000000000000000, bool fRelative=false) Line 700	C++
 	wpfgfx_cor3.dll!MilUtility_PathGeometryCombine(_MilMatrix3x2D * pGeometryMatrix=0x006bb424, _MilMatrix3x2D * pMatrix1=0x006bb3f4, MilFillMode::Enum fillRule1=Alternate, MilPathGeometry * pPathData1=0x054c66c4, unsigned int nSize1=472, _MilMatrix3x2D * pMatrix2=0x006bb3bc, MilFillMode::Enum fillRule2=Alternate, MilPathGeometry * pPathData2=0x054c694c, unsigned int nSize2=472, double rTolerance=0.25000000000000000, bool fRelative=false, void(__stdcall*)(int, int, MilPoint2F *, unsigned int, unsigned char *, unsigned int) fnAddFigureToList=0x04b7377a, MilCombineMode::Enum combineMode=Exclude, MilFillMode::Enum * pOutFillRule=0x006bb3a0) Line 386	C++
 	PresentationCore.dll!System.Windows.Media.PathGeometry.InternalCombine(System.Windows.Media.Geometry geometry1 = {System.Windows.Media.PathGeometry}, System.Windows.Media.Geometry geometry2 = {System.Windows.Media.PathGeometry}, System.Windows.Media.GeometryCombineMode mode = Exclude, System.Windows.Media.Transform transform = {System.Windows.Media.MatrixTransform}, double tolerance = 0.25, System.Windows.Media.ToleranceType type = Absolute) Line 661	C#
 	PresentationCore.dll!System.Windows.Media.Geometry.Combine(System.Windows.Media.Geometry geometry1 = {System.Windows.Media.PathGeometry}, System.Windows.Media.Geometry geometry2 = {System.Windows.Media.PathGeometry}, System.Windows.Media.GeometryCombineMode mode = Exclude, System.Windows.Media.Transform transform = {System.Windows.Media.MatrixTransform}) Line 814	C#
 	PresentationCore.dll!System.Windows.Media.CombinedGeometry.GetAsPathGeometry() Line 275	C#
 	PresentationCore.dll!System.Windows.Media.CombinedGeometry.GetBoundsInternal(System.Windows.Media.Pen pen = null, System.Windows.Media.Matrix matrix = {System.Windows.Media.Matrix}, double tolerance = 0.25, System.Windows.Media.ToleranceType type = Absolute) Line 146	C#
 	PresentationCore.dll!System.Windows.Media.Geometry.GetBoundsInternal(System.Windows.Media.Pen pen = null, System.Windows.Media.Matrix matrix = {System.Windows.Media.Matrix}) Line 161	C#
 	PresentationCore.dll!System.Windows.Media.BoundsDrawingContextWalker.PushClip(System.Windows.Media.Geometry clipGeometry = {System.Windows.Media.CombinedGeometry}) Line 375	C#
 	PresentationCore.dll!System.Windows.Media.RenderData.DrawingContextWalk(System.Windows.Media.DrawingContextWalker ctx = {System.Windows.Media.BoundsDrawingContextWalker}) Line 1336	C#
 	PresentationCore.dll!System.Windows.Media.RenderData.GetContentBounds(System.Windows.Media.BoundsDrawingContextWalker ctx = {System.Windows.Media.BoundsDrawingContextWalker}) Line 139	C#
 	PresentationCore.dll!System.Windows.UIElement.GetHitTestBounds() Line 1335	C#
```

也就是 CombinedGeometry 的底层也是通过 `PathGeometry.InternalCombine` 实现合并，然而以上的代码仅仅只是用在获取 Bounds 范围，而没有更进一步给到渲染层

但从这里也可以看到，只有很少的路径才能触发此问题，一般都能进入 WPF 的兼容处理逻辑

这也就是我决定不修复此问题的原因

本文的调试方法就是将 WPF 仓库拉下来，然后构建，构建方法请参阅 [手把手教你如何构建 WPF 官方开源框架源代码](https://blog.lindexi.com/post/%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E5%A6%82%E4%BD%95%E6%9E%84%E5%BB%BA-WPF-%E5%AE%98%E6%96%B9%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E6%BA%90%E4%BB%A3%E7%A0%81.html )

然后修改 csproj 文件，请将下面的 `C:\lindexi\Code\WPF` 替换为你的 WPF 文件夹

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net6.0-windows</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <UseWPF>true</UseWPF>
    <Platforms>x86</Platforms>
  </PropertyGroup>
  <PropertyGroup>
    <!-- Change this value based on where your local repo is located -->
    <WpfRepoRoot>C:\lindexi\Code\WPF</WpfRepoRoot>
    <!-- Change based on which assemblies you build (Release/Debug) -->
    <WpfConfig>Debug</WpfConfig>
    <WpfOuputFolder>Microsoft.DotNet.Wpf.GitHub.Debug</WpfOuputFolder>
    <!-- Publishing a self-contained app ensures our binaries are used. -->
    <SelfContained>true</SelfContained>
    <!-- The runtime identifier needs to match the architecture you built WPF assemblies for. -->
    <RuntimeIdentifier>win-x86</RuntimeIdentifier>
  </PropertyGroup>
  <ItemGroup>
    <Reference Include="$(WpfRepoRoot)\artifacts\packaging\$(WpfConfig)\$(WpfOuputFolder)\lib\net6.0\*.dll" />
    <ReferenceCopyLocalPaths Include="$(WpfRepoRoot)\artifacts\packaging\$(WpfConfig)\$(WpfOuputFolder)\lib\$(RuntimeIdentifier)\*.dll" />
    <ReferenceCopyLocalPaths Include="$(WpfRepoRoot)\artifacts\packaging\$(WpfConfig)\$(WpfOuputFolder)\runtimes\$(RuntimeIdentifier)\native\*.dll" />
    <ReferenceCopyLocalPaths Include="$(WpfRepoRoot)\artifacts\packaging\$(WpfConfig)\$(WpfOuputFolder)\runtimes\$(RuntimeIdentifier)\native\*.pdb" />
  </ItemGroup>
</Project>
```

以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/7fb51bcfda7ac0533999c1bff2dbda9054fc2cab/FerlallcabemberhalairBakifene) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/7fb51bcfda7ac0533999c1bff2dbda9054fc2cab/FerlallcabemberhalairBakifene) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 7fb51bcfda7ac0533999c1bff2dbda9054fc2cab
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 7fb51bcfda7ac0533999c1bff2dbda9054fc2cab
```

获取代码之后，进入 FerlallcabemberhalairBakifene 文件夹。记得替换 csproj 文件的 `C:\lindexi\Code\WPF` 为你的 WPF 文件夹

当前的 WPF 在 [https://github.com/dotnet/wpf](https://github.com/dotnet/wpf) 完全开源，使用友好的 MIT 协议，意味着允许任何人任何组织和企业任意处置，包括使用，复制，修改，合并，发表，分发，再授权，或者销售。在仓库里面包含了完全的构建逻辑，只需要本地的网络足够好（因为需要下载一堆构建工具），即可进行本地构建

更多 WPF 已知问题请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
