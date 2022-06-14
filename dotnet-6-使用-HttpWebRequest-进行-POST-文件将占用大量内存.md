
# dotnet 6 使用 HttpWebRequest 进行 POST 文件将占用大量内存

我有用户给我报告一个内存不足的问题，经过了调查，找到了依然是使用已经被标记过时的 HttpWebRequest 进行文件推送，推送过程中，由于 System.Net.RequestStream 将会完全将推送的文件全部读取到内存，导致了在 x86 应用下，推送超过 500MB 的文件，基本上都会抛出 OutOfMemoryException 异常

<!--more-->


<!-- CreateTime:2022/6/13 8:04:47 -->

<!-- 发布 -->
<!-- 博客 -->

这是一个 .NET Core 和 .NET Framework 行为的差异。在 .NET Framework 下，调用 WebRequest.Create 方法创建一个 HttpWebRequest 对象，使用 HttpWebRequest 对象调用 GetRequestStream 方法即可获取请求的 Stream 用于写入数据，写入的数据可以是一个文件的信息

在 .NET Framework 下，将会在 GetRequestStream 方法时，尝试和服务器建立连接。对 RequestStream 写入内容，将会发送给到服务器。然而在 .NET Core 里面，这个逻辑和网络优化是冲突的，而且 HttpWebRequest 这个 API 设计本身就存在缺陷。为了让 dotnet 底层的网络通讯方式统一，在 dotnet core 3.1 及更高版本，让 HttpWebRequest 底层走的和 HttpClient 相同的逻辑。当然，我没有考古 dotnet core 3.1 以前的故事

在 dotnet 6 下，调用 GetRequestStream 方法时，将不会立刻和服务器建立连接，这是和 dotnet framework 最大的不同。在 dotnet 6 下，调用 GetRequestStream 方法将立刻返回一个 System.Net.RequestStream 对象，大概代码如下

```csharp
        public override Stream GetRequestStream()
        {
            return InternalGetRequestStream().Result;
        }

        private Task<Stream> InternalGetRequestStream()
        {
            _requestStream = new RequestStream();

            return Task.FromResult((Stream)_requestStream);
        }
```

对 System.Net.RequestStream 对象进行写入时，由于 dotnet 6 下的 GetRequestStream 不会和服务器建立连接，因此写入的数据也不会立刻发送给服务器。这也就是大家将会发现在 dotnet 6 下调用 GetRequestStream 方法将会返回特别快速的原因

既然 RequestStream 不会立刻发送出去，为了不丢失数据，就只能缓存到内存。大家看看 RequestStream 的实现是多么简单，以下代码就是从 dotnet 官方仓库拷贝的，删除了部分不重要的逻辑。可以看到在 RequestStream 的实现里面，其实就是封装一个 MemoryStream 而已，而且只支持写入，写入的内容就放入到 MemoryStream 里面

```csharp
namespace System.Net
{
    // Cache the request stream into a MemoryStream.  This is the
    // default behavior of Desktop HttpWebRequest.AllowWriteStreamBuffering (true).
    // Unfortunately, this property is not exposed in .NET Core, so it can't be changed
    // This will result in inefficient memory usage when sending (POST'ing) large
    // amounts of data to the server such as from a file stream.
    internal sealed class RequestStream : Stream
    {
        private readonly MemoryStream _buffer = new MemoryStream();

        public RequestStream()
        {
        }

        public override void Flush()
        {
            // Nothing to do.
        }

        public override Task FlushAsync(CancellationToken cancellationToken)
        {
            // Nothing to do.
            return cancellationToken.IsCancellationRequested ?
                Task.FromCanceled(cancellationToken) :
                Task.CompletedTask;
        }

        public override long Length
        {
            get
            {
                throw new NotSupportedException();
            }
        }

        public override long Position
        {
            get
            {
                throw new NotSupportedException();
            }
            set
            {
                throw new NotSupportedException();
            }
        }

        public override int Read(byte[] buffer, int offset, int count)
        {
            throw new NotSupportedException();
        }

        public override void Write(byte[] buffer, int offset, int count)
        {
            ValidateBufferArguments(buffer, offset, count);
            _buffer.Write(buffer, offset, count);
        }

        public override Task WriteAsync(byte[] buffer, int offset, int count, CancellationToken cancellationToken)
        {
            ValidateBufferArguments(buffer, offset, count);
            return _buffer.WriteAsync(buffer, offset, count, cancellationToken);
        }

        public override IAsyncResult BeginWrite(byte[] buffer, int offset, int count, AsyncCallback? asyncCallback, object? asyncState)
        {
            ValidateBufferArguments(buffer, offset, count);
            return _buffer.BeginWrite(buffer, offset, count, asyncCallback, asyncState);
        }

        public override void EndWrite(IAsyncResult asyncResult)
        {
            _buffer.EndWrite(asyncResult);
        }

        public ArraySegment<byte> GetBuffer()
        {
            ArraySegment<byte> bytes;

            bool success = _buffer.TryGetBuffer(out bytes);
            Debug.Assert(success); // Buffer should always be visible since default MemoryStream constructor was used.

            return bytes;
        }
    }
}
```

也如上面代码的注释，在 .NET 6 使用此方法 POST 一段大一点的数据，将会非常的浪费内存。这就是上文说的，对于 x86 应用来说，如果发送一个超过 500MB 的文件，基本上都会抛出内存不足。使用 MemoryStream 时，申请的内存都是两倍两倍申请的，超过 500MB 的数据，将会在 MemoryStream 申请 1GB 的内存空间，对于 x86 的应用来说，基本上能用的内存就是只有 2GB 空间，就为了上传一个文件，申请一段 1GB 的连续空间，对大部分应用来说，即使现在剩余的空间还有超过 1GB 但是剩余的空间却不是连续的，存在一定内存碎片

大家可以看到在 RequestStream 里面，连读取的方法都标记不可用，那在什么使用用到呢。可以看到 RequestStream 多实现了 GetBuffer 方法，这个方法将可以获取所有的数据

在调用 GetResponse 时，才会真的使用 RequestStream 的数据。在 dotnet 6 的调用 GetResponse 方法实现如下

```csharp
        public override WebResponse GetResponse()
        {
            try
            {
                _sendRequestCts = new CancellationTokenSource();
                return SendRequest(async: false).GetAwaiter().GetResult();
            }
            catch (Exception ex)
            {
                throw WebException.CreateCompatibleException(ex);
            }
        }
```

底层调用的是 SendRequest 方法，咱再来看看这个方法是如何使用 RequestStream 数据

```csharp
        private async Task<WebResponse> SendRequest(bool async)
        {
            var request = new HttpRequestMessage(new HttpMethod(_originVerb), _requestUri);

            bool disposeRequired = false;
            HttpClient? client = null;
            try
            {
                client = GetCachedOrCreateHttpClient(async, out disposeRequired);
                if (_requestStream != null)
                {
                	// 在这里使用到 RequestStream 数据
                    ArraySegment<byte> bytes = _requestStream.GetBuffer();
                    request.Content = new ByteArrayContent(bytes.Array!, bytes.Offset, bytes.Count);
                }

                // Copy the HttpWebRequest request headers from the WebHeaderCollection into HttpRequestMessage.Headers and
                // HttpRequestMessage.Content.Headers.
                foreach (string headerName in _webHeaderCollection)
                {
                    // The System.Net.Http APIs require HttpRequestMessage headers to be properly divided between the request headers
                    // collection and the request content headers collection for all well-known header names.  And custom headers
                    // are only allowed in the request headers collection and not in the request content headers collection.
                    // 拷贝 Head 逻辑
                }

                request.Headers.TransferEncodingChunked = SendChunked;

                _sendRequestTask = async ?
                    client.SendAsync(request, _allowReadStreamBuffering ? HttpCompletionOption.ResponseContentRead : HttpCompletionOption.ResponseHeadersRead, _sendRequestCts!.Token) :
                    Task.FromResult(client.Send(request, _allowReadStreamBuffering ? HttpCompletionOption.ResponseContentRead : HttpCompletionOption.ResponseHeadersRead, _sendRequestCts!.Token));

                HttpResponseMessage responseMessage = await _sendRequestTask.ConfigureAwait(false);

                HttpWebResponse response = new HttpWebResponse(responseMessage, _requestUri, _cookieContainer);

                return response;
            }
            finally
            {
                if (disposeRequired)
                {
                    client?.Dispose();
                }
            }
        }
```

可以看到在 HttpWebRequest 底层是通过 HttpClient 来发送网络请求，在如上面代码注释，将 RequestStream 的数据取出作为 ByteArrayContent 进行发送。这是一个很浪费的行为，因为如果能直接使用 HttpClient 进行网络请求，那直接使用 Stream 即可，可以减少一次内存的拷贝和内存占用

也如上面代码，可以看到，完全可以使用 HttpClient 代替 HttpWebRequest 的调用。而且也如上面代码，可以看到 HttpWebRequest 是将请求存放在 `_requestStream` 字段，天然就不支持复用，从性能和 API 设计，都不如 HttpClient 好用

本文测试代码放在[github](https://github.com/lindexi/lindexi_gd/tree/7a8217d8c6f6915360f1e25b06f3166c955b8e0e/BujeardalljelKaifeljaynaba) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/7a8217d8c6f6915360f1e25b06f3166c955b8e0e/BujeardalljelKaifeljaynaba) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 7a8217d8c6f6915360f1e25b06f3166c955b8e0e
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 BujeardalljelKaifeljaynaba 文件夹

那此内存大量占用问题可以如何解决呢？十分简单，换成 HttpClient 即可

原本 HttpWebRequest 底层就是调用 HttpClient 实现发送网络请求，由因为 HttpWebRequest 的 API 限制，导致了只能将文件的数据先全部读取到内存，再进行发送。如果换成 HttpClient 的话，扔一个 StreamContent 进去即可





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。