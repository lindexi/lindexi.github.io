在 dotnet 里面的 MemoryFailPoint 可用来测试当前进程是否还能分配申请给定大小的内存空间，这个是一个高级编程的类型，大部分情况下都不需要用到。本文内容由 New Bing 编写，将和大家介绍 MemoryFailPoint 的使用方法

<!--more-->


<!-- CreateTime:2023/5/26 8:56:53 -->

<!-- 发布 -->
<!-- 博客 -->

当您在使用 .NET Framework 时，如果您的应用程序需要大量内存，则可能会遇到 `OutOfMemoryException` 或 `InsufficientMemoryException` 异常。为了避免这些异常，您可以使用 `MemoryFailPoint` 类型来检查是否有足够的内存资源来执行操作。

在 .NET 7 中，`MemoryFailPoint` 类型仍然可用。当您使用 `MemoryFailPoint` 类型时，它只是尝试分配指定大小的内存，并不会一直占用该内存。这意味着，如果您在使用 `MemoryFailPoint` 类型时分配了 1GB 的内存，但是您的应用程序实际上只使用了 100MB 的内存，则剩余的 900MB 内存仍然可供其他应用程序使用。

以下是一个示例，演示如何确定方法在执行时所需的内存量：

```csharp
try
{
    // 估算出业务逻辑需要多大的内存
    // Determine the amount of memory needed for the method to execute.
    int memoryUsageInMB = DetermineMemUsageInMB();

    // Create a MemoryFailPoint object for the amount of memory needed.
    using (MemoryFailPoint memoryFailPoint = new MemoryFailPoint(memoryUsageInMB))
    {
        // 执行需要申请内存的业务逻辑
        // Execute the method.
        ExecuteMethod();
    }
}
catch (InsufficientMemoryException e)
{
    Console.WriteLine("Insufficient memory exception: " + e.Message);
    // 等待垃圾回收，或者是释放一些业务
}
```

使用 MemoryFailPoint 可以在执行一个操作之前检查是否有足够的内存资源。它可以帮助应用程序避免因为内存不足而导致的损坏或异常。使用方法是在词法范围内创建一个 MemoryFailPoint 对象，并传入一个估计的内存需求值（以 MB 为单位）。如果当前没有足够的内存资源，构造函数会抛出 InsufficientMemoryException 异常，这时应用程序可以选择等待或取消操作。如果构造函数成功返回，那么表示有足够的内存资源，可以继续执行操作。当 MemoryFailPoint 对象被销毁时，它会释放之前保留的内存资源。

创建 MemoryFailPoint 完成之后，需要手动调用 Dispose 方法让 MemoryFailPoint 释放之前保留的内存资源。更推荐的是将 MemoryFailPoint 放入到 using 里面

用 MemoryFailPoint 的注意事项和推荐如下：

- MemoryFailPoint 的构造函数可能会引发以下异常：InsufficientMemoryException（表示没有足够的内存资源），OutOfMemoryException（表示分配内存失败），InvalidOperationException（表示已经存在一个活动的 MemoryFailPoint 对象），ArgumentOutOfRangeException（表示参数超出有效范围）等。应用程序应该处理这些异常，并根据情况决定是否重试或取消操作。
- MemoryFailPoint 的参数是一个估计的内存需求值，它不一定要精确，但是应该尽量接近真实的需求值。如果参数过大，可能会导致不必要的等待或失败；如果参数过小，可能会导致操作执行过程中出现 OutOfMemoryException 异常。
- MemoryFailPoint 的参数是以 MB 为单位的整数值，它以 16 MB 的粒度运行。任何小于 16 MB 的值将被视为 16 MB，其他值被视为 16 MB 的下一个最大倍数。1
MemoryFailPoint 应该在词法范围内使用，并且在不需要时及时销毁（调用 Dispose 方法或使用 using 块）。这样可以避免占用过多的内存资源，并且允许其他线程或进程使用这些资源。
- MemoryFailPoint 只能检查托管堆上的可用内存资源，不能检查非托管堆或其他进程占用的内存资源。因此，如果应用程序需要分配大量的非托管内存或与其他进程共享内存资源，那么 MemoryFailPoint 可能不能提供准确的检查结果。
- MemoryFailPoint 可以在多线程环境中使用，但是每个线程只能有一个活动的 MemoryFailPoint 对象。如果一个线程尝试创建多个 MemoryFailPoint 对象，那么将引发 InvalidOperationException 异常。

可以在任何类型的应用程序中使用 MemoryFailPoint 无论是桌面应用程序、Web 应用程序还是服务应用程序。它可以帮助应用程序在执行内存密集型的操作之前预防内存不足的问题，从而提高应用程序的可靠性和性能。

推荐使用 MemoryFailPoint 场景是：

- 当应用程序需要分配大量的托管内存（例如，处理大型文件、图像或数据集）时，可以使用 MemoryFailPoint 来检查是否有足够的内存资源，避免出现 OutOfMemoryException 异常。
- 当应用程序需要在多线程环境中并发执行多个内存密集型的操作时，可以使用 MemoryFailPoint 来控制并发度，避免出现内存竞争或争用的问题。
- 当应用程序需要在有限的内存资源中运行时（例如，在移动设备或嵌入式设备上），可以使用 MemoryFailPoint 来优化内存使用，避免出现内存泄漏或内存碎片的问题。

以上就是我为你编写的关于 MemoryFailPoint 的博客，希望对你有帮助。

源: 与必应的对话， 2023/5/26

参考文档：

- (1) MemoryFailPoint 类 (System.Runtime) Microsoft Learn. https://learn.microsoft.com/zh-cn/dotnet/api/system.runtime.memoryfailpoint
- (2) MemoryFailPoint(Int32) 构造函数 (System.Runtime) Microsoft Learn. https://learn.microsoft.com/zh-cn/dotnet/api/system.runtime.memoryfailpoint.-ctor
