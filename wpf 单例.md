# wpf 单例

本文告诉大家如何做一个 wpf 单例程序。单例就是用户只能运行这个程序一次，也就是内存只有存在这个程序一个。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:53 -->


<!-- csdn -->

首先创建项目，右击 App.xaml.cs 选择编译为 Page ，原来是 ApplicationDefinition

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20171019201545.jpg)

打开 App.xaml 删除`StartupUri="MainWindow.xaml"`，不然下面的代码会启动两个窗口

然后创建 Main ，请添加下面代码

```csharp
    public partial class App : Application
    {
        App()
        {
            InitializeComponent();
        }

        [STAThread]
        static void Main()
        {
            App app = new App();
            MainWindow window = new MainWindow();
            app.Run(window);
        }
    }
```

然后就在 Main 写逻辑，这里判断是否有全局锁，如果有就不启动。添加锁的代码很简单。

```csharp
static Mutex mutex = new Mutex(true, "lindexi");
```

如果程序没有启动，那么通过下面的代码判断现在只有他一个软件

```csharp
if (_mutex.WaitOne(TimeSpan.Zero, true))
```

现在的主函数就是判断可以启动就创建窗口

```csharp
        static void Main()
        {
            App app = new App();
            if (_mutex.WaitOne(TimeSpan.Zero, true))
            {
                //如果申请成功
                MainWindow window = new MainWindow();
                app.Run(window);
            }
        }
```

但是如果不能启动，就需要启动原来启动的程序。这里使用一个特殊方法，先创建一个类，可以发送消息给原先的程序。需要知道，在windows上，程序和系统通信都是使用消息，通过模拟消息就可以让其他程序做出特殊的效果，如让他放在窗口最前。

如果需要发送消息，那么需要使用下面的方法。下面代码注册了一个自己的消息，这样在自己的程序可以判断`WM_SHOWME`来把自己给用户。

```csharp
    internal class NativeMethods
    {
        public const int HWND_BROADCAST = 0xffff;
        public static readonly int WM_SHOWME = RegisterWindowMessage("WM_SHOWME");
        [DllImport("user32")]
        public static extern bool PostMessage(IntPtr hwnd, int msg, IntPtr wparam, IntPtr lparam);
        [DllImport("user32")]
        public static extern int RegisterWindowMessage(string message);
    }
```

接着修改主函数，在已经启动一个程序，就让他打开。这个方法就是发送一个特殊的消息，只有自己的程序知道这个消息是显示。

```csharp
        static void Main()
        {
            App app = new App();
            if (_mutex.WaitOne(TimeSpan.Zero, true))
            {
                //如果申请成功
                MainWindow window = new MainWindow();
                app.Run(window);
                _mutex.ReleaseMutex();
            }
            else
            {
                NativeMethods.PostMessage(
                    (IntPtr) NativeMethods.HWND_BROADCAST,
                    NativeMethods.WM_SHOWME,
                    IntPtr.Zero,
                    IntPtr.Zero);
            }
        }
```

然后发送消息需要让程序自己监听，于是打开 MainWindow.xaml.cs 添加下面的代码，如果监听消息是让自己显示，那么就打开自己。

```csharp
        protected override void OnSourceInitialized(EventArgs e)
        {
            base.OnSourceInitialized(e);
            HwndSource source = PresentationSource.FromVisual(this) as HwndSource;
            source.AddHook(WndProc);
        }

        private IntPtr WndProc(IntPtr hwnd, int msg, IntPtr wparam, IntPtr lparam, ref bool handled)
        {
            if (msg == NativeMethods.WM_SHOWME)
            {
                Activate();
            }
            return IntPtr.Zero;
        }
```

这样就可以运行一个程序，再次打开也会打开原来的程序。

但是程序有一个坑，如果是多用户，那么打开只能是一个程序，请看下面的代码。

```csharp
        private static Mutex _mutex = new Mutex(true, "lindexi" + Environment.UserName);

```

实际文章就想说如何在多用户系统使用单例。

参见：[Writing a custom Main() method for WPF applications — The Stochastic Game](https://ludovic.chabant.com/devblog/2010/04/20/writing-a-custom-main-method-for-wpf-applications/ )

[C# .NET Single Instance Application - Sanity Free Coding - C#, .NET, PHP](http://sanity-free.org/143/csharp_dotnet_single_instance_application.html )

代码：[wpf 单例 PfalmcYmgtx-CSDN下载](http://download.csdn.net/download/lindexi_gd/10030684 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  