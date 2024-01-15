# WPF 的 Viewport3D 等 3D 模块在带 Intel UHD 770 设备上抛出渲染异常

在带 Intel UHD 770 的设备上，使用旧版本驱动，即小于 30.0.101.1660 版本驱动，将会导致 WPF 的 3D 模块出现渲染异常。此问题和 WPF 无关，此问题是 Intel 的 bug 且最新驱动版本已修复

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

官方问题记录地址：<https://community.intel.com/t5/Graphics/Crash-with-UHD-770-in-WPF-applications-using-Viewport3D/m-p/1370393>

相关 WPF 记录：<https://github.com/dotnet/wpf/issues/6312>

异常现象：

```
UCEERR_RENDERTHREADFAILURE (0x88980406)
System.Runtime.InteropServices.COMException (0x88980406): UCEERR_RENDERTHREADFAILURE (0x88980406)
   at System.Windows.Media.Composition.DUCE.Channel.SyncFlush()
   at System.Windows.Interop.HwndTarget.UpdateWindowSettings(Boolean enableRenderTarget, Nullable`1 channelSet)
   at System.Windows.Interop.HwndTarget.UpdateWindowPos(IntPtr lParam)
   at System.Windows.Interop.HwndTarget.HandleMessage(WindowMessage msg, IntPtr wparam, IntPtr lparam)
   at System.Windows.Interop.HwndSource.HwndTargetFilterMessage(IntPtr hwnd, Int32 msg, IntPtr wParam, IntPtr lParam, Boolean& handled)
   at MS.Win32.HwndWrapper.WndProc(IntPtr hwnd, Int32 msg, IntPtr wParam, IntPtr lParam, Boolean& handled)
   at MS.Win32.HwndSubclass.DispatcherCallbackOperation(Object o)
   at System.Windows.Threading.ExceptionWrapper.InternalRealCall(Delegate callback, Object args, Int32 numArgs)
   at System.Windows.Threading.ExceptionWrapper.TryCatchWhen(Object source, Delegate callback, Object args, Int32 numArgs, Delegate catchHandler)
```

复现设备：

* CPU 12900K

* Intel UHD 770 (Driver: 30.0.101.1404 - latest at time of writing)

* NVIDIA RTX 3090

* OS: Windows 11

最简复现代码

```xml
    <Viewport3D>

        <Viewport3D.Camera>
            <PerspectiveCamera Position = "2,0,10" LookDirection = "0.2,0.4,-1"
               FieldOfView = "65" UpDirection = "0,1,0" />
        </Viewport3D.Camera>

        <ModelVisual3D>
            <ModelVisual3D.Content>
                <Model3DGroup>
                    <AmbientLight Color = "Bisque" />

                    <GeometryModel3D>
                        <GeometryModel3D.Geometry>
                            <MeshGeometry3D Positions = "0,0,0 0,8,0 10,0,0 8,8,0"
                           Normals = "0,0,1 0,0,1 0,0,1 0,0,1" TriangleIndices = "0,2,1 1,2,3"/>
                        </GeometryModel3D.Geometry>

                        <GeometryModel3D.Material>
                            <DiffuseMaterial Brush = "Bisque" />
                        </GeometryModel3D.Material>
                    </GeometryModel3D>

                </Model3DGroup>
            </ModelVisual3D.Content>
        </ModelVisual3D>

    </Viewport3D>
```


解决方法：只需更新 Intel 驱动即可

此问题已经在 2022 的 5 月结束战斗