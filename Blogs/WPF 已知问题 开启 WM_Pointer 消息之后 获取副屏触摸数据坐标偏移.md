本文记录 WPF 触摸的一个已知问题，仅在开启 WM_Pointer 消息之后，将应用程序运行在包含多个屏幕的带触摸屏的设备上，如此时在非主屏幕的触摸屏上进行触摸，使用 GetStylusPoint 或 GetIntermediateTouchPoints 方法获取触摸点时，将会发现所获取的触摸点的坐标是偏的，偏的坐标差值刚好是整个屏幕距离

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

此问题由 [少珺](https://github.com/kkwpsv) 小伙伴发现且修复，我只是代为记录的工具人

此问题已经报告给 WPF 官方，请看 [https://github.com/dotnet/wpf/issues/8517](https://github.com/dotnet/wpf/issues/8517)

此问题已经被 [少珺](https://github.com/kkwpsv) 修复，请看 [https://github.com/dotnet-campus/wpf/pull/9](https://github.com/dotnet-campus/wpf/pull/9)

修复后的测试包是 [https://www.nuget.org/packages/dotnetCampus.WPF.Resource/6.0.4-alpha07-test06](https://www.nuget.org/packages/dotnetCampus.WPF.Resource/6.0.4-alpha07-test06)

修复后的测试包的使用例子请参阅 [https://github.com/lindexi/lindexi_gd/tree/893292f260c4570ff63e68b9e0a29052a187d0c6/BenukalliwayaChayjanehall](https://github.com/lindexi/lindexi_gd/tree/893292f260c4570ff63e68b9e0a29052a187d0c6/BenukalliwayaChayjanehall)

问题描述：

此问题发生在开启 WM_Pointer 消息之后的触摸应用程序上，此问题要求运行在多个屏幕上，且触摸到非主屏幕上。通过 GetStylusPoint 或 GetIntermediateTouchPoints 等方法获取触摸点信息时，可以看到触摸点信息存在偏差，偏差的坐标差值刚好是整个屏幕距离，也就是差了 N 个屏幕距离

复现步骤：

1. 创建一个空 WPF 程序，按照 [WPF dotnet core 如何开启 Pointer 消息的支持](https://blog.lindexi.com/post/WPF-dotnet-core-%E5%A6%82%E4%BD%95%E5%BC%80%E5%90%AF-Pointer-%E6%B6%88%E6%81%AF%E7%9A%84%E6%94%AF%E6%8C%81.html ) 博客提供的方法开启 WM_Pointer 消息
2. 在 MainWindow 放入 InkCanvas 控件
3. 准备好环境，最简环境是有两个屏幕，其中副屏是触摸屏。运行程序，将程序的主窗口移动到副屏上，对应用程序进行触摸

此时你将会发现应用程序无法绘制出你所画出的笔迹。当你将程序移动到主屏幕上时，如果恰好此时你的主屏幕也是触摸屏，那你将可以看到应用程序实际是能正常工作的，画出你触摸的笔迹。如果你将整个窗口缩放很大，跨了你的两个屏幕，你将会发现在副屏上所画的内容将会显示到主屏幕上去。且所偏差的坐标差值刚好是整个屏幕距离，如果刚好你的两个屏幕的虚拟尺寸（非物理尺寸）是一样大的，那这个偏差就更好看出来了

问题原因：

此问题是在 WPF 框架中的 HwndPointerInputProvider.cs 的代码实现不正确导致的，和 WM_Pointer 本身关系不大，仅仅只是因为这个代码实现只有开启了 WM_Pointer 才会进入。在 HwndPointerInputProvider 的 GetOriginOffsetsLogical 方法里面，没有考虑多屏幕的坐标系问题，只是计算了当前窗口所在的屏幕的坐标，没有考虑应该计算整个大的虚拟屏幕的坐标

```csharp
 private void GetOriginOffsetsLogical(out int originOffsetX, out int originOffsetY) 
 { 
     Point originScreenCoord = _source.Value.RootVisual.PointToScreen(new Point(0, 0)); 
  
     // Use the inverse of our logical tablet to screen matrix to generate tablet coords 
     MatrixTransform screenToTablet = new MatrixTransform(_currentTabletDevice.TabletToScreen); 
     screenToTablet = (MatrixTransform)screenToTablet.Inverse; 
  
     Point originTabletCoord = originScreenCoord * screenToTablet.Matrix; 
  
     originOffsetX = (int)Math.Round(originTabletCoord.X); 
     originOffsetY = (int)Math.Round(originTabletCoord.Y); 
 }
```

以上代码的错误点在于 `Point originScreenCoord = _source.Value.RootVisual.PointToScreen(new Point(0, 0))` 计算的是当前窗口所在的显示器的窗口左上角的相对坐标。正确的实现应该考虑当前窗口所在屏幕的虚拟屏幕坐标，如以下 [少珺](https://github.com/kkwpsv) 小伙伴修复后的代码

```csharp
        private void GetOriginOffsetsLogical(out int originOffsetX, out int originOffsetY)
        {
            Point originScreenCoord = new Point();

            HwndSource hwndSource = PresentationSource.FromVisual(_source.Value.RootVisual) as HwndSource;
            if (hwndSource != null)
            {
                HandleRef handleRef = new HandleRef(hwndSource, hwndSource.CriticalHandle);

                MS.Win32.NativeMethods.POINT point = new MS.Win32.NativeMethods.POINT();
                MS.Win32.UnsafeNativeMethods.ClientToScreen(handleRef, ref point);

                var displayRect = _currentTabletDevice.DeviceInfo.DisplayRect;

                originScreenCoord = new Point(point.x - displayRect.left, point.y - displayRect.top);
            }

            // Use the inverse of our logical tablet to screen matrix to generate tablet coords
            MatrixTransform screenToTablet = new MatrixTransform(_currentTabletDevice.TabletToScreen);
            screenToTablet = (MatrixTransform)screenToTablet.Inverse;
            Point originTabletCoord = originScreenCoord * screenToTablet.Matrix;
            originOffsetX = (int)Math.Round(originTabletCoord.X);
            originOffsetY = (int)Math.Round(originTabletCoord.Y);
        }
```

以上代码先算出当前窗口的左上角相对于屏幕的坐标，也就是 `MS.Win32.UnsafeNativeMethods.ClientToScreen(handleRef, ref point);` 所实现的内容。再经过 `_currentTabletDevice.DeviceInfo.DisplayRect` 属性获取当前窗口所在屏幕的虚拟屏幕坐标，将上一步计算到的窗口相对于屏幕的坐标减去当前的屏幕的虚拟坐标才是计算到正确的坐标值

详细更改请参阅 [https://github.com/dotnet-campus/wpf/pull/9](https://github.com/dotnet-campus/wpf/pull/9)

我将 [少珺](https://github.com/kkwpsv) 小伙伴修复后的代码合入到 [https://github.com/dotnet-campus/dotnetCampus.CustomWpf/](https://github.com/dotnet-campus/dotnetCampus.CustomWpf/) 仓库里面，这个 CustomWpf 仓库是我所在的团队用来对 WPF 框架进行日常的开发测试所使用的仓库，相对 WPF 仓库来说会更加激进一点。合入之后，我打出了 NuGet 包，大家可以通过编辑 csproj 项目文件，添加以下代码使用到此测试版本的 WPF 框架

```xml
  <ItemGroup>
    <PackageReference Include="dotnetCampus.WPF.Resource" Version="6.0.4-alpha07-test06" />
  </ItemGroup>

  <ItemGroup>
    <Reference Include="$(CustomWpfAssetsFolder)lib\net6.0\*.dll" />
    <ReferenceCopyLocalPaths Include="$(CustomWpfAssetsFolder)lib\net6.0\*.dll" />
  </ItemGroup>
```

更多关于 WM_Pointer 的坑，请看 [WPF 开启Pointer消息存在的坑](https://blog.lindexi.com/post/WPF-%E5%BC%80%E5%90%AFPointer%E6%B6%88%E6%81%AF%E5%AD%98%E5%9C%A8%E7%9A%84%E5%9D%91.html )

更多关于 WPF 的博客请看 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
