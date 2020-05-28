
# dotnet ConditionalWeakTable 的底层原理

在 dotnet 中有一个特殊的类，这个类能够做到附加属性一样的功能。也就是给某个对象附加一个属性，当这个对象被回收的时候，自然解除附加的属性的对象的引用。本文就来聊聊这个类的底层原理

<!--more-->


<!-- CreateTime:5/26/2020 10:04:23 AM -->

<!-- 发布 -->

小伙伴都知道弱缓存是什么，弱缓存的核心是弱引用。也就是我虽然拿到一个对象，但是我没有给这个对象添加依赖引用，也就是这个对象不会记录被弱引用的引用。而 ConditionalWeakTable 也是一个弱缓存只是有些特殊的是关联的是其他对象。使用方法请看 [.NET/C# 使用 ConditionalWeakTable 附加字段（CLR 版本的附加属性，也可用用来当作弱引用字典 WeakDictionary） - walterlv](https://blog.walterlv.com/post/conditional-weak-table.html )

这个类一般用来做弱缓存字典，只要 Key 没有被回收，而 value 就不会被回收。如果 key 被回收，那么 value 将会减去一个依赖引用。而字典对于 key 是弱引用

通过阅读 runtime 的源代码，可以看到实际上这个类的核心需要 DependentHandle 结构体的支持，因为依靠 key 定住 value 需要 CLR 的 GC 支持。什么是依靠 key 定住 value 的功能？这里的定住是 Pin 的翻译，意思是如果 key 存在内存，那么将会给 value 添加一个引用，此时的 value 将不会被回收。而如果 key 被回收了，此时的 value 将失去 key 对他的强引用

换句话说，只要 key 的值存在，那么 value 一定不会回收

这个功能纯使用 WeakReference 是做不到的，需要 GC 的支持，而在 dotnet core 里面提供 GC 支持的对接的是 DependentHandle 结构体

那么 DependentHandle 的功能又是什么？这个结构体提供传入 `object primary, object? secondary` 构造函数，作用就是当 primary 没有被回收的时候，给 `secondary` 添加一个引用计数。在 primary 回收的时候，解除对 secondary 的引用。而这个结构体本身对于 primary 是弱引用的，对于 secondary 仅在 primary 没有被回收时是强引用，当 primary 被回收之后将是弱引用

刚好利用 GC 的只要对象至少有一个引用就不会被回收的功能，就能做到 ConditionalWeakTable 提供附加属性的功能

下面代码是 DependentHandle 结构体的代码，可以看到大量的方法都是需要 GC 层的支持，属于 CLR 部分的注入方法

```csharp
    internal struct DependentHandle
    {
        private IntPtr _handle;

        public DependentHandle(object primary, object? secondary) =>
            // no need to check for null result: nInitialize expected to throw OOM.
            _handle = nInitialize(primary, secondary);

        public bool IsAllocated => _handle != IntPtr.Zero;

        // Getting the secondary object is more expensive than getting the first so
        // we provide a separate primary-only accessor for those times we only want the
        // primary.
        public object? GetPrimary() => nGetPrimary(_handle);

        public object? GetPrimaryAndSecondary(out object? secondary) =>
            nGetPrimaryAndSecondary(_handle, out secondary);

        public void SetPrimary(object? primary) =>
            nSetPrimary(_handle, primary);

        public void SetSecondary(object? secondary) =>
            nSetSecondary(_handle, secondary);

        // Forces dependentHandle back to non-allocated state (if not already there)
        // and frees the handle if needed.
        public void Free()
        {
            if (_handle != IntPtr.Zero)
            {
                IntPtr handle = _handle;
                _handle = IntPtr.Zero;
                nFree(handle);
            }
        }

        [MethodImpl(MethodImplOptions.InternalCall)]
        private static extern IntPtr nInitialize(object primary, object? secondary);

        [MethodImpl(MethodImplOptions.InternalCall)]
        private static extern object? nGetPrimary(IntPtr dependentHandle);

        [MethodImpl(MethodImplOptions.InternalCall)]
        private static extern object? nGetPrimaryAndSecondary(IntPtr dependentHandle, out object? secondary);

        [MethodImpl(MethodImplOptions.InternalCall)]
        private static extern void nSetPrimary(IntPtr dependentHandle, object? primary);

        [MethodImpl(MethodImplOptions.InternalCall)]
        private static extern void nSetSecondary(IntPtr dependentHandle, object? secondary);

        [MethodImpl(MethodImplOptions.InternalCall)]
        private static extern void nFree(IntPtr dependentHandle);
    }
```

而核心实现的入口是在 gchandletable.cpp 的 `OBJECTHANDLE GCHandleStore::CreateDependentHandle(Object* primary, Object* secondary)` 代码，这部分属于更底的一层了，在功能上就是实现上面的需求，而实现上为了性能优化，代码可读性还是渣了一些

要实现这个功能需要在 GC 层里面写上一大堆的代码，但使用上现在仅有 ConditionalWeakTable 一个在使用





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。