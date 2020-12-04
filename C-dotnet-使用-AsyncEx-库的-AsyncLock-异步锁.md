
# C# dotnet 使用 AsyncEx 库的 AsyncLock 异步锁

本文来安利大家一个好用的库 AsyncEx 库。这个库有一个强大的功能是支持 AsyncLock 异步锁。小伙伴都知道，在 C# 里面的 lock 关键字，在 2020 年也就是 C# 9.0 都没有让这个关键字能支持锁内部添加异步方法。此时在一些需要做异步线程安全的业务就比较坑了，而 AsyncEx 库提供了 AsyncLock 异步锁的功能，可以支持在锁里面使用异步

<!--more-->


<!-- CreateTime:2020/9/23 8:58:31 -->




使用之前先安装 [Nito.AsyncEx](http://www.nuget.org/packages/Nito.AsyncEx) NuGet 库，如果是 SDK 风格的项目格式，可以在 csproj 添加下面代码

```xml
<PackageReference Include="Nito.AsyncEx" Version="5.0.0" />
```

简单的使用方法如下

```csharp
private readonly AsyncLock _mutex = new AsyncLock();
public async Task UseLockAsync()
{
  using (await _mutex.LockAsync())
  {
    await Task.Delay(TimeSpan.FromSeconds(1));
  }
}
```

而 AsyncLock 其实是 [Nito.AsyncEx](http://www.nuget.org/packages/Nito.AsyncEx) 库的基础，在 AsyncEx 库还包括了  `AsyncManualResetEvent`, `AsyncAutoResetEvent`, `AsyncConditionVariable`, `AsyncMonitor`, `AsyncSemaphore`, `AsyncCountdownEvent` 和 `AsyncReaderWriterLock` 的实现







<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。