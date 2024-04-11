
# 记 dotnet 8.0.4 修复的 WPF 的触摸模块安全问题

本文记录 dotnet 8.0.4 版本修复的 WPF 的触摸模块安全问题，此问题影响所有的 .NET 版本，修复方法是更新 SDK 和运行时

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

宣布安全漏洞地址：<https://github.com/dotnet/wpf/issues/9003>

漏洞代号：[CVE-2024-21409](https://www.cve.org/CVERecord?id=CVE-2024-21409)

核心更改： <https://github.com/dotnet/wpf/commit/c15b5c68cd74ae28bc99af539d05880658c45024>

修复的原因和修复的方法请参阅核心请参阅核心更改里面的注释，注释内容如下

```csharp
    // The CComObject<CPimcManager> destructor is the only function which calls into this
    // FinalRelease code.
    //
    // In all successful usage of CPimcManager: 1) Managed WPF code uses CoCreateInstance
    // to acquire an IPimcManager2 interface to a brand-new CPimcManager instance (created by
    // the ATL CComCreator<T>::CreateInstance machinery), meaning FinalConstruct by-definition
    // completes successfully, meaning "m_managerLock" is therefore guaranteed to be locked;
    // 2) Managed WPF code then runs through its full end-to-end usage of the CPimcManager
    // object (generally managed by the code in PenThreadWorker.cs); 3) When/if the managed WPF
    // code determines that the CPimcManager object is no longer needed, it sends a
    // RELEASE_MANAGER_EXT message (see UnsafeNativeMethods.ReleaseManagerExternalLock()) which
    // unlocks "m_managerLock"; 4) Now that it is unlocked, the CComObject<CPimcManager> object
    // can be destroyed when/if its refcount drops to zero, and this FinalRelease function will
    // run at that time.
    //
    // So in all successful usage cases, it is guaranteed that "m_managerLock" is already
    // unlocked when this code runs (because if it was still locked, the lock itself would have
    // prevented the refcount from reaching zero, and would have prevented this function from
    // ever running).
    //
    // That said, in unsuccessful usage cases, the ATL CComCreator<T>::CreateInstance machinery
    // can fail, meaning it will destroy the brand-new CPimcManager instance before returning
    // an error back to the CreateInstance caller.  Destroying the brand-new instance triggers
    // the CComObject<CPimcManager> destructor and therefore calls into this function during
    // the CComCreator<T>::CreateInstance operation itself.
    //
    // The final step in CComCreator<T>::CreateInstance is a QI which queries the newly-created
    // object for whatever interface has been requested by the caller.  This operation is the
    // main way that CComCreator<T>::CreateInstance can fail.  For example, this QI is
    // guaranteed to fail whenever the CoCreateInstance caller targets the CPimcManager CLSID
    // but passes in a "random" IID that has nothing to do with IPimcManager2 or anything else
    // that CPimcManager implements.
    //
    // (In CPimcManager construction, outside of pathological cases (e.g., where a small heap
    // allocation in OS code fails due to out-of-memory), there are no other known ways that
    // the CComCreator<T>::CreateInstance sequence can fail; so the QI failure is the only
    // failure mode that is known to be of general interest.)
    //
    // The QI failure can only occur after the preceding FinalConstruct call has completed
    // successfully (since any FinalConstruct failure would have caused
    // CComCreator<T>::CreateInstance to abort without ever trying the QI); since
    // CPimcManager::FinalConstruct always locks the "m_managerLock", this implies that the
    // "m_managerLock" is guaranteed to be locked when this code runs (which is exactly
    // opposite to what happens in all successful usage cases as discussed above).
    //
    // In this case, it is crucial to unlock "m_managerLock" before allowing this CPimcManager
    // object to be destroyed.  Without the unlock, this CPimcManager object would be destroyed
    // while the associated CStdIdentity in the OS code still holds a reference to it; during
    // any future apartment unload, the OS code would release this reference, and the release
    // would be a use-after-free at that point.
    //
    // Note that the crucial unlock causes overactive ATL debug asserts to fire if a chk build
    // of this DLL is used; specifically:
    //
    //    - The code in the CComObject<CPimcManager> destructor always stomps the refcount to
    //      0xc0000001 (i.e., "-(LONG_MAX/2)"), meaning this CPimcManager object's refcount is
    //      always 0xc0000001 when this code runs; unlocking "m_managerLock" will cause the refcount
    //      to drop by one (because, as discussed above, the crucial operation which prevents
    //      use-after-free problems will release the associated CStdIdentity's reference to this
    //      CPimcManager object, and in this way releases the reference that was added when
    //      "managerLock" was locked during FinalConstruct); as a result, unlocking "m_managerLock"
    //      will move this CPimcManager object's refcount through a "0xc0000001 -> 0xc0000000"
    //      transition.
    //
    //    - Both of the CComObjectRootEx<T>::InternalRelease specializations contain debug asserts
    //      which will fire whenever the refcount drops below 0xc0000001, so this transition always
    //      triggers a debug assert when using a chk build of this DLL.
    //
    //    - That said, all evidence strongly suggests that this is just an overactive assert in
    //      the ATL code (probably just indicating that it is rare for FinalConstruct to add
    //      "self-references" like it does for CPimcManager (since these self-references generally
    //      prevent the server object from being destroyed unless a manual action like the
    //      RELEASE_MANAGER_EXT message is taken later on), meaning it is rare to have a situation
    //      where FinalRelease needs to release self-references that were acquired in
    //      FinalConstruct, meaning this is a rare enough case that the ATL authors either didn't
    //      test it or didn't think it was common enough to warrant adjusting the assert).
    //
    // Since this change is being made in servicing, attempt to change behavior as little as
    // possible in the "successful usage" cases where "m_managerLock" is already unlocked,
    // while still ensuring that FinalRelease will always run the crucial unlock in all
    // "unsuccessful usage" cases.
```




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。