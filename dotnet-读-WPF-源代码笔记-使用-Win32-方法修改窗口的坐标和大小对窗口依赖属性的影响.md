
# dotnet 读 WPF 源代码笔记 使用 Win32 方法修改窗口的坐标和大小对窗口依赖属性的影响

咱可以使用 Win32 的 SetWindowPos 修改窗口的坐标和大小，此时 WPF 的窗口的 Left 和 Top 和 Width 和 Height 依赖属性也会受到影响，本文将会告诉大家在啥时候会同步更改 WPF 依赖属性的值，而什么时候不会

<!--more-->


<!-- 标签：WPF，WPF源代码 -->
<!-- 发布 -->

本文将会用到很多 Win32 方法，在 dotnet 基金会开源了对 win32 等的调用的封装库，请看 [https://github.com/dotnet/pinvoke](https://github.com/dotnet/pinvoke)

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/002e0b7c/FurnaheaneHejichaijair ) 欢迎小伙伴访问

在开始之前，咱先写一个 XAML 界面，用来绑定 Window 的依赖属性。以及加上几个按钮，用来使用 Win32 方法修改窗口坐标或大小

```xml
<Window x:Class="FurnaheaneHejichaijair.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:FurnaheaneHejichaijair"
        mc:Ignorable="d"
        x:Name="Root"
        Title="MainWindow" Height="450" Width="800">
  <Grid>
    <StackPanel>
      <TextBlock FontSize="50" Text="{Binding ElementName=Root,Path=Left}" />
      <TextBlock FontSize="50" Text="{Binding ElementName=Root,Path=Top}" />
      <TextBlock FontSize="50" Text="{Binding ElementName=Root,Path=Width}" />
      <TextBlock FontSize="50" Text="{Binding ElementName=Root,Path=Height}" />

      <Button x:Name="PositionButton" Margin="10,10,10,10" 
              HorizontalAlignment="Left" Content="修改坐标" 
              Click="PositionButton_OnClick"></Button>
      <Button Margin="10,10,10,10" 
              HorizontalAlignment="Left" Content="修改大小" 
              Click="SizeButton_OnClick"></Button>
      <Button x:Name="SetWindowLongPtrButton" Margin="10,10,10,10" 
              HorizontalAlignment="Left" Content="SetWindowLongPtr"
              Click="SetWindowLongPtrButton_OnClick"></Button>
    </StackPanel>
  </Grid>
</Window>
```

可以看到在完成了上面界面之后，在拖动窗口，以及修改窗口大小的时候，都可以看到值是对应变化的。接下来咱来试试 Win32 的方法来修改

在 PositionButton_OnClick 方法里面添加对窗口修改坐标的方法

```csharp
        public const string LibraryName = "user32";

        private void PositionButton_OnClick(object sender, RoutedEventArgs e)
        {
            var windowInteropHelper = new WindowInteropHelper(this);
            var SWP_NOSIZE = 0x0001;
            SetWindowPos(windowInteropHelper.Handle, IntPtr.Zero, (int)(Left + 10), (int)(Top + 10), 0, 0, SWP_NOSIZE);
        }

        /// <summary>
        /// 改变一个子窗口、弹出式窗口和顶层窗口的尺寸、位置和 Z 序。
        /// </summary>
        /// <param name="hWnd">窗口句柄。</param>
        /// <param name="hWndInsertAfter">
        /// 在z序中的位于被置位的窗口前的窗口句柄。该参数必须为一个窗口句柄，或下列值之一：
        /// <para>HWND_BOTTOM：将窗口置于 Z 序的底部。如果参数hWnd标识了一个顶层窗口，则窗口失去顶级位置，并且被置在其他窗口的底部。</para>
        /// <para>HWND_NOTOPMOST：将窗口置于所有非顶层窗口之上（即在所有顶层窗口之后）。如果窗口已经是非顶层窗口则该标志不起作用。</para>
        /// <para>HWND_TOP：将窗口置于Z序的顶部。</para>
        /// <para>HWND_TOPMOST：将窗口置于所有非顶层窗口之上。即使窗口未被激活窗口也将保持顶级位置。</para>
        /// 如无须更改，请使用 IntPtr.Zero 的值
        /// </param>
        /// <param name="x">以客户坐标指定窗口新位置的左边界。</param>
        /// <param name="y">以客户坐标指定窗口新位置的顶边界。</param>
        /// <param name="cx">以像素指定窗口的新的宽度。如无须更改，请在 <paramref name="wFlagslong"/> 设置 <see cref="WindowPositionFlags.SWP_NOSIZE"/> 的值 </param>
        /// <param name="cy">以像素指定窗口的新的高度。如无须更改，请在 <paramref name="wFlagslong"/> 设置 <see cref="WindowPositionFlags.SWP_NOSIZE"/> 的值</param>
        /// <param name="wFlagslong">
        /// 可传入 <see cref="WindowPositionFlags"/> 枚举中的值
        /// 窗口尺寸和定位的标志。该参数可以是下列值的组合：
        /// <para>SWP_ASYNCWINDOWPOS：如果调用进程不拥有窗口，系统会向拥有窗口的线程发出需求。这就防止调用线程在其他线程处理需求的时候发生死锁。</para>
        /// <para>SWP_DEFERERASE：防止产生 WM_SYNCPAINT 消息。</para>
        /// <para>SWP_DRAWFRAME：在窗口周围画一个边框（定义在窗口类描述中）。</para>
        /// <para>SWP_FRAMECHANGED：给窗口发送 WM_NCCALCSIZE 消息，即使窗口尺寸没有改变也会发送该消息。如果未指定这个标志，只有在改变了窗口尺寸时才发送 WM_NCCALCSIZE。</para>
        /// <para>SWP_HIDEWINDOW：隐藏窗口。</para>
        /// <para>SWP_NOACTIVATE：不激活窗口。如果未设置标志，则窗口被激活，并被设置到其他最高级窗口或非最高级组的顶部（根据参数hWndlnsertAfter设置）。</para>
        /// <para>SWP_NOCOPYBITS：清除客户区的所有内容。如果未设置该标志，客户区的有效内容被保存并且在窗口尺寸更新和重定位后拷贝回客户区。</para>
        /// <para>SWP_NOMOVE：维持当前位置（忽略X和Y参数）。</para>
        /// <para>SWP_NOOWNERZORDER：不改变 Z 序中的所有者窗口的位置。</para>
        /// <para>SWP_NOREDRAW：不重画改变的内容。如果设置了这个标志，则不发生任何重画动作。适用于客户区和非客户区（包括标题栏和滚动条）和任何由于窗回移动而露出的父窗口的所有部分。如果设置了这个标志，应用程序必须明确地使窗口无效并区重画窗口的任何部分和父窗口需要重画的部分。</para>
        /// <para>SWP_NOREPOSITION：与 SWP_NOOWNERZORDER 标志相同。</para>
        /// <para>SWP_NOSENDCHANGING：防止窗口接收 WM_WINDOWPOSCHANGING 消息。</para>
        /// <para>SWP_NOSIZE：维持当前尺寸（忽略 cx 和 cy 参数）。</para>
        /// <para>SWP_NOZORDER：维持当前 Z 序（忽略 hWndlnsertAfter 参数）。</para>
        /// <para>SWP_SHOWWINDOW：显示窗口。</para>
        /// </param>
        /// <returns>如果函数成功，返回值为非零；如果函数失败，返回值为零。若想获得更多错误消息，请调用 GetLastError 函数。</returns>
        [DllImport(LibraryName, ExactSpelling = true, SetLastError = true)]
        public static extern Int32 SetWindowPos(IntPtr hWnd, IntPtr hWndInsertAfter, Int32 x, Int32 y, Int32 cx,
            Int32 cy, Int32 wFlagslong);
```

可以看到点击修改坐标按钮，就可以修改窗口的坐标，此时点击的时候，依赖属性也跟随变化

再来实现修改窗口大小的方法，点击方法将调用 SetWindowPos 方法修改窗口的宽度和高度

```csharp
        private void SizeButton_OnClick(object sender, RoutedEventArgs e)
        {
            var windowInteropHelper = new WindowInteropHelper(this);
            var SWP_NOMOVE = 0x0002;
            SetWindowPos(windowInteropHelper.Handle, IntPtr.Zero, 0, 0, (int)(Width + 10), (int)(Height + 10), SWP_NOMOVE);
        }
```

此时点击修改窗口大小的按钮，通过 Win32 方法修改窗口大小，也可以看到依赖属性也进行变化。但如果此时咱点击一下最大化，那么点击修改窗口坐标按钮，是可以修改窗口坐标的，同时窗口的状态依然是最大化。但是此时的依赖属性没有跟随变化

原因还需要从完全开源的 WPF 仓库里面了解，官方的开源仓库放在 [https://github.com/dotnet/wpf](https://github.com/dotnet/wpf) 欢迎大家下载所有源代码

在 `src\Microsoft.DotNet.Wpf\src\PresentationFramework\System\Windows\Window.cs` 文件里面有以下的定义

```csharp
        /// <summary>
        ///     This is the hook to HwndSource that is called when window messages related to
        ///     this window occur. Currently, we listen to the following messages
        ///
        ///         WM_CLOSE        : We listen to this message in order to fire the Closing event.
        ///                           If the user cancels window closing, we set handled to true so
        ///                           that the DefWindowProc does not handle this message. Otherwise,
        ///                           we set handled to false.
        ///         WM_DESTROY      : We listen to this message in order to fire the Closed event.
        ///                           Handled is always set to false.
        ///         WM_ACTIVATE     : Used for Activated and deactivated events
        ///         WM_SIZE         : Used for SizeChanged, StateChanged events. Also, helps us keep our
        ///                           size updated
        ///         WM_MOVE:        : Used for location changed event and to keep our cached top/left
        ///                           updated
        ///         WM_GETMINMAXINFO: Used to enforce Max/MinHeight and Max/MinWidth
        /// </summary>
        /// <param name="hwnd"></param>
        /// <param name="msg"></param>
        /// <param name="wParam"></param>
        /// <param name="lParam"></param>
        /// <param name="handled"></param>
        /// <returns></returns>
        private IntPtr WindowFilterMessage( IntPtr hwnd,
            int msg,
            IntPtr wParam,
            IntPtr lParam,
            ref bool handled)
        {

        }
```

在这个方法里面，将会从 Win 消息拿到对应的值分发给对应的方法处理，如下面代码

```csharp
switch (message)
{
    case WindowMessage.WM_CLOSE:
        handled = WmClose();
        break;
    case WindowMessage.WM_DESTROY:
        handled = WmDestroy();
        break;
    case WindowMessage.WM_ACTIVATE:
        handled = WmActivate(wParam);
        break;
    case WindowMessage.WM_MOVE: // 窗口移动
        handled = WmMoveChanged();
        break;
    case WindowMessage.WM_NCHITTEST:
        handled = WmNcHitTest(lParam, ref retInt);
        break;
    case WindowMessage.WM_SHOWWINDOW:
        handled = WmShowWindow(wParam, lParam);
        break;
    case WindowMessage.WM_COMMAND:
        handled = WmCommand(wParam, lParam);
        break;
    default:
        handled = false;
        break;
}
```

在 `WindowMessage.WM_MOVE` 消息里面，将会调用到 WmMoveChanged 方法，这个方法的逻辑大概如下

```csharp
        private bool WmMoveChanged()
        {
        	// 在 WindowBounds 属性里面，将会获取当前 Win32 窗口的坐标和大小
            // the input lparam gives the client location,
            // so just call GetWindowRect for Left and Top.
            NativeMethods.RECT rc = WindowBounds;

            // 此时需要将屏幕的坐标转换为 WPF 的坐标
            Point ptLogicalUnits = DeviceToLogicalUnits(new Point(rc.left, rc.top));

            // 如果值更新了，那么将会更新 _actualLeft 和 _actualTop 属性
            if (!DoubleUtil.AreClose(_actualLeft, ptLogicalUnits.X) ||
                !DoubleUtil.AreClose(_actualTop, ptLogicalUnits.Y))
            {
                _actualLeft = ptLogicalUnits.X;
                _actualTop = ptLogicalUnits.Y;

                // In Window, WmMoveChangedHelper write the local value of Top/Left
                // (if necessary) or updates the property system values for
                // Top/Left by calling CoerceValue.  Furthermore, it fires the
                // LocationChanged event.  RBW overrides WmMoveChangedHelper to do
                // nothing as writing Top/Left is not supported for RBW and
                // LocationChanged is never fired for it either.
                WmMoveChangedHelper();
            }

            return false;
        }

        private NativeMethods.RECT WindowBounds
        {
            get
            {
                Debug.Assert( _swh != null );
                return _swh.WindowBounds;
            }
        }
```

在 `_swh.WindowBounds` 通用也是一个只有 get 的属性，定义如下

```csharp
        internal class SourceWindowHelper
        {
                internal NativeMethods.RECT WindowBounds
                {
                    get
                    {
                        NativeMethods.RECT rc = new NativeMethods.RECT(0,0,0,0);
                        SafeNativeMethods.GetWindowRect(new HandleRef(this, CriticalHandle), ref rc);

                        return rc;
                    }
                }
        }
```

也就是说本质是通过 User32.dll 的 GetWindowRect 方法获取 Win32 窗口的坐标和大小

而更改依赖属性的逻辑是放在 WmMoveChangedHelper 方法的，代码如下

```csharp
        internal void WmMoveChangedHelper()
        {
        	// 如果窗口是最大化，不更新依赖属性，但是窗口最大化可以通过 Win32 方法修改窗口坐标和大小，此时的依赖属性就没有和实际窗口的坐标相同
            if (WindowState == WindowState.Normal)
            {
                try
                {
                    _updateHwndLocation = false;
                    // 更新依赖属性
                    SetValue(LeftProperty, _actualLeft);
                    SetValue(TopProperty, _actualTop);
                }
                finally
                {
                    _updateHwndLocation = true;
                }

                // Event handler exception continuality: if exception occurs in LocationChanged event handler, our state will not be
                // corrupted because the states related to LocationChanged, LeftProperty, TopProperty, Left and Top are set before the event is fired.
                // Please check event handler exception continuality if the logic changes.
                OnLocationChanged(EventArgs.Empty);
            }
        }
```

可以看到在 WmMoveChangedHelper 方法里面会判断 `WindowState == WindowState.Normal` 才会更新 Left 和 Top 依赖属性。这就是为什么最大化的时候修改坐标不会更新依赖属性

另外在 WmMoveChanged 方法的实现里面，可以看到一个坑，在判断是否需要更新的时候，是采用 `_actualLeft` 和 `_actualTop` 判断的

```csharp
 // 如果值更新了，那么将会更新 _actualLeft 和 _actualTop 属性
 if (!DoubleUtil.AreClose(_actualLeft, ptLogicalUnits.X) ||
     !DoubleUtil.AreClose(_actualTop, ptLogicalUnits.Y))
 {
 	// 忽略代码
 }
```

如果此时我在使用 Win32 更改的过程中，也修改了 Left 和 Top 依赖属性呢？可以看到此时的 `_actualLeft` 和 `_actualTop` 和 Win32 相同，此时就不会再次调用更新了，此时的 Left 和 Top 依赖属性就没有和 Win32 同步了

上面是说到的是修改窗口的坐标，那如果修改的是窗口的大小呢？在 WindowFilterMessage 方法里面，除了调用 WmMoveChanged 方法外，还有以下代码

```csharp
            switch (message)
            {
                case WindowMessage.WM_GETMINMAXINFO:
                    handled = WmGetMinMaxInfo(lParam);
                    break;
                case WindowMessage.WM_SIZE:
                    handled = WmSizeChanged(wParam);
                    break;
            }
```

可以看到在消息是 `WindowMessage.WM_SIZE` 将会调用 WmSizeChanged 方法，这个方法的逻辑如下

```csharp
        private bool WmSizeChanged(IntPtr wParam)
        {
            // 调用 WindowBounds 属性，获取当前的坐标
            NativeMethods.RECT rc = WindowBounds;
            // 计算窗口的大小，尽管使用的是 Point 但实际含义是 Size 哦，原因是为了重复调用 DeviceToLogicalUnits 方法而已
            Point windowSize = new Point(rc.right - rc.left, rc.bottom - rc.top);
            // 转换为 WPF 坐标，这里的 Point 其实是 Size 哈，只是 WPF 的开发者 hack 一下，使用 DeviceToLogicalUnits 方法返回的 Point 而已
            Point ptLogicalUnits = DeviceToLogicalUnits(windowSize);

            try
            {
            	// 修改依赖属性
                _updateHwndSize = false;
                SetValue(FrameworkElement.WidthProperty, ptLogicalUnits.X);
                SetValue(FrameworkElement.HeightProperty, ptLogicalUnits.Y);
            }
            finally
            {
                _updateHwndSize = true;
            }

            // 忽略代码

            return false;
        }

```

因此 WPF 的依赖属性是根据 Windows 消息，更新依赖属性的，而在 Left 和 Top 属性的更新里面，会先判断 `_actualLeft` 和 `_actualTop` 是否和 Win32 的相同，如果相同就不更新，因此行为上和宽度和高度的属性有点差别。另外最大化也会影响 Left 和 Top 属性，因为在更新这两个属性之前会先判断窗口，如果是最大化的，将不会更新这两个依赖属性。但是宽度和高度属性就没有这个判断

当前的 WPF 在 [https://github.com/dotnet/wpf](https://github.com/dotnet/wpf) 完全开源，使用友好的 MIT 协议，意味着允许任何人任何组织和企业任意处置，包括使用，复制，修改，合并，发表，分发，再授权，或者销售。在仓库里面包含了完全的构建逻辑，只需要本地的网络足够好（因为需要下载一堆构建工具），即可进行本地构建





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。