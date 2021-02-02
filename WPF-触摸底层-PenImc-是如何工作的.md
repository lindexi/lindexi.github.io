
# WPF 触摸底层 PenImc 是如何工作的

在 WPF 里面有其他软件完全比不上的超快速的触摸，这个触摸是通过 PenImc 获取的。现在 WPF 开源了，本文就带大家来阅读触摸底层的代码，阅读本文需要一点 C# 和 C++ 基础

<!--more-->


<!-- CreateTime:4/19/2020 5:23:45 PM -->



现在 WPF 开源，所有源代码都可以在官方代码找到，本文只是让大家能够更快的了解整个触摸的代码和更快的了解代码，和知道对应的功能在哪个代码

在WPF的触摸的 PenThreadWorker 调用 ThreadProc 的方法，就通过 MS.Win32.Penimc.UnsafeNativeMethods.GetPenEvent 方法获取触摸。本文仅讨论在 PenThreadWorker 下层的内容，在此上层的内容，请看[WPF 触摸到事件](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%88%B0%E4%BA%8B%E4%BB%B6.html)

那么在 PenImc 里面做了什么？

在 PenImc 原理里面，其实就是通过共享内存和 COM 的方式通过 RealTimeStylus 的方式快速获取触摸消息

先通过 WISPTIS_SM_SECTION_NAME 和 WISPTIS_SM_MUTEX_NAME 分别拿到共享内存和进程锁这样可以通过锁通知共享内存收到消息，然后通过读取内存的信息返回到上层

整个初始化的代码放在 PimcContext.cpp 里

在 `HRESULT CPimcContext::InitNamedCommunications(__in CComPtr<ITabletContextP> pCtxP)` 的方法里面，初始 szSectionName 字符串作为命名管道连接方法

```C++
    TCHAR szSectionName[MAX_PATH + 1];

    StringCchPrintf(
        szSectionName,
        LENGTHOFARRAY(szSectionName),
        WISPTIS_SM_SECTION_NAME,
        dwPid,
        dwFileMappingId);
```

而 `WISPTIS_SM_SECTION_NAME` 的定义如下

```C++
#define WISPTIS_SM_MORE_DATA_EVENT_NAME     _T("wisptis-1-%d-%u")
#define WISPTIS_SM_MUTEX_NAME               _T("wisptis-2-%d-%u")
#define WISPTIS_SM_SECTION_NAME             _T("wisptis-3-%d-%u")
#define WISPTIS_SM_THREAD_EVENT_NAME        _T("wisptis-4-%u")
```

此时通过打开内存的方式

```C++
    m_hFileMappingSharedMemory = OpenFileMapping(FILE_MAP_READ | FILE_MAP_WRITE, FALSE, szSectionName);
```

可以获取内存信息

```C++
    m_pSharedMemoryHeader = (SHAREDMEMORY_HEADER*)MapViewOfFile(
        m_hFileMappingSharedMemory,     // handle
        FILE_MAP_READ | FILE_MAP_WRITE, // desired access
        0,       // offset in file, High
        0,       // offset in file, Low
        sizeof(SHAREDMEMORY_HEADER));   // number of bytes to map

    m_pbSharedMemoryRawData = (BYTE*)MapViewOfFile(
        m_hFileMappingSharedMemory,     // handle
        FILE_MAP_READ,                  // desired access
        0,       // offset in file, High
        0,       // offset in file, Low
        m_pSharedMemoryHeader->cbTotal);// number of bytes to map
```

关于打开的代码请看

[ITabletContextP::UseNamedSharedMemoryCommunications method - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/tablet/itabletcontextp-usenamedsharedmemorycommunications )

此时就可以通过 `m_pbSharedMemoryRawData` 获取内存信息

这就是初始化的代码

在 WPF 调用 GetPenEvent 方法，将会进入 PimcContext.cpp 的 GetPenEvent 方法

在这个方法里面先通过 MsgWaitForMultipleObjectsEx 等待 Wisp 服务的收集，在收集完成之后会释放锁，进入 GetPenEventCore 方法

在 GetPenEventCore 使用很长的判断逻辑，其中主要是判断当前是获取数据才会进入到 WPF 的收集到触摸点

```C++
 switch (dwWait)
 {
    case WAIT_TIMEOUT:
        m_fSingleFireTimeout = FALSE; // (only fire the timeout once before more data shows up)
        *pEvt      = 1; // timeout event
        *pCursorId = 0;
        *pcPackets = 0;
        *pcbPacket = 0;
        *pPackets  = NULL;
        break;

    case WAIT_OBJECT_0 + 0: // update
       // 忽略代码

    case WAIT_OBJECT_0 + 1: // more data
    // 这里就是等待共享内存
    DWORD dwWaitAccess = WaitForSingleObject(m_hMutexSharedMemory, INFINITE);
  }
```

通过上面代码可以看到在 `m_hMutexSharedMemory` 的信息，可以在 `m_pSharedMemoryHeader` 读取

```C++
switch (m_pSharedMemoryHeader->dwEvent)
{
    case WM_TABLET_PACKET:
    case WM_TABLET_CURSORDOWN:
    case WM_TABLET_CURSORUP:
        *pEvt      = m_pSharedMemoryHeader->dwEvent;
        *pCursorId = m_pSharedMemoryHeader->cid;
        *pcPackets = m_pSharedMemoryHeader->cPackets;
        *pcbPacket = m_pSharedMemoryHeader->cbPackets / m_pSharedMemoryHeader->cPackets;
        CHR(EnsurePackets(m_pSharedMemoryHeader->cbPackets));
        CopyMemory(m_pbPackets, m_pbSharedMemoryPackets, m_pSharedMemoryHeader->cbPackets);
        *pPackets  = (INT_PTR)m_pbPackets;

#ifdef DELIVERY_PROFILING
        for (INT iPacket = 0; iPacket < *pcPackets; iPacket++)
        {
             INT iOffset = iPacket * (*pcbPacket) / sizeof(LONG);
             switch (m_pSharedMemoryHeader->dwEvent)
             {
                 case WM_TABLET_PACKET:     ProfilePackets(/*fDown*/FALSE, /*fUp*/FALSE, ((LONG*)m_pbSharedMemoryPackets)[iOffset + 0], ((LONG*)m_pbSharedMemoryPackets)[iOffset + 1]); break;
                 case WM_TABLET_CURSORDOWN: ProfilePackets(/*fDown*/TRUE,  /*fUp*/FALSE, ((LONG*)m_pbSharedMemoryPackets)[iOffset + 0], ((LONG*)m_pbSharedMemoryPackets)[iOffset + 1]); break;
                 case WM_TABLET_CURSORUP:   ProfilePackets(/*fDown*/FALSE, /*fUp*/TRUE,  ((LONG*)m_pbSharedMemoryPackets)[iOffset + 0], ((LONG*)m_pbSharedMemoryPackets)[iOffset + 1]); break;
             }
        }
#endif
        break;

    case WM_TABLET_CURSORINRANGE:
    case WM_TABLET_CURSOROUTOFRANGE:
        *pEvt      = m_pSharedMemoryHeader->dwEvent;
        *pCursorId = m_pSharedMemoryHeader->cid;
        *pcPackets = 0;
        *pcbPacket = 0;
        *pPackets  = NULL;
        break;

    case WM_TABLET_SYSTEMEVENT:
        *pEvt      = m_pSharedMemoryHeader->dwEvent;
        *pCursorId = m_pSharedMemoryHeader->cid;
        *pcPackets = 0;
        *pcbPacket = 0;
        *pPackets  = NULL;
        m_sysEvt     = m_pSharedMemoryHeader->sysEvt;
        m_sysEvtData = m_pSharedMemoryHeader->sysEvtData;
        break;

    default:
        *pEvt      = 0;
        *pCursorId = 0;
        *pcPackets = 0;
        *pcbPacket = 0;
        *pPackets  = NULL;
        break;
}
```


定义的代码放在 pentypes.h 文件

```C++
#define WM_TABLET_DEFBASE    0x02C0

#define WM_TABLET_CONTEXTCREATE  (WM_TABLET_DEFBASE + 0)
#define WM_TABLET_CONTEXTDESTROY (WM_TABLET_DEFBASE + 1)
#define WM_TABLET_CURSORNEW      (WM_TABLET_DEFBASE + 2)
#define WM_TABLET_CURSORINRANGE  (WM_TABLET_DEFBASE + 3)
#define WM_TABLET_CURSOROUTOFRANGE           (WM_TABLET_DEFBASE + 4)
#define WM_TABLET_CURSORDOWN     (WM_TABLET_DEFBASE + 5)
#define WM_TABLET_CURSORUP       (WM_TABLET_DEFBASE + 6)
#define WM_TABLET_PACKET         (WM_TABLET_DEFBASE + 7)
#define WM_TABLET_ADDED          (WM_TABLET_DEFBASE + 8)
#define WM_TABLET_DELETED        (WM_TABLET_DEFBASE + 9)
#define WM_TABLET_SYSTEMEVENT    (WM_TABLET_DEFBASE + 10)
#define WM_TABLET_MAX            (WM_TABLET_DEFBASE + WM_TABLET_MAXOFFSET)
```

这里的 WM_TABLET_CURSORINRANGE 是 (WM_TABLET_DEFBASE + 3) 也就是 707 对应在 WPF 定义的 PenEventPenInRange 的值

```csharp
 const int PenEventPenInRange    = 707;
 const int PenEventPenOutOfRange = 708;
 const int PenEventPenDown       = 709;
 const int PenEventPenUp         = 710;
 const int PenEventPackets       = 711;
 const int PenEventSystem        = 714;
```

也就是上面的代码就是整个触摸的核心代码

更多代码请看 [https://github.com/dotnet/wpf/](https://github.com/dotnet/wpf/) 

[IRealTimeStylus::GetPacketDescriptionData (rtscom.h) - Win32 apps](https://docs.microsoft.com/zh-cn/windows/win32/api/rtscom/nf-rtscom-irealtimestylus-getpacketdescriptiondata?redirectedfrom=MSDN )

更多触摸请看 [WPF 触摸相关](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E7%9B%B8%E5%85%B3.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。