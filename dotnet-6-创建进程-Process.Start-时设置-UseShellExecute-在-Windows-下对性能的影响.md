
# dotnet 6 创建进程 Process.Start 时设置 UseShellExecute 在 Windows 下对性能的影响

本文将告诉大家，在 dotnet 6 或 dotnet 7 版本里，启动新的进程时，在 StartInfo 设置 UseShellExecute 为 true 和 false 时，对性能的影响

<!--more-->


<!-- CreateTime:2022/12/27 8:50:09 -->

<!-- 发布 -->

在 dotnet 6 或 dotnet 7 版本里，其他的版本我没有测试和去了解哈，启动新的进程时，在 StartInfo 设置 UseShellExecute 为 true 时，且当调用线程非 STA 时，在 Windows 下，性能会较差

为什么性能会比较差？下面将从 dotnet 源代码的角度来告诉大家

开始之前，回顾一下 UseShellExecute 属性的作用，在 Process.Start 里，是允许调用 Shell 打开进程的，传入的不一定要求是一个 exe 等可执行文件，还可以是某个文件，例如 txt 文件。传入文件时，系统将会根据默认打开程序，使用文件的默认打开程序打开文件，例如 txt 文件默认将使用记事本程序打开。想要实现此效果，就需要将 UseShellExecute 设置为 true 的值

设置为 true 的值，在 dotnet 底层将会调用 win32 的 [ShellExecuteExW](https://learn.microsoft.com/en-us/windows/win32/api/shellapi/nf-shellapi-shellexecuteexw) 函数

而对于打开某个 exe 来说，很多时候，除非是需要加上 verb 等，否则也是不需要用到 [ShellExecuteExW](https://learn.microsoft.com/en-us/windows/win32/api/shellapi/nf-shellapi-shellexecuteexw) 启动的。换句话说，如果明确知道是启动一个进程，只是启动时传传参数等，且没有其他的需求，可以放心设置 UseShellExecute 为 false 的值，当然，为 false 也是默认值

为什么将 UseShellExecute 设置为 true 的性能比较差？这还需要从 dotnet 的调用 [ShellExecuteExW](https://learn.microsoft.com/en-us/windows/win32/api/shellapi/nf-shellapi-shellexecuteexw) 函数方法开始聊起

在 dotnet 的 Process.Start 方法里面，有许多重载方法，最终都会调进去 `public bool Start()` 方法里面，在此方法里，将进入平台有关的 StartCore 方法

这里只讨论 Windows 下的 StartCore 方法的实现，其实现是根据 Windows 下的创建进程使用的 [CreateProcessW](https://learn.microsoft.com/zh-cn/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessw) 和 [ShellExecuteExW](https://learn.microsoft.com/en-us/windows/win32/api/shellapi/nf-shellapi-shellexecuteexw) 函数的不同从而需要判断 UseShellExecute 属性来决定调用哪个方法

```csharp
    public partial class Process : IDisposable
    {
        ... // 忽略其他代码

        private bool StartCore(ProcessStartInfo startInfo)
        {
            return startInfo.UseShellExecute
                ? StartWithShellExecuteEx(startInfo)
                : StartWithCreateProcess(startInfo);
        }

        ... // 忽略其他代码
    }
```

先来看看 StartWithCreateProcess 方法吧，这个方法比较简单，省略的代码如下

```csharp
    public partial class Process : IDisposable
    {
        ... // 忽略其他代码

        private unsafe bool StartWithCreateProcess(ProcessStartInfo startInfo)
        {
            if (startInfo.UserName.Length != 0)
            {
                  ... // 忽略其他代码

                                retVal = Interop.Advapi32.CreateProcessWithLogonW(
                                    startInfo.UserName,
                                    startInfo.Domain,
                                    (passwordPtr != IntPtr.Zero) ? passwordPtr : (IntPtr)passwordInClearTextPtr,
                                    logonFlags,
                                    null,            // we don't need this since all the info is in commandLine
                                    commandLinePtr,
                                    creationFlags,
                                    (IntPtr)environmentBlockPtr,
                                    workingDirectory,
                                    ref startupInfo,        // pointer to STARTUPINFO
                                    ref processInfo         // pointer to PROCESS_INFORMATION
                                );
            }
            else
            {
                  ... // 忽略其他代码
                        retVal = Interop.Kernel32.CreateProcess(
                                null,                // we don't need this since all the info is in commandLine
                                commandLinePtr,      // pointer to the command line string
                                ref unused_SecAttrs, // address to process security attributes, we don't need to inherit the handle
                                ref unused_SecAttrs, // address to thread security attributes.
                                true,                // handle inheritance flag
                                creationFlags,       // creation flags
                                (IntPtr)environmentBlockPtr, // pointer to new environment block
                                workingDirectory,    // pointer to current directory name
                                ref startupInfo,     // pointer to STARTUPINFO
                                ref processInfo      // pointer to PROCESS_INFORMATION
                            );
            }

            ... // 忽略其他代码
        }

        ... // 忽略其他代码
    }
```

在 dotnet 代码里面看到 StartWithCreateProcess 方法需要的代码很多，但其实只是调用 win32 方法比较繁琐而已

接下来看看 StartWithShellExecuteEx 方法的实现，通过这个方法的实现就可以知道为什么在 Windows 下，设置 UseShellExecute 为 true 且当调用线程非 STA 时，性能会较差的原因


```csharp
    public partial class Process : IDisposable
    {
        ... // 忽略其他代码

        private unsafe bool StartWithShellExecuteEx(ProcessStartInfo startInfo)
        {
                ... // 忽略其他代码
                ShellExecuteHelper executeHelper = new ShellExecuteHelper(&shellExecuteInfo);
                if (!executeHelper.ShellExecuteOnSTAThread())
                {
                    ... // 忽略其他代码
                }
                ... // 忽略其他代码
        }

        ... // 忽略其他代码
    }
```

可以看到在 StartWithShellExecuteEx 里使用的是 ShellExecuteHelper 辅助方法来实现，通过 ShellExecuteOnSTAThread 也能猜到，这是在 STA 线程执行的。这是因为启动线程如果是用来调用文件打开，一些 COM 是需要 STA 线程的。然而如果当前的线程不是 STA 的线程，那需要如何执行

接下来继续看 ShellExecuteOnSTAThread 的实现

```csharp
    internal unsafe class ShellExecuteHelper
    {
        ... // 忽略其他代码
            public bool ShellExecuteOnSTAThread()
            {
                // ShellExecute() requires STA in order to work correctly.

                if (Thread.CurrentThread.GetApartmentState() != ApartmentState.STA)
                {
                    ThreadStart threadStart = new ThreadStart(ShellExecuteFunction);
                    Thread executionThread = new Thread(threadStart)
                    {
                        IsBackground = true,
                        Name = ".NET Process STA"
                    };
                    executionThread.SetApartmentState(ApartmentState.STA);
                    executionThread.Start();
                    executionThread.Join();
                }
                else
                {
                    ShellExecuteFunction();
                }

                ... // 忽略其他代码

                return _succeeded;
            }

            private void ShellExecuteFunction()
            {
                try
                {
                    if (!(_succeeded = Interop.Shell32.ShellExecuteExW(_executeInfo)))
                        ErrorCode = Marshal.GetLastWin32Error();
                }
                catch (EntryPointNotFoundException)
                {
                    _notpresent = true;
                }
            }

        ... // 忽略其他代码
    }
```

可以看到在 dotnet 里面，判断当前线程，如果不是 STA 线程，那就再启动一个 STA 线程去执行代码，而且是等待启动的 STA 线程执行完成再方法，同步等待另一个线程。这就是为什么性能比较差的原因，性能差在需要启动线程和等待线程执行完成

那有伙伴说，那是不是每次都放在客户端的 STA 主线程调用好了，这样就让 dotnet 底层不需要启动新的线程？其实这不好，因为 ShellExecuteExW 这个 win32 方法不是非常快速的，在一些系统上，将会等待很长时间，特别是有 360 等的情况，如果在主线程被进入等待，那自然是不如多开一个后台线程

看完了原理之后，相信大家也就知道，如果明确知道是启动一个进程，只是启动时传传参数等，且没有其他的需求，可以放心设置 UseShellExecute 为 false 的值，当然，为 false 也是默认值，这样性能会更高




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。