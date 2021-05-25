
# dotnet 读 WPF 源代码笔记 插入触摸设备的初始化获取设备信息

在 WPF 触摸应用中，插入触摸设备，即可在应用里面使用上插入的触摸设备。在 WPF 使用触摸设备的触摸时，需要获取到触摸设备的信息，才能实现触摸

<!--more-->


<!-- CreateTime:2021/5/24 18:58:18 -->


<!-- 标签：WPF，WPF源代码 -->
<!-- 发布 -->

## 获取触摸设备插入

在 WPF 中，通过 Windows 消息获取触摸设备插入事件，在 `src\Microsoft.DotNet.Wpf\src\PresentationCore\System\Windows\Input\Stylus\Wisp\WispLogic.cs` 的 HandleMessage 将获取 Windows 消息，代码如下

```csharp
        internal override void HandleMessage(WindowMessage msg, IntPtr wParam, IntPtr lParam)
        {
            switch (msg)
            {
                // 忽略代码
                case WindowMessage.WM_TABLET_ADDED:
                    OnTabletAdded((uint)NativeMethods.IntPtrToInt32(wParam));
                    break;

                case WindowMessage.WM_TABLET_DELETED:
                    OnTabletRemovedImpl((uint)NativeMethods.IntPtrToInt32(wParam), isInternalCall: true);
                    break;
            }
        }
```

在 WPF 框架，使用 [WM_TABLET_ADDED](https://docs.microsoft.com/en-us/windows/win32/tablet/wm-tablet-added?WT.mc_id=WD-MVP-5003260 ) 和 [WM_TABLET_DELETED](https://docs.microsoft.com/en-us/windows/win32/tablet/wm-tablet-deleted?WT.mc_id=WD-MVP-5003260 ) 消息获取设备的插入和删除事件

如上面代码，在设备插入时，将会调用 OnTabletAdded 方法。如 [WM_TABLET_ADDED 官方文档](https://docs.microsoft.com/en-us/windows/win32/tablet/wm-tablet-added?WT.mc_id=WD-MVP-5003260 )描述，以上代码获取的参数是 Wisptis 的 Index 序号。这是因为用户可以插入多个触摸设备，通过传入序号可以拿到插入的设备

在 WPF 中，每次插入触摸设备，都会重新更新所有的触摸设备的信息，而不是只更新插入的设备。在 OnTabletAdded 方法里面，将会调用 GetDeviceCount 方法，在 GetDeviceCount 方法里面将通过 PenThread 的 WorkerGetTabletsInfo 更新所有触摸设备的信息，代码如下

```csharp
        private void OnTabletAdded(uint wisptisIndex)
        {
            lock (__penContextsLock)
            {
                WispTabletDeviceCollection tabletDeviceCollection = WispTabletDevices;
                // 忽略代码

                // Update the last known device count.
                _lastKnownDeviceCount = GetDeviceCount();

                uint tabletIndex = UInt32.MaxValue;
                // HandleTabletAdded returns true if we need to update contexts due to a change in tablet devices.
                if (tabletDeviceCollection.HandleTabletAdded(wisptisIndex, ref tabletIndex))
                {
                        // Update all contexts with this new tablet device.
                        foreach (PenContexts contexts in __penContextsMap.Values)
                        {
                            contexts.AddContext(tabletIndex);
                        }
                }
            }
        }

        private int GetDeviceCount()
        {
            PenThread penThread = null;

            // Get a PenThread by mimicking a subset of the code in TabletDeviceCollection.UpdateTablets().
            TabletDeviceCollection tabletDeviceCollection = TabletDevices;
            if (tabletDeviceCollection != null && tabletDeviceCollection.Count > 0)
            {
                penThread = tabletDeviceCollection[0].As<WispTabletDevice>().PenThread;
            }

            if (penThread != null)
            {
                // Use the PenThread to get the full, unfiltered tablets info to see how many there are.
                TabletDeviceInfo[] tabletdevices = penThread.WorkerGetTabletsInfo();
                return tabletdevices.Length;
            }
            else
            {
                // if there's no PenThread yet, return "unknown"
                return -1;
            }
} // WPF 代码格式化就是这样
```

以上代码调用 WorkerGetTabletsInfo 方法实际的获取触摸信息逻辑是放在触摸线程，上面代码需要先获取触摸线程 PenThread 然后调用触摸线程类的 WorkerGetTabletsInfo 方法，在这个方法里面执行逻辑

## 触摸线程

在 [WPF 触摸到事件](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%88%B0%E4%BA%8B%E4%BB%B6.html ) 博客里面告诉大家，在 WPF 框架，为了让触摸的性能足够强，将触摸的获取放在独立的进程里面

在获取触摸信息时，也需要调度到触摸线程执行。在 WPF 中，通过 PenThread 类的相关方法可以调度到触摸线程

在调用 WorkerGetTabletsInfo 方法时，进入 WorkerGetTabletsInfo 方法依然是主线程，里面代码如下

```csharp
        internal TabletDeviceInfo[] WorkerGetTabletsInfo()
        {
            // Set data up for this call
            WorkerOperationGetTabletsInfo getTablets = new WorkerOperationGetTabletsInfo();
            
            lock(_workerOperationLock)
            {
                _workerOperation.Add(getTablets);
            }

            // Kick thread to do this work.
            MS.Win32.Penimc.UnsafeNativeMethods.RaiseResetEvent(_pimcResetHandle.Value);

            // Wait for this work to be completed.
            getTablets.DoneEvent.WaitOne();
            getTablets.DoneEvent.Close();
        
            return getTablets.TabletDevicesInfo;
        }
```

实际上以上代码是放在 PenThreadWorker.cs 文件中，在 WPF 的触摸线程设计上，触摸线程是一个循环，将会等待 PenImc 层发送触摸消息，或者等待 `_pimcResetHandle` 锁被释放。如上面代码，先插入 WorkerOperationGetTabletsInfo 到 `_workerOperation` 列表中，然后调用 RaiseResetEvent 方法释放 `_pimcResetHandle` 对象。触摸线程将会因为 `_pimcResetHandle` 被释放而跳出循环，然后获取 `_workerOperation` 列表里面的项，进行执行逻辑

主线程将会在 `getTablets.DoneEvent.WaitOne` 方法里面进入锁，等待触摸线程执行 WorkerOperationGetTabletsInfo 完成之后释放这个锁，才能让主线程继续执行

触摸线程的循环逻辑代码大概如下

```csharp
        internal void ThreadProc()
        {
            Thread.CurrentThread.Name = "Stylus Input";
            while (!__disposed)
            {
                    // 忽略代码
                    WorkerOperation[] workerOps = null;

                    lock(_workerOperationLock)
                    {
                        if (_workerOperation.Count > 0)
                        {
                            workerOps = _workerOperation.ToArray();
                            _workerOperation.Clear();
                        }
                    }

                    if (workerOps != null)
                    {
                        for (int j=0; j<workerOps.Length; j++)
                        {
                            workerOps[j].DoWork();
                        }
                        workerOps = null;
                    }

                    // 这是第二层循环
                    while (true)
                    {
                            // 忽略代码

                    	    if (!MS.Win32.Penimc.UnsafeNativeMethods.GetPenEvent(
                                _handles[0], _pimcResetHandle.Value,
                                out evt, out stylusPointerId,
                                out cPackets, out cbPacket, out pPackets))
                            {
                                break;
                            }
                    }
            }
```

默认 WPF 的触摸线程都会在第二层循环，在 GetPenEvent 方法里面等待 PenImc 发送触摸消息或等待 `_pimcResetHandle` 释放。在跳出第二层循环，将会去获取 `_workerOperation` 的项，然后执行

```csharp
                    WorkerOperation[] workerOps = null;

                    lock(_workerOperationLock)
                    {
                        if (_workerOperation.Count > 0)
                        {
                            workerOps = _workerOperation.ToArray();
                            _workerOperation.Clear();
                        }
                    }

                    if (workerOps != null)
                    {
                        for (int j=0; j<workerOps.Length; j++)
                        {
                            workerOps[j].DoWork();
                        }
                        workerOps = null;
                    }
```

## 获取触摸信息

在调用 WorkerOperationGetTabletsInfo 的 DoWork 方法时，将会在触摸线程获取触摸设备信息

```csharp
        private class WorkerOperationGetTabletsInfo : WorkerOperation
        {
            internal TabletDeviceInfo[] TabletDevicesInfo
            {
                get { return _tabletDevicesInfo;}
            }

            /////////////////////////////////////////////////////////////////////////
            /// <summary>
            ///     Returns the list of TabletDeviceInfo structs that contain information
            ///     about all of the TabletDevices on the system.
            /// </summary>
            protected override void OnDoWork()
            {
                try
                {
                    // create new collection of tablets
                    MS.Win32.Penimc.IPimcManager3 pimcManager = MS.Win32.Penimc.UnsafeNativeMethods.PimcManager;
                    uint cTablets;
                    pimcManager.GetTabletCount(out cTablets);

                    TabletDeviceInfo[] tablets = new TabletDeviceInfo[cTablets];

                    for ( uint iTablet = 0; iTablet < cTablets; iTablet++ )
                    {
                        MS.Win32.Penimc.IPimcTablet3 pimcTablet;
                        pimcManager.GetTablet(iTablet, out pimcTablet);

                        tablets[iTablet] = PenThreadWorker.GetTabletInfoHelper(pimcTablet);
                    }

                    // Set result data and signal we are done.
                    _tabletDevicesInfo = tablets;
                }
                catch (Exception e) when (PenThreadWorker.IsKnownException(e))
                {
                    Debug.WriteLine("WorkerOperationGetTabletsInfo.OnDoWork failed due to: {0}{1}", Environment.NewLine, e.ToString());
                }
            }

            TabletDeviceInfo[] _tabletDevicesInfo = Array.Empty<TabletDeviceInfo>();
        }
```

上面代码的 IPimcManager3 接口是一个 COM 接口，实际逻辑是在 PenImc 层进行定义，在 PenImcRcw.cs 引用，代码如下

```csharp
    [
    ComImport,
    Guid(PimcConstants.IPimcManager3IID),
    InterfaceType(ComInterfaceType.InterfaceIsIUnknown)
    ]
    interface IPimcManager3
    {
        void GetTabletCount(out UInt32 count);
        void GetTablet(UInt32 tablet, out IPimcTablet3 IPimcTablet);
    }
```

在 PenImc 层的 PenImc.idl 文件里面，定义了公开的接口

```csharp
[
	object,
	uuid(BD2C38C2-E064-41D0-A999-940F526219C2),
	nonextensible,
	helpstring("IPimcManager3 Interface"),
	pointer_default(unique)
]
interface IPimcManager3 : IUnknown {
    [helpstring("method GetTabletCount")] HRESULT GetTabletCount([out] ULONG* pcTablets);
    [helpstring("method GetTablet")     ] HRESULT GetTablet([in] ULONG iTablet, [out] IPimcTablet3** ppTablet);
};
```

在 WPF 中，在 C# 代码使用的不是最底层的方法，也就是 `BD2C38C2-E064-41D0-A999-940F526219C2` 组件只是 WPF 用的，而不是系统等给的接口

实际调用底层的代码是在 PenImc 层的 C++ 代码，但 PenImc 层的 C++ 代码只是一层转发调用而已，换句话说，如果使用 C# 调用底层的系统的组件也是完全可以的

如上面代码通过 GetTabletCount 方法获取当前的触摸设备，此方法是通过 COM 调用到在 PenImc.idl 文件定义的 GetTabletCount 获取的，实际定义的代码是 PimcManager.cpp 文件的 GetTabletCount 方法

```C++
STDMETHODIMP CPimcManager::GetTabletCount(__out ULONG* pcTablets)
{
    DHR;

    ULONG cTablets = 0;

    LoadWisptis(); // Try to load wisptis via the surrogate object.
    
    // we will return 0 in the case that there is no stylus since mouse is not considered a stylus anymore
    if (m_fLoadedWisptis)
    {
        CHR(m_pMgrS->GetTabletCount(&cTablets));
    }
    
    *pcTablets = cTablets;
    
CLEANUP:
    RHR;
}
```

以上代码里面用到了一些宏，如 `DHR` 的含义是定义 HRESULT 变量，代码如下

```C++
#define DHR                                         \
    HRESULT hr = S_OK;
```

而 `CHR` 表示的是判断 HRESULT 的值，如果失败了，将会调用 `CLEANUP` 标签的内容。在 `CHR` 里面用到 goto 的方法

```C++
#define CHR(hr_op)                                  \
    {                                               \
        hr = hr_op;                                 \
        if (FAILED(hr))                             \
            goto CLEANUP;                           \
    }
```

上面代码的 `RHR` 表示的是返回 HRESULT 变量

```C++
#define RHR                                         \
    return hr;
```

因此以上代码实际就是如下代码

```C++
STDMETHODIMP CPimcManager::GetTabletCount(__out ULONG* pcTablets)
{
    HRESULT hr = S_OK;

    ULONG cTablets = 0;

    LoadWisptis(); // Try to load wisptis via the surrogate object.
    
    // we will return 0 in the case that there is no stylus since mouse is not considered a stylus anymore
    if (m_fLoadedWisptis)
    {
        hr = m_pMgrS->GetTabletCount(&cTablets);
        if (FAILED(hr))
        {
            goto CLEANUP;
        }
    }
    
    *pcTablets = cTablets;
    
CLEANUP:
    return hr;
}
```

通过上面代码可以看到，实际调用的是 `m_pMgrS` 的 GetTabletCount 方法，也就是如下代码定义的方法

```C++
    MIDL_INTERFACE("764DE8AA-1867-47C1-8F6A-122445ABD89A")
    ITabletManager : public IUnknown
    {
    public:
        virtual /* [helpstring] */ HRESULT STDMETHODCALLTYPE GetDefaultTablet( 
            /* [out] */ __RPC__deref_out_opt ITablet **ppTablet) = 0;
        
        virtual /* [helpstring] */ HRESULT STDMETHODCALLTYPE GetTabletCount( 
            /* [out] */ __RPC__out ULONG *pcTablets) = 0;
        
        virtual /* [helpstring] */ HRESULT STDMETHODCALLTYPE GetTablet( 
            /* [in] */ ULONG iTablet,
            /* [out] */ __RPC__deref_out_opt ITablet **ppTablet) = 0;
        
        virtual /* [helpstring] */ HRESULT STDMETHODCALLTYPE GetTabletContextById( 
            /* [in] */ TABLET_CONTEXT_ID tcid,
            /* [out] */ __RPC__deref_out_opt ITabletContext **ppContext) = 0;
        
        virtual /* [helpstring] */ HRESULT STDMETHODCALLTYPE GetCursorById( 
            /* [in] */ CURSOR_ID cid,
            /* [out] */ __RPC__deref_out_opt ITabletCursor **ppCursor) = 0;
        
    };
```

可以看到这是一个 COM 接口调用，实际使用的就是系统提供的 [ITabletManager](https://docs.microsoft.com/en-us/windows/win32/tablet/itabletmanager?WT.mc_id=WD-MVP-5003260 ) 组件

在底层系统组件，先调用 [ITabletManager 的 GetTabletCount 方法](https://docs.microsoft.com/en-us/windows/win32/tablet/itabletmanager-gettabletcount?WT.mc_id=WD-MVP-5003260 ) 获取触摸设备数量，然后遍历触摸设备序号拿到 ITablet 对象

在 C# 代码里面的逻辑如下

```csharp
                    pimcManager.GetTabletCount(out cTablets);

                    TabletDeviceInfo[] tablets = new TabletDeviceInfo[cTablets];

                    for ( uint iTablet = 0; iTablet < cTablets; iTablet++ )
                    {
                        MS.Win32.Penimc.IPimcTablet3 pimcTablet;
                        pimcManager.GetTablet(iTablet, out pimcTablet);

                        tablets[iTablet] = PenThreadWorker.GetTabletInfoHelper(pimcTablet);
                    }
```

这里的 pimcManager.GetTablet 方法将会调用到 PimcManager.cpp 的 GetTablet 方法

```C++
STDMETHODIMP CPimcManager::GetTablet(ULONG iTablet, __deref_out IPimcTablet3** ppTablet)
{
    DHR;

    switch (iTablet)
    {
        case RELEASE_MANAGER_EXT:
        {
            CHR(m_managerLock.Unlock());
        }
        break;
        default:
        {
            CHR(GetTabletImpl(iTablet, ppTablet));
        }
    }

CLEANUP:
    RHR;
}

STDMETHODIMP CPimcManager::GetTabletImpl(ULONG iTablet, __deref_out IPimcTablet3** ppTablet)
{
    DHR;
    LoadWisptis(); // Make sure wisptis has been loaded! (Can happen when handling OnTabletAdded message)
    
    CComPtr<ITablet>            pTabS;
    CComObject<CPimcTablet> *   pTabC;

    // Can only call if we have real tablet hardware which means wisptis must be loaded!
    CHR(m_fLoadedWisptis ? S_OK : E_UNEXPECTED);
    CHR(CComObject<CPimcTablet>::CreateInstance(&pTabC));
    CHR(pTabC->QueryInterface(IID_IPimcTablet3, (void**)ppTablet));
    CHR(m_pMgrS->GetTablet(iTablet, &pTabS));
    CHR(pTabC->Init(m_fLoadedWisptis?pTabS:NULL, this));

CLEANUP:
    RHR;
}
```

本质调用的是 `m_pMgrS` 的 GetTablet 方法，也就是系统提供的 [ITabletManager 的 GetTablet 方法](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/legacy/aa373683(v=vs.85) ) 获取 ITablet 接口。只是在 C++ 代码里面，将 ITablet 接口再做一层封装，返回给 C# 的是 IPimcTablet3 接口

接下来就是通过 PenThreadWorker 的 GetTabletInfoHelper 方法获取触摸信息

```csharp
        private static TabletDeviceInfo GetTabletInfoHelper(IPimcTablet3 pimcTablet)
        {
            TabletDeviceInfo tabletInfo = new TabletDeviceInfo();

            tabletInfo.PimcTablet = new SecurityCriticalDataClass<IPimcTablet3>(pimcTablet);
            pimcTablet.GetKey(out tabletInfo.Id);
            pimcTablet.GetName(out tabletInfo.Name);
            pimcTablet.GetPlugAndPlayId(out tabletInfo.PlugAndPlayId);
            int iTabletWidth, iTabletHeight, iDisplayWidth, iDisplayHeight;
            pimcTablet.GetTabletAndDisplaySize(out iTabletWidth, out iTabletHeight, out iDisplayWidth, out iDisplayHeight);
            tabletInfo.SizeInfo = new TabletDeviceSizeInfo(new Size(iTabletWidth, iTabletHeight),
                                                           new Size(iDisplayWidth, iDisplayHeight));
            int caps;
            pimcTablet.GetHardwareCaps(out caps);
            tabletInfo.HardwareCapabilities = (TabletHardwareCapabilities)caps;
            int deviceType;
            pimcTablet.GetDeviceType(out deviceType);
            tabletInfo.DeviceType = (TabletDeviceType)(deviceType -1);

            // 
            // REENTRANCY NOTE: Let a PenThread do this work to avoid reentrancy!
            //                  The IPimcTablet3 object is created in the pen thread. If we access it from the UI thread,
            //                  COM will set up message pumping which will cause reentrancy here.
            InitializeSupportedStylusPointProperties(pimcTablet, tabletInfo);
            tabletInfo.StylusDevicesInfo = GetStylusDevicesInfo(pimcTablet);

            
            // Obtain the WispTabletKey for future use in locking the WISP tablet.
            tabletInfo.WispTabletKey = MS.Win32.Penimc.UnsafeNativeMethods.QueryWispTabletKey(pimcTablet);

            
            // If the manager has not already been created and locked, we will lock it here.  This is the first opportunity
            // we will have to lock the manager as it will have been created on the thread to instantiate the first tablet.
            MS.Win32.Penimc.UnsafeNativeMethods.SetWispManagerKey(pimcTablet);

            MS.Win32.Penimc.UnsafeNativeMethods.LockWispManager();

            return tabletInfo;
        }
```

实际调用的就是 [ITablet 接口](https://docs.microsoft.com/en-us/windows/win32/tablet/itablet?WT.mc_id=WD-MVP-5003260 ) 的方法

以上代码的 `pimcTablet.GetKey` 方法是在 C++ 层封装的，而不是系统提供的

```C++
STDMETHODIMP CPimcTablet::GetKey(__out INT * pKey)
{
    DHR;
    CHR(pKey ? S_OK : E_INVALIDARG);
    *pKey = (INT)PtrToInt(m_pTabS.p);
CLEANUP:
    RHR;
}

    CComPtr<ITablet>      m_pTabS;
```

在 WPF 框架，获取的方法本质就是通过 Tablet PC 系统组件获取

更多触摸请看 [WPF 触摸相关](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E7%9B%B8%E5%85%B3.html )






<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。