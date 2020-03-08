# asp dotnet core 提示 Cannot access a disposed object 解决方法

我在写一个简单的文件服务器，想要用来做客户端下载器的测试服务器，但是返回的方法提示 ObjectDisposedException: Cannot access a disposed object. Object name: 'Cannot access a closed file.' 原因是我的文件被释放

<!--more-->
<!-- CreateTime:2020/1/30 16:55:33 -->

<!-- 发布 -->

在开发端访问链接可以返回一些提示，如我使用下面代码做一个文件下载服务器

```
    [ApiController]
    public class FileDownloadController:ControllerBase
    {
        [HttpGet("download")]
        public IActionResult Download()
        {
            var file = @"F:\win10.14926.1000.160910-1529.RS_PRERELEASE_CLIENTPRO_OEMRET_X64FRE_ZH-CN.ISO";

            using var fileStream = new FileStream(file, FileMode.Open);

            return File(fileStream, MimeType, "win10.14926.1000.160910-1529.RS_PRERELEASE_CLIENTPRO_OEMRET_X64FRE_ZH-CN.ISO");
        }

        private const string MimeType = "application/octet-stream";
    }
```

上面代码我返回一个大的文件，但是访问 `https://localhost:5001/download` 会提示文件被释放

```
ObjectDisposedException: Cannot access a disposed object.
Object name: 'Cannot access a closed file.'.
System.IO.FileStream.BeginRead(byte[] array, int offset, int numBytes, AsyncCallback callback, object state)

ObjectDisposedException: Cannot access a disposed object. Object name: 'Cannot access a closed file.'.
System.IO.FileStream.BeginRead(byte[] array, int offset, int numBytes, AsyncCallback callback, object state)
System.IO.Stream+<>c.<BeginEndReadAsync>b__48_0(Stream stream, ReadWriteParameters args, AsyncCallback callback, object state)
System.Threading.Tasks.TaskFactory<TResult>.FromAsyncTrim<TInstance, TArgs>(TInstance thisRef, TArgs args, Func<TInstance, TArgs, AsyncCallback, object, IAsyncResult> beginMethod, Func<TInstance, IAsyncResult, TResult> endMethod)
System.IO.Stream.BeginEndReadAsync(byte[] buffer, int offset, int count)
System.IO.FileStream.ReadAsync(byte[] buffer, int offset, int count, CancellationToken cancellationToken)
Microsoft.AspNetCore.Http.StreamCopyOperationInternal.CopyToAsync(Stream source, Stream destination, Nullable<long> count, int bufferSize, CancellationToken cancel)
Microsoft.AspNetCore.Mvc.Infrastructure.FileResultExecutorBase.WriteFileAsync(HttpContext context, Stream fileStream, RangeItemHeaderValue range, long rangeLength)
Microsoft.AspNetCore.Mvc.Infrastructure.FileStreamResultExecutor.ExecuteAsync(ActionContext context, FileStreamResult result)
Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeNextResultFilterAsync>g__Awaited|29_0<TFilter, TFilterAsync>(ResourceInvoker invoker, Task lastTask, State next, Scope scope, object state, bool isCompleted)
Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.Rethrow(ResultExecutedContextSealed context)
Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.ResultNext<TFilter, TFilterAsync>(ref State next, ref Scope scope, ref object state, ref bool isCompleted)
Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.InvokeResultFilters()
Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeFilterPipelineAsync>g__Awaited|19_0(ResourceInvoker invoker, Task lastTask, State next, Scope scope, object state, bool isCompleted)
Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeAsync>g__Awaited|17_0(ResourceInvoker invoker, Task task, IDisposable scope)
Microsoft.AspNetCore.Routing.EndpointMiddleware.<Invoke>g__AwaitRequestTask|6_0(Endpoint endpoint, Task requestTask, ILogger logger)
Microsoft.AspNetCore.Authorization.AuthorizationMiddleware.Invoke(HttpContext context)
Microsoft.AspNetCore.Diagnostics.DeveloperExceptionPageMiddleware.Invoke(HttpContext context)
```

因为我在返回之前用了 using 也就是释放 FileStream 资源

在 C# 8.0 提供了让 using 放在对象创建之前，可以在对象作用范围结束自动释放对象，也就是下面代码是相同的

```
            using var fileStream = new FileStream(file, FileMode.Open);

            return File(fileStream, MimeType, "win10.14926.1000.160910-1529.RS_PRERELEASE_CLIENTPRO_OEMRET_X64FRE_ZH-CN.ISO");

// 和这个代码是相同的

            using (var fileStream = new FileStream(file, FileMode.Open))
            {
                return File(fileStream, MimeType, "win10.14926.1000.160910-1529.RS_PRERELEASE_CLIENTPRO_OEMRET_X64FRE_ZH-CN.ISO");
            }
```

在返回 File 方法之后将会释放 fileStream 但是在 asp dotnet core 返回给客户端的信息是在 Download 方法之后，也就是在结束 Download 方法之后读取 FileStream 内容，读取一个被释放的 FileStream 会提示不能读取文件

解决方法就是去掉 using 就可以了

最简单返回一个文件的方法是通过 PhysicalFile 方法，请看代码

```
        [HttpGet("download")]
        public IActionResult Download()
        {
            var file = @"F:\win10.14926.1000.160910-1529.RS_PRERELEASE_CLIENTPRO_OEMRET_X64FRE_ZH-CN.ISO";

            return PhysicalFile(file, MimeType);
        }

        private const string MimeType = "application/octet-stream";
```

在 PhysicalFile 处理了文件的自动释放等问题，使用很简单，但是我发现这货存在内存泄漏，请看 [Maybe PhysicalFile will be memory leak](https://github.com/dotnet/aspnetcore/issues/13535)

使用 PhysicalFile 等方法可以快速实现断点续传的功能

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
