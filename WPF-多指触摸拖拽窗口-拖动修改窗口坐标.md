
# WPF 多指触摸拖拽窗口 拖动修改窗口坐标

在 WPF 中，如果是鼠标点击拖动窗口坐标，可以调用 Window 的 DragMove 方法，但是如果是触摸，就需要自己调用 Win32 的方法实现

<!--more-->


<!-- CreateTime:2020/11/26 10:07:10 -->

<!-- 发布 -->

在 WPF 中，调用 Window 的 DragMove 方法要求鼠标的左键（主键）按下，否则将会抛出如下代码

```csharp
System.InvalidOperationException:“只能在按下主鼠标按钮时调用 DragMove。”
```

或英文版的代码

```csharp
System.InvalidOperationException:"Can only call DragMove when primary mouse button is down"
```

因此想要在 WPF 中使用手指 finger 进行 Touch 触摸拖拽窗口，拖动修改窗口坐标就需要用到 Win32 的方法了。相信大家都知道，在修改某个容器的坐标的时候，不能使用这个容器内的坐标做参考，所以在 Touch 拖动修改窗口坐标的时候，就不能使用监听窗口的事件拿到的坐标来作为参考

想要能平滑的移动窗口，就需要获取相对于屏幕的坐标，而如果此时处理多指的 Manipulation 的动作，那么整个逻辑将会非常复杂。本文仅仅支持使用一个手指的移动，因为使用了 GetCursorPos 的方法

当然了，此时假装是支持多指拖动也是可以的，只需要在进行多指触摸的时候开启拖动就可以了，此时用户的交互上不会有很大的差别

在开始之前，咱来封装一个类 DragMoveWindowHelper 用来在触摸下拖动窗口

```csharp
    public static class DragMoveWindowHelper
    {
        public static void DragMove(Window window)
        {
        	// 这里的 DragMoveMode 在下文实现
            var dragMoveMode = new DragMoveMode(window);
            dragMoveMode.Start();
        }
    }
```

上面代码的 DragMoveMode 类放在下文实现。在封装完成了 DragMoveWindowHelper 类就可以尝试在拖动的时候使用，如下面代码

```csharp
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();

            TouchDown += MainWindow_TouchDown;
            TouchUp += MainWindow_TouchUp;
        }

        private void MainWindow_TouchUp(object sender, TouchEventArgs e)
        {
            _currentTouchCount--;
        }

        private void MainWindow_TouchDown(object sender, TouchEventArgs e)
        {
            CaptureTouch(e.TouchDevice);

            if (_currentTouchCount == 0)
            {
                DragMoveWindowHelper.DragMove(this);
            }

            _currentTouchCount++;
        }

        private uint _currentTouchCount;
    }
```

上面代码有一点需要小心就是 CaptureTouch 是必备的，否则你会发现拖动的时候，拖动太快了，就丢失触摸设备了，触摸设备被你窗口后面的其他软件抓了

下面开始实现 DragMoveMode 也就是核心的通过触摸拖动窗口的逻辑

大概对外的接口方法实现请看代码

```csharp
        class DragMoveMode
        {
            public DragMoveMode(Window window)
            {
                _window = window;
            }

            public void Start()
            {
                var window = _window;

                window.PreviewMouseMove += Window_PreviewMouseMove;
                window.PreviewMouseUp += Window_PreviewMouseUp;
                window.LostMouseCapture += Window_LostMouseCapture;
            }

            public void Stop()
            {
                Window window = _window;

                window.PreviewMouseMove -= Window_PreviewMouseMove;
                window.PreviewMouseUp -= Window_PreviewMouseUp;
                window.LostMouseCapture -= Window_LostMouseCapture;
            }

            private readonly Window _window;
        }
```

在上面代码里面监听 PreviewMouseMove 是为了获取移动的时机，而不是为了获取相对的坐标。而 PreviewMouseUp 可以用来了解啥时候结束。当然了 LostMouseCapture 也需要监听，和 PreviewMouseUp 一样用来了解啥时候结束

在 Window_PreviewMouseMove 方法需要先判断是否第一次进入移动，因此咱没有监听 MouseDown 方法。为什么没有监听 MouseDown 方法，是因为在上层业务此时业务调用 MoseDown 完成

判断是否第一次进入移动需要一个辅助的字段，咱定义一个叫上一次点击的坐标字段

```csharp
            private Win32.User32.Point? _lastPoint;
```

上面代码的 Win32.User32 是我定义的代码，这些定义将会放在本文最后

判断是第一次进入移动可以使用下面代码

```csharp
            private void Window_PreviewMouseMove(object sender, MouseEventArgs e)
            {
                Win32.User32.GetCursorPos(out var lpPoint);

                if (_lastPoint == null)
                {
                    _lastPoint = lpPoint;
                    _window.CaptureMouse();
                }
            }
```

通过 GetCursorPos 的 Win32 方法可以拿到相对于屏幕坐标的鼠标坐标，而触摸默认会将第一个触摸点转换为鼠标坐标，因此拿到的坐标点不是相对于窗口内的，这样就能做到在移动的时候不会抖

接下来判断相对上一次的移动距离，如下面代码

```csharp
                var dx = lpPoint.X - _lastPoint.Value.X;
                var dy = lpPoint.Y - _lastPoint.Value.Y;

                Debug.WriteLine($"dx={dx} dy={dy}");
```

拿到的 dx 和 dy 就可以用来设置窗口的左上角坐标了。而此时不能通过 Window 的 Top 和 Left 属性获取，这两个属性的值使用的是 WPF 单位和坐标，而咱计算的 dx 和 dy 是相对于屏幕的坐标，因此需要调用 GetWindowRect 这个 win32 方法获取窗口所在屏幕的坐标

设置窗口坐标也需要使用屏幕坐标来设置，需要调用 SetWindowPos 方法，代码如下

```csharp
     var handle = new WindowInteropHelper(_window).Handle;

     Win32.User32.GetWindowRect(handle, out var lpRect);

     Win32.User32.SetWindowPos(handle, IntPtr.Zero, lpRect.Left + dx, lpRect.Top + dy, 0, 0,
                        (int) (Win32.User32.WindowPositionFlags.SWP_NOSIZE |
                               Win32.User32.WindowPositionFlags.SWP_NOZORDER));
```

这个 Window_PreviewMouseMove 方法代码如下

```csharp
            private void Window_PreviewMouseMove(object sender, MouseEventArgs e)
            {
                Win32.User32.GetCursorPos(out var lpPoint);

                if (_lastPoint == null)
                {
                    _lastPoint = lpPoint;
                    _window.CaptureMouse();
                }

                var dx = lpPoint.X - _lastPoint.Value.X;
                var dy = lpPoint.Y - _lastPoint.Value.Y;

                Debug.WriteLine($"dx={dx} dy={dy}");

                // 以下的 60 是表示最大移动速度
                if (Math.Abs(dx) < 60 && Math.Abs(dy) < 60)
                {
                    var handle = new WindowInteropHelper(_window).Handle;

                    Win32.User32.GetWindowRect(handle, out var lpRect);

                    Win32.User32.SetWindowPos(handle, IntPtr.Zero, lpRect.Left + dx, lpRect.Top + dy, 0, 0,
                        (int) (Win32.User32.WindowPositionFlags.SWP_NOSIZE |
                               Win32.User32.WindowPositionFlags.SWP_NOZORDER));
                }

                _lastPoint = lpPoint;
            }

```

在 Window_PreviewMouseUp 和 Window_LostMouseCapture 方法调用的是清理的代码，解决内存泄露

```csharp
            private void Window_LostMouseCapture(object sender, MouseEventArgs e)
            {
                Stop();
            }

            private void Window_PreviewMouseUp(object sender, MouseButtonEventArgs e)
            {
                Stop();
            }
```

大概就完成了触摸拖动窗口的逻辑，下面代码是 Win32 的代码，需要加到你的代码里面，这样才能构建通过

```csharp
        private static class Win32
        {
            public static class User32
            {
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
                /// </param>
                /// <param name="x">以客户坐标指定窗口新位置的左边界。</param>
                /// <param name="y">以客户坐标指定窗口新位置的顶边界。</param>
                /// <param name="cx">以像素指定窗口的新的宽度。</param>
                /// <param name="cy">以像素指定窗口的新的高度。</param>
                /// <param name="wFlagslong">
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

                [Flags]
                public enum WindowPositionFlags
                {
                    /// <summary>
                    ///     If the calling thread and the thread that owns the window are attached to different input queues, the system posts
                    ///     the request to the thread that owns the window. This prevents the calling thread from blocking its execution while
                    ///     other threads process the request.
                    /// </summary>
                    SWP_ASYNCWINDOWPOS = 0x4000,

                    /// <summary>
                    ///     Prevents generation of the WM_SYNCPAINT message.
                    /// </summary>
                    SWP_DEFERERASE = 0x2000,

                    /// <summary>
                    ///     Draws a frame (defined in the window's class description) around the window.
                    /// </summary>
                    SWP_DRAWFRAME = 0x0020,

                    /// <summary>
                    ///     Applies new frame styles set using the SetWindowLong function. Sends a WM_NCCALCSIZE message to the window, even if
                    ///     the window's size is not being changed. If this flag is not specified, WM_NCCALCSIZE is sent only when the window's
                    ///     size is being changed.
                    /// </summary>
                    SWP_FRAMECHANGED = 0x0020,

                    /// <summary>
                    ///     Hides the window.
                    /// </summary>
                    SWP_HIDEWINDOW = 0x0080,

                    /// <summary>
                    ///     Does not activate the window. If this flag is not set, the window is activated and moved to the top of either the
                    ///     topmost or non-topmost group (depending on the setting of the hWndInsertAfter parameter).
                    /// </summary>
                    SWP_NOACTIVATE = 0x0010,

                    /// <summary>
                    ///     Discards the entire contents of the client area. If this flag is not specified, the valid contents of the client
                    ///     area are saved and copied back into the client area after the window is sized or repositioned.
                    /// </summary>
                    SWP_NOCOPYBITS = 0x0100,

                    /// <summary>
                    ///     Retains the current position (ignores X and Y parameters).
                    /// </summary>
                    SWP_NOMOVE = 0x0002,

                    /// <summary>
                    ///     Does not change the owner window's position in the Z order.
                    /// </summary>
                    SWP_NOOWNERZORDER = 0x0200,

                    /// <summary>
                    ///     Does not redraw changes. If this flag is set, no repainting of any kind occurs. This applies to the client area,
                    ///     the nonclient area (including the title bar and scroll bars), and any part of the parent window uncovered as a
                    ///     result of the window being moved. When this flag is set, the application must explicitly invalidate or redraw any
                    ///     parts of the window and parent window that need redrawing.
                    /// </summary>
                    SWP_NOREDRAW = 0x0008,

                    /// <summary>
                    ///     Same as the SWP_NOOWNERZORDER flag.
                    /// </summary>
                    SWP_NOREPOSITION = 0x0200,

                    /// <summary>
                    ///     Prevents the window from receiving the WM_WINDOWPOSCHANGING message.
                    /// </summary>
                    SWP_NOSENDCHANGING = 0x0400,

                    /// <summary>
                    ///     Retains the current size (ignores the cx and cy parameters).
                    /// </summary>
                    SWP_NOSIZE = 0x0001,

                    /// <summary>
                    ///     Retains the current Z order (ignores the hWndInsertAfter parameter).
                    /// </summary>
                    SWP_NOZORDER = 0x0004,

                    /// <summary>
                    ///     Displays the window.
                    /// </summary>
                    SWP_SHOWWINDOW = 0x0040
                }


                public const string LibraryName = "user32";

                /// <summary>
                /// 获取的是以屏幕为坐标轴窗口坐标
                /// </summary>
                /// <param name="hWnd"></param>
                /// <param name="lpRect"></param>
                /// <returns></returns>
                [return: MarshalAs(UnmanagedType.Bool)]
                [DllImport(LibraryName, ExactSpelling = true)]
                public static extern bool GetWindowRect(IntPtr hWnd, out Rectangle lpRect);

                /// <summary>
                /// 在 Win32 函数使用的矩形
                /// </summary>
                [StructLayout(LayoutKind.Sequential)]
                public partial struct Rectangle : IEquatable<Rectangle>
                {
                    /// <summary>
                    ///  创建在 Win32 函数使用的矩形
                    /// </summary>
                    /// <param name="left"></param>
                    /// <param name="top"></param>
                    /// <param name="right"></param>
                    /// <param name="bottom"></param>
                    public Rectangle(int left = 0, int top = 0, int right = 0, int bottom = 0)
                    {
                        Left = left;
                        Top = top;
                        Right = right;
                        Bottom = bottom;
                    }

                    /// <summary>
                    /// 创建在 Win32 函数使用的矩形
                    /// </summary>
                    /// <param name="width">矩形的宽度</param>
                    /// <param name="height">矩形的高度</param>
                    public Rectangle(int width = 0, int height = 0) : this(0, 0, width, height)
                    {
                    }

                    public int Left;
                    public int Top;
                    public int Right;
                    public int Bottom;

                    public bool Equals(Rectangle other)
                    {
                        return (Left == other.Left) && (Right == other.Right) && (Top == other.Top) &&
                               (Bottom == other.Bottom);
                    }

                    public override bool Equals(object obj)
                    {
                        return obj is Rectangle && Equals((Rectangle) obj);
                    }

                    public static bool operator ==(Rectangle left, Rectangle right)
                    {
                        return left.Equals(right);
                    }

                    public static bool operator !=(Rectangle left, Rectangle right)
                    {
                        return !(left == right);
                    }

                    public override int GetHashCode()
                    {
                        unchecked
                        {
                            var hashCode = (int) Left;
                            hashCode = (hashCode * 397) ^ (int) Top;
                            hashCode = (hashCode * 397) ^ (int) Right;
                            hashCode = (hashCode * 397) ^ (int) Bottom;
                            return hashCode;
                        }
                    }

                    /// <summary>
                    /// 获取当前矩形是否空矩形
                    /// </summary>
                    public bool IsEmpty => this.Left == 0 && this.Top == 0 && this.Right == 0 && this.Bottom == 0;

                    /// <summary>
                    /// 矩形的宽度
                    /// </summary>
                    public int Width
                    {
                        get { return unchecked((int) (Right - Left)); }
                        set { Right = unchecked((int) (Left + value)); }
                    }

                    /// <summary>
                    /// 矩形的高度
                    /// </summary>
                    public int Height
                    {
                        get { return unchecked((int) (Bottom - Top)); }
                        set { Bottom = unchecked((int) (Top + value)); }
                    }

                    /// <summary>
                    /// 通过 x、y 坐标和宽度高度创建矩形
                    /// </summary>
                    /// <param name="x"></param>
                    /// <param name="y"></param>
                    /// <param name="width"></param>
                    /// <param name="height"></param>
                    /// <returns></returns>
                    public static Rectangle Create(int x, int y, int width, int height)
                    {
                        unchecked
                        {
                            return new Rectangle(x, y, (int) (width + x), (int) (height + y));
                        }
                    }

                    public static Rectangle From(ref Rectangle lvalue, ref Rectangle rvalue,
                        Func<int, int, int> leftTopOperation,
                        Func<int, int, int> rightBottomOperation = null)
                    {
                        if (rightBottomOperation == null)
                            rightBottomOperation = leftTopOperation;
                        return new Rectangle(
                            leftTopOperation(lvalue.Left, rvalue.Left),
                            leftTopOperation(lvalue.Top, rvalue.Top),
                            rightBottomOperation(lvalue.Right, rvalue.Right),
                            rightBottomOperation(lvalue.Bottom, rvalue.Bottom)
                        );
                    }

                    public void Add(Rectangle value)
                    {
                        Add(ref this, ref value);
                    }

                    public void Subtract(Rectangle value)
                    {
                        Subtract(ref this, ref value);
                    }

                    public void Multiply(Rectangle value)
                    {
                        Multiply(ref this, ref value);
                    }

                    public void Divide(Rectangle value)
                    {
                        Divide(ref this, ref value);
                    }

                    public void Deflate(Rectangle value)
                    {
                        Deflate(ref this, ref value);
                    }

                    public void Inflate(Rectangle value)
                    {
                        Inflate(ref this, ref value);
                    }

                    public void Offset(int x, int y)
                    {
                        Offset(ref this, x, y);
                    }

                    public void OffsetTo(int x, int y)
                    {
                        OffsetTo(ref this, x, y);
                    }

                    public void Scale(int x, int y)
                    {
                        Scale(ref this, x, y);
                    }

                    public void ScaleTo(int x, int y)
                    {
                        ScaleTo(ref this, x, y);
                    }

                    public static void Add(ref Rectangle lvalue, ref Rectangle rvalue)
                    {
                        lvalue.Left += rvalue.Left;
                        lvalue.Top += rvalue.Top;
                        lvalue.Right += rvalue.Right;
                        lvalue.Bottom += rvalue.Bottom;
                    }

                    public static void Subtract(ref Rectangle lvalue, ref Rectangle rvalue)
                    {
                        lvalue.Left -= rvalue.Left;
                        lvalue.Top -= rvalue.Top;
                        lvalue.Right -= rvalue.Right;
                        lvalue.Bottom -= rvalue.Bottom;
                    }

                    public static void Multiply(ref Rectangle lvalue, ref Rectangle rvalue)
                    {
                        lvalue.Left *= rvalue.Left;
                        lvalue.Top *= rvalue.Top;
                        lvalue.Right *= rvalue.Right;
                        lvalue.Bottom *= rvalue.Bottom;
                    }

                    public static void Divide(ref Rectangle lvalue, ref Rectangle rvalue)
                    {
                        lvalue.Left /= rvalue.Left;
                        lvalue.Top /= rvalue.Top;
                        lvalue.Right /= rvalue.Right;
                        lvalue.Bottom /= rvalue.Bottom;
                    }

                    public static void Deflate(ref Rectangle target, ref Rectangle deflation)
                    {
                        target.Top += deflation.Top;
                        target.Left += deflation.Left;
                        target.Bottom -= deflation.Bottom;
                        target.Right -= deflation.Right;
                    }

                    public static void Inflate(ref Rectangle target, ref Rectangle inflation)
                    {
                        target.Top -= inflation.Top;
                        target.Left -= inflation.Left;
                        target.Bottom += inflation.Bottom;
                        target.Right += inflation.Right;
                    }

                    public static void Offset(ref Rectangle target, int x, int y)
                    {
                        target.Top += y;
                        target.Left += x;
                        target.Bottom += y;
                        target.Right += x;
                    }

                    public static void OffsetTo(ref Rectangle target, int x, int y)
                    {
                        var width = target.Width;
                        var height = target.Height;
                        target.Left = x;
                        target.Top = y;
                        target.Right = width;
                        target.Bottom = height;
                    }

                    public static void Scale(ref Rectangle target, int x, int y)
                    {
                        target.Top *= y;
                        target.Left *= x;
                        target.Bottom *= y;
                        target.Right *= x;
                    }

                    public static void ScaleTo(ref Rectangle target, int x, int y)
                    {
                        unchecked
                        {
                            x = (int) (target.Left / x);
                            y = (int) (target.Top / y);
                        }

                        Scale(ref target, x, y);
                    }
                }

                [DllImport(LibraryName, SetLastError = true)]
                [return: MarshalAs(UnmanagedType.Bool)]
                public static extern bool GetCursorPos(out Point lpPoint);

                [StructLayout(LayoutKind.Sequential)]
                public struct Point
                {
                    public int X;
                    public int Y;

                    public Point(int x, int y)
                    {
                        this.X = x;
                        this.Y = y;
                    }

                    public static implicit operator System.Drawing.Point(Point p)
                    {
                        return new System.Drawing.Point(p.X, p.Y);
                    }

                    public static implicit operator Point(System.Drawing.Point p)
                    {
                        return new Point(p.X, p.Y);
                    }
                }
            }
        }
```

如果发现你的代码依然无法构建通过，还请参阅我的测试代码从里面抄代码解决找不到某个类

本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/a4e6bdc2a18cddea7defa4abcf461629e30763d2/BeehijemwaboHaihafobe)欢迎小伙伴访问

关于 Win32 方法的定义，我推荐使用官方的 [dotnet/pinvoke: A library containing all P/Invoke code so you don't have to import it every time. Maintained and updated to support the latest Windows OS.](https://github.com/dotnet/pinvoke ) 库

参考

[WPF Touch DragMove() - CodeProject](https://www.codeproject.com/Questions/669512/WPF-Touch-DragMove )

[[Solved] How to drag window with finger not mouse - CodeProject](https://www.codeproject.com/Questions/671379/How-to-drag-window-with-finger-not-mouse )

[Way to make a Windowless WPF window draggable without getting InvalidOperationException - Stack Overflow](https://stackoverflow.com/questions/3274097/way-to-make-a-windowless-wpf-window-draggable-without-getting-invalidoperationex/3275712#3275712 )

[Can only call DragMove when primary mouse button is down.](https://social.msdn.microsoft.com/Forums/vstudio/en-US/975aa7ec-8558-4dcc-90b5-ca12555faa34/can-only-call-dragmove-when-primary-mouse-button-is-down?forum=wpf )

[microsoft/XamlBehaviorsWpf: Home for WPF XAML Behaviors on GitHub.](https://github.com/microsoft/XamlBehaviorsWpf )






<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。