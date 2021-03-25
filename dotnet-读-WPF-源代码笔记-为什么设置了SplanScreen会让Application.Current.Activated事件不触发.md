
# dotnet 读 WPF 源代码笔记 为什么设置了SplanScreen会让Application.Current.Activated事件不触发

在 WPF 应用中，可以非常方便将一张图片设置为 SplanScreen 启动界面欢迎图，但是如果有设置了启动界面欢迎界面，那么 Application.Current.Activated 事件就不会被触发。本文通过 WPF 框架开源的代码告诉大家这个原因

<!--more-->


<!-- 发布 -->
<!-- 标签：WPF，WPF源代码 -->

这是在 GitHub 上，一个小伙伴问的问题，详细请看 [After adding a splashscreen Application.Current.Activated event is no longer fired · Issue #4316 · dotnet/wpf](https://github.com/dotnet/wpf/issues/4316 )

设置某个图片作为 SplanScreen 启动图的方式很简单，只需要右击图片，设置属性，选择 SplanScreen 就可以。也可以在 csproj 中添加如下代码设置

```xml
  <ItemGroup>
    <SplashScreen Include="SplanScreen.png" />
  </ItemGroup>
```

尝试在 App 的构造函数里面添加如下代码用来监听 Activated 事件

```csharp
    public partial class App : Application
    {
        public App()
        {
            Current.Activated += Current_Activated;
        }

        private void Current_Activated(object sender, EventArgs e)
        {
        }
    }
```

原因是在设置 SplanScreen 启动界面，那么在生成的 App.g.cs 文件里面将会包含下面代码

```csharp
        [System.STAThreadAttribute()]
        [System.Diagnostics.DebuggerNonUserCodeAttribute()]
        [System.CodeDom.Compiler.GeneratedCodeAttribute("PresentationBuildTasks", "5.0.1.0")]
        public static void Main() 
        {
            SplashScreen splashScreen = new SplashScreen("splanscreen.png");
            splashScreen.Show(true);
            App app = new App();
            app.InitializeComponent();
            app.Run();
        }
```

也就是说 SplashScreen 将会在 Main 函数里面最开始就执行，因此启动图显示的速度也比较快。在 SplashScreen 显示完成之后，再创建 App 出来，也就是说监听 Activated 事件是在启动图之后

那么 Activated 事件是由谁分发的？在 `src\Microsoft.DotNet.Wpf\src\PresentationFramework\System\Windows\Application.cs` 的 EnsureHwndSource 函数里面将是入口代码，而在 WmActivateApp 函数就是触发的逻辑，先看一下 WmActivateApp 的代码

```csharp
        private bool WmActivateApp(Int32 wParam)
        {
            int temp = wParam;
            bool isActivated = (temp == 0? false : true);

            // Event handler exception continuality: if exception occurs in Activate/Deactivate event handlers, our state would not
            // be corrupted because no internal state are affected by Activate/Deactivate. Please check Event handler exception continuality
            // if a state depending on those events is added.
            if (isActivated == true)
            {
                OnActivated(EventArgs.Empty);
            }
            else
            {
                OnDeactivated(EventArgs.Empty);
            }
            return false;
        }
```

也就是说调用进入 WmActivateApp 的参数将决定是否调用 OnActivated 函数，在 OnActivated 函数里面就是事件触发

```csharp
        protected virtual void OnActivated(EventArgs e)
        {
            VerifyAccess();
            if (Activated != null)
            {
                Activated(this, e);
            }
        }
```

而 WmActivateApp 从函数名就可以看出，这是一个由 Win32 的 Windows 消息触发的方法，在 AppFilterMessage 函数里面将会调用到 WmActivateApp 方法。而 AppFilterMessage 函数的命名意思是 `App` 类的 FilterMessage 方法，也就是说这是一个处理应用级的 Windows 消息的函数，代码如下

```csharp
        private IntPtr AppFilterMessage(IntPtr hwnd, int msg, IntPtr wParam, IntPtr lParam, ref bool handled)
        {
            IntPtr retInt = IntPtr.Zero;
            switch ((WindowMessage)msg)
            {
                case WindowMessage.WM_ACTIVATEAPP:
                    handled = WmActivateApp(NativeMethods.IntPtrToInt32(wParam));
                    break;
                case WindowMessage.WM_QUERYENDSESSION :
                    handled = WmQueryEndSession(lParam, ref retInt);
                    break;
                default:
                    handled = false;
                    break;
            }
            return retInt;
        }
```

这个 AppFilterMessage 方法是在 EnsureHwndSource 函数里面注册消息的，请看代码

```csharp
        private void EnsureHwndSource()
        {
            if (_parkingHwnd == null)
            {
                // _appFilterHook needs to be member variable otherwise
                // it is GC'ed and we don't get messages from HwndWrapper
                // (HwndWrapper keeps a WeakReference to the hook)

                _appFilterHook = new HwndWrapperHook(AppFilterMessage);
                HwndWrapperHook[] wrapperHooks = {_appFilterHook};

                _parkingHwnd = new HwndWrapper(
                                0,
                                0,
                                0,
                                0,
                                0,
                                0,
                                0,
                                "",
                                IntPtr.Zero,
                                wrapperHooks);
            }
        }
```

也就是说 Activated 事件的触发就是依靠 WindowMessage.WM_ACTIVATEAPP 消息的，这个消息详细请看 [WM_ACTIVATEAPP 官方文档](https://docs.microsoft.com/zh-cn/windows/win32/winmsg/wm-activateapp?WT.mc_id=WD-MVP-5003260 )

因为 SplashScreen 本身将会创建窗口，也因为 SplashScreen 的速度足够快，因此在 Application 的 EnsureHwndSource 函数调用之前，系统发送了 WM_ACTIVATEAPP 消息给到应用了

所以在 App 的构造函数监听 Activated 事件将不会收到触发





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。