# 为什么 WPF 软件在 win7 启动时会尝试调起 wisptis 进程

我看到一个问题是在 win7 系统上，如果开机启动的软件是 WPF 软件，而这个 WPF 软件在系统的 wisptis 进程启动之前就启动了，那么 WPF 将会调起 wisptis 进程。而在 wisptis 进程已经启动完成，此时启动 WPF 进程不会再打开新的 wisptis 进程。但是被 WPF 启动的 wisptis 进程存在这样的问题，在触摸屏上 win7 的双指打开右键菜单等功能不可用

<!--more-->
<!-- CreateTime:2020/1/20 16:28:32 -->



在 WPF 启动时，将会在 Window 类的 Visibility 修改时调用到 WispLogic.RegisterHwndForInput 方法进行初始化触摸，这部分详细请看 [WPF 触摸到事件](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%88%B0%E4%BA%8B%E4%BB%B6.html) 而在初始化触摸时，需要用到 PenIMC 的逻辑

在 win7 系统上，触摸需要通过 wisptis 进程的辅助才能让 WPF 进程能够完成实时触摸，这里的 wisptis 是 Windows Ink Services Platform Tablet Input Subsystem 进程，用于处理触摸书写等功能。也是 [RealTimeStylus](https://docs.microsoft.com/en-us/windows/win32/tablet/realtimestylus-reference ) 的提供，通过一些不靠谱的文档和经验，其实 PenIMC 的核心逻辑就是 [RealTimeStylus](https://docs.microsoft.com/en-us/windows/win32/tablet/realtimestylus-reference ) 提供的。上面这句话对或不对我不敢说，只能说用 [RealTimeStylus](https://docs.microsoft.com/en-us/windows/win32/tablet/realtimestylus-reference ) 可以实现 PenIMC 的效果，而且 API 和参数差不多

那么 PenIMC 又是什么呢？其实 PenIMC 是 `penimc2_v0400.dll` 文件，在不同的版本的 .NET Framework 和系统上这个文件是不同的，包括文件名也不同，看这个文件命名就知道。没错，你可以在 `penimc2_v0400.dll` 文件所在的文件夹找到一堆 penimc 文件。这个文件就是提供给 WPF 的触摸核心 `PenThreadWorker` 的 COM 组件（其实没有文档说这货是纯 COM 组件） 也就是和触摸相关的

也就是在 WPF 窗口打开显示将会初始化触摸，初始化触摸需要依赖 PenIMC 组件，这个库本质是对 [RealTimeStylus](https://docs.microsoft.com/en-us/windows/win32/tablet/realtimestylus-reference ) 的封装，依赖于 win7 的 Windows Ink Services Platform Tablet Input Subsystem （wisptis） 服务。这也能说明为什么会尝试调起 wisptis 进程

具体调起的代码放在 WPF 的 `src\Microsoft.DotNet.Wpf\src\PenImc\dll\PimcManager.cpp` 文件里面，实现的代码大概如下

```c++
void CPimcManager::LoadWisptis()
{
    // 删除一些代码
        // **********
        // NOTE:    PenIMC has duplicated the code for loading wisptis from InkObj.
        //          Whenever WIC team makes any changes, we should coordinate with them to work on fixes.
        // **********
        if (IsVistaOrGreater())
        {
            // DDVSO 144719. There are some scenarios were we must skip loading wisptis since 
            // they are not supported and can cause delays or crashes.
            if (ShouldLoadWisptis())
            {
                // we do this to signal TabSvc that it needs to spin up wisptis
                //  so that it is at the right IL.
                HANDLE hEventRequest = OpenEvent(EVENT_MODIFY_STATE, FALSE, PENPROCESS_WISPTIS_REQUEST_EVENT);
                HANDLE hEventRunning = OpenEvent(SYNCHRONIZE, FALSE, PENPROCESS_WISPTIS_RUNNING_EVENT);

                //if we don't have the event (TabSvc isn't running), or we timed out,
                // that means Wisptis isn't running, so we'll start it; we do this via
                // ShellExecute so that it gets started at high-IL (as indicated by
                // Wisptis's manifest) to avoid IL-mismatch issues
                //we allow wisptis to be started without TabSvc for backcompat
            
                if(hEventRunning == NULL)
                {
                   // create the event since TabSvc isn't running
                   hEventRunning = CreateEvent(NULL, TRUE, FALSE, PENPROCESS_WISPTIS_RUNNING_EVENT);
                }

                if(hEventRequest != NULL && hEventRunning != NULL)
                {
                    //when this wait returns, wisptis will have registered its classes with COM
                    //if this fails or times out, we'll risk starting wisptis at a mismatched IL
                    DWORD dwResult = SignalObjectAndWait(hEventRequest, hEventRunning, 30000 /* thirty seconds */, FALSE);

                    hr = dwResult == WAIT_OBJECT_0 ? S_OK : E_FAIL;
                }

                // DDVSO:398137
                // Since hEventRequest is no longer of use at this point, close the handle.
                SafeCloseHandle(&hEventRequest);

                if(/* wait timed out */ FAILED(hr) ||
                   /* couldn't open the event for some reason */ hEventRunning == NULL ||
                   /* wisptis isn't already running */ WaitForSingleObject(hEventRunning, 0) == WAIT_TIMEOUT)
                {
                    PVOID pvOldValue = NULL;
                    BOOL bIsWow64 = FALSE;
                    LPFNWOW64DISABLEWOW64FSREDIRECTION fnWow64DisableWow64FsRedirection = NULL;
                    LPFNWOW64REVERTWOW64FSREDIRECTION fnWow64RevertWow64FsRedirection = NULL;
                    HMODULE hKernel32 = NULL;

                    // Check whether this is running under Wow64 and, if so, disable file system redirection
                    // on the current thread - otherwise it will look for wisptis in the syswow64 directory
                    // instead of system32.
                    TPDBG_VERIFY(IsWow64Process(GetCurrentProcess(),&bIsWow64));
                    if (bIsWow64)
                    {
                        // NOTICE-2006/06/13-WAYNEZEN,
                        // Since penimc may also run on the top of XPSP2, We cannot call Wow64DisableWow64FsRedirection/Wow64RevertWow64FsRedirection
                        // directly. Otherwise it will cause Entry Point Not Found error even though we don't really on those functions on 32-bit XP.
                        // So we have to use GetProcAddress to resovle the function address dynamically.
                        hKernel32 = GetModuleHandle(KERNEL32_NAME);
                        fnWow64DisableWow64FsRedirection = (LPFNWOW64DISABLEWOW64FSREDIRECTION)GetProcAddress(
                                                                hKernel32, WOW64DISABLEWOW64FSREDIRECTION_NAME);
                        fnWow64RevertWow64FsRedirection = (LPFNWOW64REVERTWOW64FSREDIRECTION)GetProcAddress(
                                                                hKernel32, WOW64REVERTWOW64FSREDIRECTION_NAME);

                        TPDBG_VERIFY(fnWow64DisableWow64FsRedirection(&pvOldValue));
                    }

                    SHELLEXECUTEINFO sei = {0};

                    sei.cbSize = sizeof(sei);
                    sei.lpFile = WISPTIS_DIR WISPTIS_NAME;
                    sei.lpParameters = WISPTIS_MANUAL_LAUNCH;
                    sei.lpVerb = NULL;
                    sei.fMask = SEE_MASK_FLAG_DDEWAIT | SEE_MASK_DOENVSUBST | SEE_MASK_FLAG_NO_UI;
                    sei.lpDirectory = WISPTIS_DIR;
                    sei.hInstApp = (HINSTANCE)0;

                    // 就是在这里启动进程的
                    BOOL bResult = ShellExecuteEx(&sei);

                    // Restore the file system redirection settings.
                    if (bIsWow64)
                    { 
                        TPDBG_VERIFY(fnWow64RevertWow64FsRedirection(pvOldValue));
                    }

                    hr = bResult ? S_OK : E_FAIL;
                    if(FAILED(hr))
                    {
                       OutputDebugString(L"PimcManager::LoadWisptis failed to ShellExecuteEx.\r\n");
                    }
                }
                   
                if(SUCCEEDED(hr) && hEventRunning != NULL)
                {
                    (void)WaitForSingleObject(hEventRunning, PENPROCESS_WISPTIS_LOADING_TIMEOUT /* 30 seconds */);
                    //regardless of the return from this, we'll still try to spin wisptis up via COM
                }
                
                SafeCloseHandle(&hEventRunning);

                if(SUCCEEDED(hr))
                {
                   CHR(m_pMgrS.CoCreateInstance(CLSID_TabletManagerS)); //, NULL, CLSCTX_INPROC_SERVER | CLSCTX_LOCAL_SERVER));

                   // Ensure the WISP tablet manager is added to the GIT.
                   m_wispManagerLock = GitComLockableWrapper<ITabletManager>(m_pMgrS, ComApartmentVerifier::Mta());
                   CHR(m_wispManagerLock.CheckCookie());

                   m_fLoadedWisptis = TRUE;
                }
            }
        }
}
```

更多关于 PenImc 的原理请参阅 [WPF 触摸底层 PenImc 是如何工作的](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%BA%95%E5%B1%82-PenImc-%E6%98%AF%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%E7%9A%84.html )

而为什么 WPF 启动的 wisptis 进程有很多坑？大部分只是启动进程权限问题，比如进程被删除等，更详细我也不知道

规避方法是什么？其实不让触摸执行也就是可以了，但是我如何让 WPF 还能交互？没关系，假装自己是一个古老的应用，只支持鼠标消息就可以啦。但是我想要做多指触摸怎么办？先不要触摸，等待 wisptis 进程启动之后，通过 [WPF 模拟触摸设备](https://blog.lindexi.com/post/WPF-%E6%A8%A1%E6%8B%9F%E8%A7%A6%E6%91%B8%E8%AE%BE%E5%A4%87.html) 方案重新注册一遍触摸

我一开始启动太快了，没关系，我一开始启动的是一个 win32 的启动图，等待后台逻辑判断 wisptis 启动之后，我才打开 WPF 的窗口。根据上面的说法，其实窗口没有修改 Visiliblity 之前是不会初始化触摸的，也就是不会启动 wisptis 进程的

现在 win7 已经不受微软支持了，是时候升级 win10 啦。因为 Win10 下不再是一个进程了，详细请看 [Win10 的 WPF 程序的 wisptis 服务是附加到进程的窗口](https://blog.lindexi.com/post/Win10-%E7%9A%84-WPF-%E7%A8%8B%E5%BA%8F%E7%9A%84-wisptis-%E6%9C%8D%E5%8A%A1%E6%98%AF%E9%99%84%E5%8A%A0%E5%88%B0%E8%BF%9B%E7%A8%8B%E7%9A%84%E7%AA%97%E5%8F%A3.html)

更多触摸请看 [WPF 触摸相关](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E7%9B%B8%E5%85%B3.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
