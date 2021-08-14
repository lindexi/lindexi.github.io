# WPF 基于 WER 注册应用崩溃无响应回调和重启方法

本文来告诉大家如何在 Windows 上利用从 Vista 引入的 Windows Error Reporting (WER) 机制来实现，在应用崩溃、无响应等异常的时候收到回调用于处理信息保存

<!--more-->
<!-- 发布 -->

在 《Application Recovery and Restart Reference》 里可以了解到可以通过 Application Recovery and Restart (ARR) 技术，在应用崩溃的时候，有时机可以保存应用的信息。例如做一个类似 Office 的 PPT 的软件，可以在此软件在崩溃的时候，依然有时机可以保存用户的文档信息。从而实现尽可能不会因为软件崩溃而丢失信息

在开始之前，先来做一个演示。如下应用将因为写了逗比代码而无响应，在弹出 WER 时，可以让用户选择重启或退出等。无论选择什么，都可以让应用有机会弹出 `应用程序炸掉` 提示。换句话说，可以有时机弹出提示，也就是相当于可以做很多保存信息的逻辑，或者说上报的动作，或者制作 DUMP 文件同时上传等

如果用户选择重启的话，还可以在重启的时候将命令行参数发送到重启的应用里面，这样就可以实现在重启的应用里面继续上一个应用的逻辑。例如做的是类似 PPT 的软件，那么在重启之后，依然可以打开原先的文档。这样可以在尽可能在软件没有做好的时候，让用户减少砸桌子


<!-- ![](image/WPF 基于 WER 注册应用崩溃无响应回调和重启方法/WPF 基于 WER 注册应用崩溃无响应回调和重启方法0.gif) -->

![](http://image.acmx.xyz/lindexi%2FWPF%2520%25E5%259F%25BA%25E4%25BA%258E%2520WER%2520%25E6%25B3%25A8%25E5%2586%258C%25E5%25BA%2594%25E7%2594%25A8%25E5%25B4%25A9%25E6%25BA%2583%25E6%2597%25A0%25E5%2593%258D%25E5%25BA%2594%25E5%259B%259E%25E8%25B0%2583%25E5%2592%258C%25E9%2587%258D%25E5%2590%25AF%25E6%2596%25B9%25E6%25B3%25950.gif)


以下是这个逗比应用的代码

```csharp
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();

            if (Environment.GetCommandLineArgs().Contains("重启信息"))
            {
                MessageBox.Show("重启");
            }

            Loaded += MainWindow_Loaded;
        }

        private async void MainWindow_Loaded(object sender, RoutedEventArgs e)
        {
            var hr = ApplicationCrashRecovery.RegisterApplicationRestart("重启信息", CrashRecoveryApi.RestartFlags.NONE);

            ApplicationCrashRecovery.RegisterApplicationRecoveryCallback();
            ApplicationCrashRecovery.OnApplicationCrashed += ApplicationCrashRecovery_OnApplicationCrashed;

            await Task.Delay(TimeSpan.FromSeconds(3));

            Thread.Sleep(1000000000);
        }

        private void ApplicationCrashRecovery_OnApplicationCrashed()
        {
            // [RegisterApplicationRecoveryCallback function (winbase.h) - Win32 apps | Microsoft Docs](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-registerapplicationrecoverycallback )
            //  By default, the interval is 5 seconds (RECOVERY_DEFAULT_PING_INTERVAL). The maximum interval is 5 minutes. 
            MessageBox.Show("应用程序炸掉");
        }
    }
```

在启动的时候判断是否有命令行，有的话，就显示命令行的内容。通过以上可以证明可以被用户重启

另外，在 `ApplicationCrashRecovery_OnApplicationCrashed` 可以弹出提示，证明应用是可以记录更多信息。大概在进入此方法还能使用 5 分钟最多。如果是期望记录 DUMP 文件，可以尝试通过跨进程调用的方法，调用另一个进程辅助记录

本文核心是通过 ARR 的辅助方法，这几个 API 都是 Win32 的方法，可以使用如下代码进行引用

```csharp
    public class CrashRecoveryApi
    {
        [Flags]
        public enum RestartFlags
        {
            NONE = 0,
            RESTART_NO_CRASH = 1,
            RESTART_NO_HANG = 2,
            RESTART_NO_PATCH = 4,
            RESTART_NO_REBOOT = 8
        }

        [DllImport("kernel32.dll", CharSet = CharSet.Unicode)]
        internal static extern int RegisterApplicationRestart(string pwsCommandLine, RestartFlags dwFlags);

        [DllImport("kernel32.dll")]
        internal static extern uint RegisterApplicationRecoveryCallback(IntPtr pRecoveryCallback, IntPtr pvParameter, int dwPingInterval, int dwFlags);

        [DllImport("kernel32.dll")]
        internal static extern uint ApplicationRecoveryInProgress(out bool pbCancelled);

        [DllImport("kernel32.dll")]
        internal static extern uint ApplicationRecoveryFinished(bool bSuccess);

        [DllImport("kernel32.dll")]
        internal static extern uint UnregisterApplicationRecoveryCallback();

        [DllImport("kernel32.dll")]
        internal static extern uint UnregisterApplicationRestart();
    }
```

以上代码的 ApplicationCrashRecovery 是进一步的封装，如下面代码

```csharp
    public class ApplicationCrashRecovery
    {
        #region Delegates & Events
        private delegate int ApplicationRecoveryCallback(IntPtr pvParameter);
        public delegate void ApplicationCrashHandler();

        /// <summary>
        /// 监测到发生异常(包括崩溃、卡顿无响应等异常)
        /// </summary>
        public static event ApplicationCrashHandler OnApplicationCrashed;
        #endregion

        private static ApplicationRecoveryCallback _recoverApplication;

        /// <summary>
        /// 向WER注册应用程序重启机制
        /// </summary>
        /// <param name="pwsCommandLine">重启命令行参数</param>
        /// <param name="restartFlag">重启标志，参考RestartFlags</param>
        /// <returns>0表示成功</returns>
        public static int RegisterApplicationRestart(string pwsCommandLine, CrashRecoveryApi.RestartFlags restartFlag)
        {
            if (!IsWindowsVistaOrLater)
            {
                return -1;
            }
            var result = CrashRecoveryApi.RegisterApplicationRestart(pwsCommandLine, restartFlag);
            return result;
        }

        /// <summary>
        /// Registers the application for notification by windows of a failure.
        /// </summary>
        /// <returns>0 if successfully registered for restart notification</returns>
        public static uint RegisterApplicationRecoveryCallback()
        {
            if (!IsWindowsVistaOrLater)
            {
                return 1;
            }
            //  By default, the interval is 5 seconds (RECOVERY_DEFAULT_PING_INTERVAL). The maximum interval is 5 minutes. 
            _recoverApplication = new ApplicationRecoveryCallback(HandleApplicationCrash);
            IntPtr ptrOnApplicationCrash = Marshal.GetFunctionPointerForDelegate(_recoverApplication);
            var result = CrashRecoveryApi.RegisterApplicationRecoveryCallback(ptrOnApplicationCrash, IntPtr.Zero, (int)TimeSpan.FromMinutes(5).TotalMilliseconds,
                0);
            return result;
        }

        /// <returns>0 if successfully unRegistered for restart notification</returns>  
        public static uint UnRegisterForRestart()
        {
            if (!IsWindowsVistaOrLater)
            {
                return 1;
            }
            CrashRecoveryApi.UnregisterApplicationRecoveryCallback();
            var result = CrashRecoveryApi.UnregisterApplicationRestart();
            return result;
        }

        #region Data Persistance Methods
        private static bool IsWindowsVistaOrLater => Environment.OSVersion.Version.Major >= 6;

        /// <summary>
        /// This is the callback function that is executed in the event of the application crashing.
        /// </summary>
        /// <param name="pvParameter"></param>
        /// <returns></returns>
        private static int HandleApplicationCrash(IntPtr pvParameter)
        {
            //Allow the user to cancel the recovery.  The timer polls for that cancel.
            using (System.Threading.Timer t = new System.Threading.Timer(CheckForRecoveryCancel, null, 1000, 1000))
            {
                //Handle this event in your own code
                OnApplicationCrashed?.Invoke();

                CrashRecoveryApi.ApplicationRecoveryFinished(true);
            }

            return 0;
        }

        /// <summary>
        /// Checks to see if the user has cancelled the recovery.
        /// </summary>
        /// <param name="o"></param>
        private static void CheckForRecoveryCancel(object o)
        {
            CrashRecoveryApi.ApplicationRecoveryInProgress(out var userCancelled);

            if (userCancelled)
            {
                Environment.FailFast("User cancelled application recovery");
            }
        }
        #endregion
    }
```

以上代码也是我从旧项目抄的，也许这个代码也不知道是从哪里抄的，但是大概是可以使用的

此方法的缺点在于如果用户的设备上没有关闭了 WER 那么将无法工作

本文所有代码在 [github](https://github.com/lindexi/lindexi_gd/tree/4412141e79589e436699f1c84326292afeb273c1/LabujeeneferejurFeqawjewur) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/4412141e79589e436699f1c84326292afeb273c1/LabujeeneferejurFeqawjewur) 上完全开源

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 4412141e79589e436699f1c84326292afeb273c1
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 LabujeeneferejurFeqawjewur 文件夹

详细请看 [Application Recovery and Restart - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/recovery/application-recovery-and-restart-portal?WT.mc_id=WD-MVP-5003260 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
