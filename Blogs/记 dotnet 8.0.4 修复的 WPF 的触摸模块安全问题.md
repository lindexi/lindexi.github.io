---
title: 记 dotnet 8.0.4 修复的 WPF 的触摸模块安全问题
description: 本文记录 dotnet 8.0.4 版本修复的 WPF 的触摸模块安全问题，此问题影响所有的 .NET 版本，修复方法是更新 SDK 和运行时
tags: WPF,dotnet
category: 
---

<!-- CreateTime:2024/04/12 07:01:47 -->

<!-- 发布 -->
<!-- 博客 -->

宣布安全漏洞地址： <https://github.com/dotnet/wpf/issues/9003>

安全漏洞宣布地址： <https://github.com/dotnet/announcements/issues/303>

漏洞代号： [CVE-2024-21409](https://www.cve.org/CVERecord?id=CVE-2024-21409)

核心更改： <https://github.com/dotnet/wpf/commit/c15b5c68cd74ae28bc99af539d05880658c45024>

影响模块： 触摸模块

开发者侧的修复方法： 升级 .NET SDK 或运行时版本，携带此更新的版本分别如下

- .NET 6 : 6.0.29
- .NET 7 : 7.0.18
- .NET 8 : 8.0.4

微软系统更新 Microsoft Update 将会自动推送以上版本的 .NET Core 更新，以及相应的 .NET Framework 质量更新

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

以下是 new Bing 对上面注释的解释

这段代码注释讨论的是一个关于 **CPimcManager** 类的析构函数（destructor）的问题。让我来解释一下：

1. 首先，我们有一个 **CPimcManager** 类，它的析构函数（destructor）被称为 **FinalRelease**。
2. 在成功使用 **CPimcManager** 的情况下，以下步骤发生：
    - 托管的 WPF 代码使用 **CoCreateInstance** 来获取一个指向全新 **CPimcManager** 实例的 **IPimcManager2** 接口（由 **`ATL CComCreator<T>::CreateInstance`** 机制创建）。
    - 这意味着 **FinalConstruct** 已经成功完成，也就是说，“m_managerLock”已经被锁定。
    - 然后，托管的 WPF 代码通过发送 **RELEASE_MANAGER_EXT** 消息（参见 **UnsafeNativeMethods.ReleaseManagerExternalLock()**）来解锁 “m_managerLock”，表示不再需要 **CPimcManager** 对象。
    - 现在，“m_managerLock”已经解锁，**`CComObject<CPimcManager>`** 对象可以在其引用计数降至零时被销毁，此时 **FinalRelease** 函数将运行。
3. 因此，在所有成功的使用情况下，当此代码运行时，“m_managerLock”已经解锁（因为如果它仍然被锁定，锁本身将阻止引用计数达到零，从而阻止此函数运行）。
4. 但是，在不成功的使用情况下，**`ATL CComCreator<T>::CreateInstance`** 机制可能会失败，这意味着它将在将错误返回给 **CreateInstance** 调用者之前销毁全新的 **CPimcManager** 实例。
5. 销毁全新实例会触发 **`CComObject<CPimcManager>`** 析构函数，因此在 **`CComCreator<T>::CreateInstance`**操作本身期间会调用此函数。
6. **`CComCreator<T>::CreateInstance`** 的最后一步是查询新创建的对象，以获取已重新定义的任何接口。

总之，这段注释详细描述了 **CPimcManager** 类的析构函数在不同使用情况下的行为和保证。
