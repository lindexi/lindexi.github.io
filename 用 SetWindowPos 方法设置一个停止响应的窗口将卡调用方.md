# 用 SetWindowPos 方法设置一个停止响应的窗口将卡调用方

我使用 User32 的 SetWindowPos 方法去设置一个跨进程的窗口，这个窗口是停止响应的，将让调用的 SetWindowPos 方法卡住，不继续执行逻辑。通过堆栈分析是卡在 NtUserSetWindowPos 方法上，调用 SetWindowPos 方法不返回

<!--more-->
<!-- CreateTime:2022/5/16 8:49:51 -->

<!-- 发布 -->

原本我以为调用 User32 里面的函数，大部分都是很十分快速返回的。刚好今天遇到了测试告诉我应用没响应，这是一个多进程模型的应用。刚好 [lsj](https://blog.sdlsj.net) 修好了 [dnSpy](https://blog.lindexi.com/post/%E6%94%AF%E6%8C%81-dotnet-6-%E7%9A%84-dnSpy-%E7%A5%9E%E5%99%A8%E7%89%88%E6%9C%AC.html) 在 dotnet 6 的调试，于是我就在测试小姐姐那里用 dnSpy 挂上调试

然而我看到了在主应用里面，没有响应的原因是主线程在等待 User32.dll 的 SetWindowPos 方法返回。开始我以为又是某数字杀毒软件干的，虽然没有啥理由，但某数字杀毒软件就是专门用来背锅的

过了几天，在服务器上又有另外一个应用未响应，通过抓 DUMP 回来分析，居然也是主线程在等待 SetWindowPos 方法返回

于是我就开始调查为什么 SetWindowPos 这样的方法能不返回，理论上这个方法不就是设置某个窗口的坐标和宽度高度等信息的？十分简单的一个方法

询问了一圈了解到，其实这个方法不返回的一个可能的原因是，如果设置的窗口没有处理 Windows 消息，那此 SetWindowPos 方法将不返回。也就是说阻塞 SetWindowPos 方法的其中一个原因就是和 SendMessage 一样，如果对应的窗口的 Windows 消息没有被读取，那么调用方将被阻塞

重新等待下一次复现。经过调试发现出现问题的时候，采用 SetWindowPos 设置的窗口句柄确实是属于另一个进程的窗口，而另一个进程刚好也是处于无响应的状态。也就是说本质原因是另一个进程无响应，导致了当前进程通过 SetWindowPos 设置另一个进程的窗口，由于另一个进程无响应，没有处理 Windows 消息，从而让当前进程阻塞也无响应

学到的知识： 如果某个应用调用 SetWindowPos 方法阻塞，那么优先调试调用 SetWindowPos 方法传入的窗口句柄参数，通过窗口句柄寻找对应的进程，调查对应的进程是否无响应或者窗口所在的线程没有继续处理 Windows 消息

那为什么 SetWindowPos 的行为和 SendMessage 如此相同？我请教了 [lsj](https://blog.sdlsj.net) 这个问题，经过 [lsj](https://blog.sdlsj.net) 阅读了 XP 的部分代码，找到了在系统底层里面，在 SetWindowPos 方法的实现里面就调用了 SendMessage 方法。因此 SetWindowPos 卡住的一个原因就如 SendMessage 的原因，要求只有在对方处理了消息才返回

我写了一个简单的 demo 来复现此问题

先创建两个项目，其中一个项目是 WpfApp1 项目，这个项目的功能是在点击按钮时，让主线程卡住，也就是让 UI 线程不处理 Windows 消息，模拟一个未响应进程

在 WpfApp1 项目的 MainWindow.xaml 上放一个按钮，这个按钮就是点击的时候，执行逗比逻辑，卡住主 UI 线程

```xml
    <Grid>
        <Button HorizontalAlignment="Center" VerticalAlignment="Center" Click="Button_Click">卡</Button>
    </Grid>
```

在后台代码编写 Button_Click 方法，我执行的是 Thread.Sleep 的逻辑，让 UI 线程不断执行，而无法处理 Windows 消息

```csharp
    private void Button_Click(object sender, RoutedEventArgs e)
    {
        while (true)
        {
            Thread.Sleep(TimeSpan.FromSeconds(1));
        }
    }
```

为什么这段逗比代码要用 `while (true)` 来做？因为我期望可以通过 VisualStudio 断点调试，跳出循环，也就是让 WpfApp1 进程继续处理 Windows 消息

再新建一个叫 NawnayarlallliwurHifowaleeli 的项目，在这个项目尝试去获取 WpfApp1 进程的 MainWindow 且调用 SetWindowPos 方法设置 WpfApp1 进程的 MainWindow 的坐标

为了方便调用 SetWindowPos 方法，我采用了 dotnet 官方开源的 P/Invoke 库，详细请看 [https://github.com/dotnet/pinvoke](https://github.com/dotnet/pinvoke)

在 NawnayarlallliwurHifowaleeli 项目，尝试不断设置 WpfApp1 进程的 MainWindow 的坐标。在没有让 WpfApp1 进程卡主线程时，预期是能成功设置且快速返回

```csharp
using PInvoke;

using System;
using System.Diagnostics;
using System.Linq;
using System.Threading.Tasks;
using System.Windows;

namespace NawnayarlallliwurHifowaleeli;

/// <summary>
/// Interaction logic for MainWindow.xaml
/// </summary>
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();

        Loaded += MainWindow_Loaded;
    }

    private async void MainWindow_Loaded(object sender, RoutedEventArgs e)
    {
        var process = Process.GetProcessesByName("WpfApp1").First();

        if (process.MainWindowTitle == "MainWindow")
        {
            while (_contentLoaded)
            {
                User32.SetWindowPos
                (
                    hWnd: process.MainWindowHandle,
                    hWndInsertAfter: IntPtr.Zero,
                    X: Random.Shared.Next(5),
                    Y: Random.Shared.Next(5),
                    cx: 0,
                    cy: 0,
                    uFlags: User32.SetWindowPosFlags.SWP_NOSIZE
                    | User32.SetWindowPosFlags.SWP_NOZORDER
                    | User32.SetWindowPosFlags.SWP_NOACTIVATE
                );
                await Task.Delay(TimeSpan.FromMilliseconds(500));
            }
        }
    }
}
```

为了测试 NawnayarlallliwurHifowaleeli 进程是否进入无响应，也在 NawnayarlallliwurHifowaleeli 的 MainWindow 上放一个按钮，通过鼠标移动到按钮上的效果，即可了解窗口是否无响应

```xml
    <Grid>
        <Button Margin="10,10,10,10"></Button>
    </Grid>
```

跑起来两个项目的进程，可以看到 WpfApp1 的窗口不断被 NawnayarlallliwurHifowaleeli 设置窗口坐标，开始跳 disco 起来

接下来点击 WpfApp1 的按钮让 WpfApp1 进程无响应。可以看到 NawnayarlallliwurHifowaleeli 进程也跟着无响应起来

在 VisualStudio 里面勾选 Native 调试（本机调试，可以调试非托管部分）可以看到 NawnayarlallliwurHifowaleeli 进程是卡在调用 SetWindowPos 方法，如何预期

以下就是 NawnayarlallliwurHifowaleeli 的调用堆栈

```
 	win32u.dll!NtUserSetWindowPos()	未知
 	[托管到本机的转换]	
	NawnayarlallliwurHifowaleeli.dll!NawnayarlallliwurHifowaleeli.MainWindow.MainWindow_Loaded(object sender, System.Windows.RoutedEventArgs e) 行 31	C#
 	[正在恢复异步方法]	
 	System.Private.CoreLib.dll!System.Runtime.CompilerServices.AsyncTaskMethodBuilder<System.Threading.Tasks.VoidTaskResult>.AsyncStateMachineBox<System.__Canon>.ExecutionContextCallback(object s)	未知
 	System.Private.CoreLib.dll!System.Threading.ExecutionContext.RunInternal(System.Threading.ExecutionContext executionContext, System.Threading.ContextCallback callback, object state)	未知
 	System.Private.CoreLib.dll!System.Runtime.CompilerServices.AsyncTaskMethodBuilder<System.Threading.Tasks.VoidTaskResult>.AsyncStateMachineBox<NawnayarlallliwurHifowaleeli.MainWindow.<MainWindow_Loaded>d__1>.MoveNext(System.Threading.Thread threadPoolThread)	未知
 	System.Private.CoreLib.dll!System.Runtime.CompilerServices.AsyncTaskMethodBuilder<System.Threading.Tasks.VoidTaskResult>.AsyncStateMachineBox<System.__Canon>.MoveNext()	未知
 	System.Private.CoreLib.dll!System.Runtime.CompilerServices.TaskAwaiter.OutputWaitEtwEvents.AnonymousMethod__12_0(System.Action innerContinuation, System.Threading.Tasks.Task innerTask)	未知
 	System.Private.CoreLib.dll!System.Runtime.CompilerServices.AsyncMethodBuilderCore.ContinuationWrapper.Invoke()	未知
 	System.Private.CoreLib.dll!System.Threading.Tasks.SynchronizationContextAwaitTaskContinuation.GetActionLogDelegate.AnonymousMethod__0()	未知
 	System.Private.CoreLib.dll!System.Threading.Tasks.SynchronizationContextAwaitTaskContinuation..cctor.AnonymousMethod__8_0(object state)	未知
 	WindowsBase.dll!System.Windows.Threading.ExceptionWrapper.InternalRealCall(System.Delegate callback, object args, int numArgs)	未知
 	WindowsBase.dll!System.Windows.Threading.ExceptionWrapper.TryCatchWhen(object source, System.Delegate callback, object args, int numArgs, System.Delegate catchHandler)	未知
 	WindowsBase.dll!System.Windows.Threading.DispatcherOperation.InvokeImpl()	未知
 	WindowsBase.dll!MS.Internal.CulturePreservingExecutionContext.CallbackWrapper(object obj)	未知
 	System.Private.CoreLib.dll!System.Threading.ExecutionContext.RunInternal(System.Threading.ExecutionContext executionContext, System.Threading.ContextCallback callback, object state)	未知
 	WindowsBase.dll!MS.Internal.CulturePreservingExecutionContext.Run(MS.Internal.CulturePreservingExecutionContext executionContext, System.Threading.ContextCallback callback, object state)	未知
 	WindowsBase.dll!System.Windows.Threading.DispatcherOperation.Invoke()	未知
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.ProcessQueue()	未知
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.WndProcHook(System.IntPtr hwnd, int msg, System.IntPtr wParam, System.IntPtr lParam, ref bool handled)	未知
 	WindowsBase.dll!MS.Win32.HwndWrapper.WndProc(System.IntPtr hwnd, int msg, System.IntPtr wParam, System.IntPtr lParam, ref bool handled)	未知
 	WindowsBase.dll!MS.Win32.HwndSubclass.DispatcherCallbackOperation(object o)	未知
 	WindowsBase.dll!System.Windows.Threading.ExceptionWrapper.InternalRealCall(System.Delegate callback, object args, int numArgs)	未知
 	WindowsBase.dll!System.Windows.Threading.ExceptionWrapper.TryCatchWhen(object source, System.Delegate callback, object args, int numArgs, System.Delegate catchHandler)	未知
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.LegacyInvokeImpl(System.Windows.Threading.DispatcherPriority priority, System.TimeSpan timeout, System.Delegate method, object args, int numArgs)	未知
 	WindowsBase.dll!MS.Win32.HwndSubclass.SubclassWndProc(System.IntPtr hwnd, int msg, System.IntPtr wParam, System.IntPtr lParam)	未知
 	[本机到托管的转换]	
 	user32.dll!00007ffe839ce858()	未知
 	user32.dll!00007ffe839ce299()	未知
 	[托管到本机的转换]	
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.PushFrameImpl(System.Windows.Threading.DispatcherFrame frame)	未知
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.PushFrame(System.Windows.Threading.DispatcherFrame frame)	未知
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.Run()	未知
 	PresentationFramework.dll!System.Windows.Application.RunDispatcher(object ignore)	未知
 	PresentationFramework.dll!System.Windows.Application.RunInternal(System.Windows.Window window)	未知
 	PresentationFramework.dll!System.Windows.Application.Run()	未知
 	NawnayarlallliwurHifowaleeli.dll!NawnayarlallliwurHifowaleeli.App.Main()	未知
 	[本机到托管的转换]	
 	[内联框架] hostpolicy.dll!coreclr_t::execute_assembly(int) 行 89	C++
 	hostpolicy.dll!run_app_for_context(const hostpolicy_context_t & context, int argc, const wchar_t * * argv) 行 255	C++
 	hostpolicy.dll!run_app(const int argc, const wchar_t * * argv) 行 284	C++
 	hostpolicy.dll!corehost_main(const int argc, const wchar_t * * argv) 行 430	C++
 	hostfxr.dll!execute_app(const std::wstring & impl_dll_dir, corehost_init_t * init, const int argc, const wchar_t * * argv) 行 146	C++
 	hostfxr.dll!`anonymous namespace'::read_config_and_execute(const std::wstring & host_command, const host_startup_info_t & host_info, const std::wstring & app_candidate, const std::unordered_map<enum known_options,std::vector<std::wstring,std::allocator<std::wstring>>,known_options_hash,std::equal_to<enum known_options>,std::allocator<std::pair<enum known_options const ,std::vector<std::wstring,std::allocator<std::wstring>>>>> & opts, int new_argc, const wchar_t * * new_argv, host_mode_t mode, const bool is_sdk_command, wchar_t * out_buffer, int buffer_size, int * required_buffer_size) 行 533	C++
 	hostfxr.dll!fx_muxer_t::handle_exec_host_command(const std::wstring & host_command, const host_startup_info_t & host_info, const std::wstring & app_candidate, const std::unordered_map<enum known_options,std::vector<std::wstring,std::allocator<std::wstring>>,known_options_hash,std::equal_to<enum known_options>,std::allocator<std::pair<enum known_options const ,std::vector<std::wstring,std::allocator<std::wstring>>>>> & opts, int argc, const wchar_t * * argv, int argoff, host_mode_t mode, const bool is_sdk_command, wchar_t * result_buffer, int buffer_size, int * required_buffer_size) 行 1018	C++
 	hostfxr.dll!fx_muxer_t::execute(const std::wstring host_command, const int argc, const wchar_t * * argv, const host_startup_info_t & host_info, wchar_t * result_buffer, int buffer_size, int * required_buffer_size) 行 579	C++
 	hostfxr.dll!hostfxr_main_startupinfo(const int argc, const wchar_t * * argv, const wchar_t * host_path, const wchar_t * dotnet_root, const wchar_t * app_path) 行 61	C++
 	NawnayarlallliwurHifowaleeli.exe!00007ff77e5524b8()	未知
 	NawnayarlallliwurHifowaleeli.exe!00007ff77e55282b()	未知
 	NawnayarlallliwurHifowaleeli.exe!00007ff77e553cd8()	未知
 	kernel32.dll!BaseThreadInitThunk()	未知
 	ntdll.dll!RtlUserThreadStart()	未知
```

本文的测试代码放在[github](https://github.com/lindexi/lindexi_gd/tree/72ec5a3dc9c43662d6f7cce7b676ef7bc5488f44/NawnayarlallliwurHifowaleeli) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/72ec5a3dc9c43662d6f7cce7b676ef7bc5488f44/NawnayarlallliwurHifowaleeli) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 72ec5a3dc9c43662d6f7cce7b676ef7bc5488f44
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 NawnayarlallliwurHifowaleeli 文件夹

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
