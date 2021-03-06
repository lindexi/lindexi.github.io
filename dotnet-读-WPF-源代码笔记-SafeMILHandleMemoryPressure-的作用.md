
# dotnet 读 WPF 源代码笔记 SafeMILHandleMemoryPressure 的作用

本文来告诉大家在 WPF 里面的 SafeMILHandleMemoryPressure 类的作用。这是一个 internal 不开放的类，是在 WPF 中和 Dx 等模块调用使用的，用途就是辅助 GC 统计当前内存情况，用来在内存不够的时候触发回收

<!--more-->


<!-- CreateTime:2020/12/25 9:09:44 -->


<!-- 发布 -->

这个类放在 `src\Microsoft.DotNet.Wpf\src\PresentationCore\System\Windows\Media\SafeMILHandleMemoryPressure.cs` 文件，核心调用是通过 [GC.AddMemoryPressure(Int64)](https://docs.microsoft.com/en-us/dotnet/api/system.gc.addmemorypressure?WT.mc_id=WD-MVP-5003260 ) 方法告诉 GC 当前非托管部分占用了多少内存

根据 [GC.AddMemoryPressure(Int64) 官方文档](https://docs.microsoft.com/en-us/dotnet/api/system.gc.addmemorypressure?WT.mc_id=WD-MVP-5003260 ) 的说法，这个 AddMemoryPressure 需要和 RemoveMemoryPressure 成对使用，在使用的时候必须由业务方成对调用，否则将会影响 GC 的效率

为什么需要有 [GC.AddMemoryPressure](https://docs.microsoft.com/en-us/dotnet/api/system.gc.addmemorypressure?WT.mc_id=WD-MVP-5003260 ) 这个方法？原因是假定咱的所有代码都是托管的清真的代码，那么 GC 是能统计当前占用了多少的内存的。但如果咱调用了一些非托管部分，这些模块也申请了内存，此时的 GC 是不了解当前使用到多少内存的，属于这个非托管模块用的内存是多少。通过 [GC.AddMemoryPressure](https://docs.microsoft.com/en-us/dotnet/api/system.gc.addmemorypressure?WT.mc_id=WD-MVP-5003260 ) 这个方法可以告诉 GC 当前这个非托管模块使用到多少内存了

而 GC 的清理是需要根据当前内存占用量决定的，假定现在内存多的是，而且进程也没有用多少内存，那么 GC 将不会进行全清理。但如果当前进程用到了大量的内存了，那么 GC 也许就需要考虑来一次完全内存回收了。上面说的内存完全回收大概可以理解为回收到二代同时压缩内存，更多内存细节请看伟民哥翻译的 .NET内存管理宝典 - 提高代码质量、性能和可扩展性 这本书

那如果我只是调用了 [GC.AddMemoryPressure](https://docs.microsoft.com/en-us/dotnet/api/system.gc.addmemorypressure?WT.mc_id=WD-MVP-5003260 ) 但没有调用 RemoveMemoryPressure 方法会如何？此时的 GC 将会以为内存里面有这些模块占用了内存，而且这些模块也没有释放

为了能在 WPF 里面更好管理内存，同时成对调用 [GC.AddMemoryPressure](https://docs.microsoft.com/en-us/dotnet/api/system.gc.addmemorypressure?WT.mc_id=WD-MVP-5003260 ) 和 RemoveMemoryPressure 方法，而且是准确在非托管释放的时候调用 RemoveMemoryPressure 方法，就封装了 SafeMILHandleMemoryPressure 类

在 SafeMILHandleMemoryPressure 的构造函数里面，将会传入当前非托管模块使用到的内存量

```csharp
        internal SafeMILHandleMemoryPressure(long gcPressure)
        {
            _gcPressure = gcPressure;
            _refCount = 0;

            
            // Removed WPF specific GC algorithm and all bitmap allocations/deallocations
            // are now tracked with GC.Add/RemoveMemoryPressure.
            GC.AddMemoryPressure(_gcPressure);
        }
```

接着跟随非托管的指针引用添加或减少引用，相当于自己实现了引用计算。在引用数量为 零 的时候，调用 RemoveMemoryPressure 方法告诉 GC 非托管没有占用资源

```csharp
        internal void AddRef()
        {
            Interlocked.Increment(ref _refCount);
        }

        internal void Release()
        {
            if (Interlocked.Decrement(ref _refCount) == 0)
            {
                
                // Removed WPF specific GC algorithm and all bitmap allocations/deallocations
                // are now tracked with GC.Add/RemoveMemoryPressure.
                GC.RemoveMemoryPressure(_gcPressure);
                _gcPressure = 0;
            }
        }

        // Estimated size in bytes of the unmanaged memory
        private long _gcPressure;

        //
        // SafeMILHandleMemoryPressure does its own ref counting in managed code, because the
        // associated memory pressure should be removed when there are no more managed
        // references to the unmanaged resource. There can still be references to it from
        // unmanaged code elsewhere, but that should not prevent the memory pressure from being
        // released.
        //
        private int _refCount;
```

当前这个类只是在和 MIL 调用这里使用，但设计是通用的

[GC.AddMemoryPressure(Int64) Method (System)](https://docs.microsoft.com/en-us/dotnet/api/system.gc.addmemorypressure?WT.mc_id=WD-MVP-5003260)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。