# dotnet 读 WPF 源代码 Popup 的 StaysOpen 为 false 将会吃掉其他窗口的首次激活

在 WPF 中，使用 Popup 控件，可以设置 StaysOpen 属性来控制是否在 Popup 失去焦点时，也就是点击界面空白处，自动收起 Popup 控件。但如果有两个窗口，在设置 Popup 控件的 StaysOpen 属性为 false 那么将会吃掉在点击其他窗口的第一次交互，如鼠标点击或触摸点击时将不会让本进程的其他窗口 Activate 激活

<!--more-->
<!-- CreateTime:2021/6/7 8:50:52 -->


<!-- 标签：WPF，WPF源代码 -->
<!-- 发布 -->

在 WPF 中，通过 Popup 控件可以方便设置浮出的窗口，本质上 Popup 控件也是一个窗口，只是这是一个特殊的窗口。但是在使用 Popup 控件时，如果通过设置 Popup 控件的 StaysOpen 属性为 false 的方式让 Popup 在点击非 Popup 范围内，包括点击窗口其他空白部分，或者点击其他应用程序或桌面等，自动收起。那么 Popup 将会在点击本进程内的其他窗口时，点击的交互被 Popup 吃掉，而让其他窗口收不到一次交互

行为如下：

假定有两个窗口，其中一个是 MainWindows 主窗口，另一个是用来承载 Popup 的 Window1 窗口。 其中 Windows1 窗口有一个按钮，点击按钮时将会弹出一个 Popup 控件，代码过于简单，我就不将所有代码全部写在博客。所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/8cee55ad8832fa6772bd82ded313681275b1e5f2/JallbacelarlearyaLereyilagawhelna) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/8cee55ad8832fa6772bd82ded313681275b1e5f2/JallbacelarlearyaLereyilagawhelna) 欢迎小伙伴访问

以下是 Windows1 的界面，有一个按钮，和一个 Popup 控件，点击按钮自动弹出 Popup 控件

```xml
    <Grid>
        <Button x:Name="OpenPopupButton" HorizontalAlignment="Center" VerticalAlignment="Center"
                Content="Open Popup" Click="OpenPopupButton_OnClick"></Button>
        <Popup x:Name="Popup" StaysOpen="False" PlacementTarget="{x:Reference OpenPopupButton}">
            <Grid Background="Gray" Width="100" Height="100">
                <TextBlock HorizontalAlignment="Center" VerticalAlignment="Center">The popup</TextBlock>
            </Grid>
        </Popup>
    </Grid>  
```

以下是 Windows1 点击按钮的代码

```csharp
        private void OpenPopupButton_OnClick(object sender, RoutedEventArgs e)
        {
            Popup.IsOpen = true;
        }
```

在 MainWindow 里，在 Loaded 事件里面弹出 Windows1 请看代码

```csharp
        public MainWindow()
        {
            InitializeComponent();

            Loaded += MainWindow_Loaded;
        }

        private void MainWindow_Loaded(object sender, RoutedEventArgs e)
        {
            var window1 = new Window1();
            window1.Show();
        }
```

请运行代码，可以看到打开两个窗口，此时如果点击 MainWindows 那么可以让 MainWindows 获取焦点。接下来请点击 Window1 的空白，然后点击 `Open Popup` 按钮，此时将会弹出 Popup 控件。再点击 MainWindows 的空白，可以看到 MainWindows 只是获取到鼠标按下和抬起事件，但是没有被激活没有获取到焦点，依然焦点是 Windows1 窗口

在 MainWindows 上添加一些代码，这样可以方便在 VisualStudio 的输出窗口里面，看到窗口的各个事件

```csharp
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();

            Loaded += MainWindow_Loaded;
            MouseDown += MainWindow_MouseDown;
            MouseUp += MainWindow_MouseUp;
            Activated += MainWindow_Activated;
            Deactivated += MainWindow_Deactivated;
            LostFocus += MainWindow_LostFocus;
        }

        private void MainWindow_Loaded(object sender, RoutedEventArgs e)
        {
            var window1 = new Window1();
            window1.Show();
        }

        private void MainWindow_MouseUp(object sender, MouseButtonEventArgs e)
        {
            Debug.WriteLine($"MainWindow_MouseUp");
        }

        private void MainWindow_MouseDown(object sender, MouseButtonEventArgs e)
        {
            Debug.WriteLine($"MainWindow_MouseDown");
        }

        private void MainWindow_Activated(object sender, EventArgs e)
        {
            Debug.WriteLine($"MainWindow_Activated");
        }

        private void MainWindow_Deactivated(object sender, EventArgs e)
        {
            Debug.WriteLine($"MainWindow_Deactivated");
        }

        private void MainWindow_LostFocus(object sender, RoutedEventArgs e)
        {
            Debug.WriteLine($"MainWindow_LostFocus");
        }
    }
```

下面来执行以下两个不同的动作，了解一下弹出 Popup 对进程内的其他窗口的行为

动作1的步骤：

- 运行代码，默认焦点是在 Window1 上
- 点击 MainWindow 的空白

此时可以看到 VisualStudio 输出的内容如下

```
MainWindow_Activated
MainWindow_Deactivated

MainWindow_Activated
MainWindow_MouseDown
MainWindow_MouseUp
```

第一次 MainWindow_Activated 和 MainWindow_Deactivated 是在 MainWindows 的 Loaded 弹出 Window1 而激活和失去焦点的

第二次的 MainWindow_Activated 和鼠标按下和抬起是在点击 MainWindow 的空白，这是符合预期的

动作2的步骤：

- 运行代码，默认焦点是在 Window1 上
- 点击 Window1 的 Open Popup 按钮
- 点击 MainWindow 的空白

此时可以看到 VisualStudio 输出的内容如下

```
MainWindow_Activated
MainWindow_Deactivated

MainWindow_MouseDown
MainWindow_MouseUp
```

对比可以了解，在点击 Window1 的 Open Popup 按钮弹出 Popup 控件之后，下一次点击 MainWindow 是不会激活 MainWindow 只是收到鼠标的按下和抬起

那为什么 Popup 会影响进程的其他窗口的行为？下面来阅读 Popup 的源代码

在 Popup 的 OnLostMouseCapture 方法里面，触发的定义如下

```csharp
        static Popup()
        {
            EventManager.RegisterClassHandler(typeof(Popup), Mouse.LostMouseCaptureEvent, new MouseEventHandler(OnLostMouseCapture));

            // 忽略其他代码
        }

        private static void OnLostMouseCapture(object sender, MouseEventArgs e)
        {
            Popup popup = sender as Popup;

            if (!popup.StaysOpen)
            {
                PopupRoot root = popup._popupRoot.Value;

                // Reestablish capture if an element within us lost capture
                // (hence we receive the LostCapture routed event) and capture
                // is not being acquired anywhere else.
                //
                // Note we do not reestablish capture if we are losing capture
                // ourselves.
                bool reestablishCapture = e.OriginalSource != root && Mouse.Captured == null && MS.Win32.SafeNativeMethods.GetCapture() == IntPtr.Zero;

                if(reestablishCapture)
                {
                    popup.EstablishPopupCapture();
                    e.Handled = true;
                }
                else
                {
                    // 忽略其他代码
                }
            }
        }
```

在点击 MainWindow 的空白，将会触发到 Popup 的 OnLostMouseCapture 方法，接着进入 EstablishPopupCapture 方法

```csharp
        private void EstablishPopupCapture(bool isRestoringCapture=false)
        {
            if (!_cacheValid[(int)CacheBits.CaptureEngaged] && (_popupRoot.Value != null) &&
                (!StaysOpen))
            {
                IInputElement capturedElement = Mouse.Captured;
                PopupRoot parentPopupRoot = capturedElement as PopupRoot;
                if (parentPopupRoot != null)
                {
                    if (isRestoringCapture)
                    {
                        // if the other PopupRoot is restoring capture back to this
                        // popup, ignore mouse button events until both buttons have been
                        // released.  Otherwise a mouse click outside a chain of
                        // "nested" popups would dismiss two of them - one on MouseDown
                        // and another on MouseUp.
                        if (Mouse.LeftButton != MouseButtonState.Released ||
                            Mouse.RightButton != MouseButtonState.Released)
                        {
                            _cacheValid[(int)CacheBits.IsIgnoringMouseEvents] = true;
                        }
                    }
                    else
                    {
                        // this is a "nested" popup, invoked while another popup is open.
                        // We need to restore capture to the previous popup root when
                        // we're done
                        ParentPopupRootField.SetValue(this, parentPopupRoot);
                    }

                    // in either case, taking capture away from the other PopupRoot is OK.
                    capturedElement = null;
                }

                if (capturedElement == null)
                {
                    // When the mouse is not already captured, we will consider the following:
                    // In all cases but Modeless, we want the popup and subtree to receive
                    // mouse events and prevent other elements from receiving those messages.
                    Mouse.Capture(_popupRoot.Value, CaptureMode.SubTree);
                    _cacheValid[(int)CacheBits.CaptureEngaged] = true;
                }
            }
        }
```

在 EstablishPopupCapture 方法里面重新调用了 Mouse.Capture 将会让本进程内的其他窗口没有被激活

以上是大琛告诉我的，我只是记录的工具人

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、 使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
