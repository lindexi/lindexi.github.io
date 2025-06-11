---
title: Avalonia 取消窗口关闭导致 Linux 麒麟系统无法关机注销重启
description: 在 Avalonia 的窗口 OnClosing 事件里面，将 WindowClosingEventArgs 的 Cancel 属性赋值为 true 用来取消窗口关闭，此时 Linux 麒麟系统的 ukui 组件在进行关机时，将会在调用 会话管理器 时，收到会话管理器取消了本次操作，进而无法进行关机或注销或重启
tags: Avalonia
category: 
---

<!-- 发布 -->
<!-- 博客 -->

最简复现代码如下

```csharp
public partial class MainWindow : Window
{
    protected override void OnClosing(WindowClosingEventArgs e)
    {
        e.Cancel = true;
        base.OnClosing(e);
    }

    ... // 忽略其他代码
}
```

使用以上代码之后，能够在麒麟带 ukui 桌面环境的设备上，复现出从开始菜单点关机或注销或重启失败问题，且此时通知栏收到会话管理器取消了本次操作的提示。此时依然可以从命令行用 shutdown 命令进行关闭

解决方法是判断当前的关闭原因，如果是应用关闭或系统关闭的话，就不阻止窗口关闭即可，代码如下

```csharp
public partial class MainWindow : Window
{
    protected override void OnClosing(WindowClosingEventArgs e)
    {
        if (e.CloseReason is WindowCloseReason.ApplicationShutdown or WindowCloseReason.OSShutdown)
        {
            // 应用关闭或操作系统关闭时，允许关闭窗口
            // 为什么需要判断 ApplicationShutdown 而不是只判断 OSShutdown 呢？这是因为在 11.3.1 及以前版本，在 Avalonia 底层挖了一个坑，无法区分原因。我将此问题报告给官方，详细请看
            // [Incorrect WindowCloseReason Passed During System Shutdown in Avalonia · Issue #19027 · AvaloniaUI/Avalonia](https://github.com/AvaloniaUI/Avalonia/issues/19027 )
        }
        else
        {
            e.Cancel = true;
        }

        base.OnClosing(e);
    }

    ... // 忽略其他代码
}
```

详细机制原因如下

在 Linux 里，有一个机制叫 xsm （session manager） 会话管理器机制。详细请参阅 <https://linux.die.net/man/1/xsm>

在 Avalonia 里面，通过 `libSM.so.6` 库和 xsm 建立联系。通过 `SmcOpenConnection` 机制建立联系。当系统进行关机或注销或重启时，会进入 SaveYourself 机制回调。在 SaveYourself 里面，通过 libSM 的 SmcInteractRequest 方法传入了 Avalonia 的处理逻辑

在 Avalonia 的处理逻辑里面，先判断了 `AVALONIA_X11_USE_SESSION_MANAGEMENT` 环境变量是否被设置为 0 的值，如果被设置了，则直接退出，不触发业务上的逻辑。这就意味着，如果想要调查是否 Avalonia 应用导致 Linux 麒麟系统无法关机或注销或重启时，可以通过 `export AVALONIA_X11_USE_SESSION_MANAGEMENT=0` 设置环境变量再启动应用来进行测试

再接着触发 ShutdownRequestedEventArgs 参数的 ShutdownRequested 事件。这将在 ClassicDesktopStyleApplicationLifetime 里面进行处理。处理方法是遍历当前所有窗口，尝试将其关闭，如果依然还有存活的窗口，则设置取消应用退出。核心代码如下

```csharp
        private bool DoShutdown(
            ShutdownRequestedEventArgs e,
            bool isProgrammatic,
            bool force = false,
            int exitCode = 0)
        {
            if (!force)
            {
                ShutdownRequested?.Invoke(this, e);

                if (e.Cancel)
                    return false;

                if (_isShuttingDown)
                    throw new InvalidOperationException("Application is already shutting down.");
            }

            _exitCode = exitCode;
            _isShuttingDown = true;
            var shutdownCancelled = false;

            try
            {
                // When an OS shutdown request is received, try to close all non-owned windows. Windows can cancel
                // shutdown by setting e.Cancel = true in the Closing event. Owned windows will be shutdown by their
                // owners.
                foreach (var w in new List<Window>(_windows))
                {
                    if (w.Owner is null)
                    {
                        var ignoreCancel = force || (ShutdownMode == ShutdownMode.OnMainWindowClose && w != MainWindow);
                        w.CloseCore(WindowCloseReason.ApplicationShutdown, isProgrammatic, ignoreCancel);
                    }
                }

                if (!force && Windows.Count > 0)
                {
                    e.Cancel = true; // 核心阻止关闭的代码
                    shutdownCancelled = true;
                    return false;
                }

                var args = new ControlledApplicationLifetimeExitEventArgs(exitCode);
                Exit?.Invoke(this, args);
                _exitCode = args.ApplicationExitCode;                
            }
            finally
            {
                _isShuttingDown = false;

                if (!shutdownCancelled)
                {
                    _cts?.Cancel();
                    _cts = null;
                    Dispatcher.UIThread.InvokeShutdown();
                }
            }

            return true;
        }
```

当 ShutdownRequestedEventArgs 的 Cancel 被设置为 true 时，将传递到 libSM 的 SmcInteractDone 方法，从而通知会话管理器取消了本次操作。代码如下

```csharp
                var e = new ShutdownRequestedEventArgs();

                if (_platform.Options.EnableSessionManagement)
                {
                    ShutdownRequested?.Invoke(this, e);
                }

                SMLib.SmcInteractDone(smcConn, e.Cancel);

                if (e.Cancel)
                {
                    return;
                }

                _saveYourselfPhase = false;

                SMLib.SmcSaveYourselfDone(smcConn, true);
```

如 <https://www.x.org/releases/X11R7.7/doc/libSM/SMlib.html> 文档所示，调用的 SmcInteractDone 方法的签名如下

```
void SmcInteractDone(SmcConn smc_conn, Bool cancel_shutdown);
```

> After interacting with the user (in response to an “Interact” message), you should call SmcInteractDone
>
> smc_conn：The session management connection object.
>
> cancel_shutdown：If True, indicates that the user requests that the entire shutdown be cancelled.
>
> cancel_shutdown：如果是 True 值，表示用户请求取消整个关闭操作

这就意味着只要有窗口没有关闭，则会阻止系统关闭

在麒麟 ukui 底层里面，就依赖了 xsm 机制。当收到了有至少一个应用取消关闭时，将会取消关机等的令牌，从而取消关机等命令

由于 ukui 和 Windows 系统的机制不同，不会再弹出提示窗口说有哪些应用没有退出，是否强制关机。因此会导致用户产生困扰，用户将无法从开始菜单进行关机。我认为更好的解决方法是让 ukui 仿照 Windows 系统的做法，弹出提示窗口，如果用户没有主动取消关机，则等几秒强制关机
