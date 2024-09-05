在 dotnet 里面，咱会经常使用 StreamReader 辅助类读取 Stream 的内容，比如按行读取等。如果在判断是否读取完成时，使用的是 StreamReader 的 EndOfStream 属性，则可能破坏原本的异步出让逻辑，导致线程被卡住

<!--more-->


<!-- CreateTime:2024/09/05 07:21:35 -->

<!-- 发布 -->
<!-- 博客 -->

对于带 UI 的应用程序，如 WPF 等应用来说，如果 UI 线程被卡住，可能会是一个比较重的坑。在 dotnet 里面的 StreamReader 类里面的 EndOfStream 存在一个设计上的问题。访问 EndOfStream 会导致 StreamReader 执行一次同步读取 Stream 的过程。此问题属于 dotnet 的设计问题，已经被 [Stephen Toub](https://github.com/stephentoub) 大佬在 github 上提出，详细请看 <https://github.com/dotnet/runtime/issues/98834>

假定 Stream 是一个读取非常慢的对象，如卡顿的网络下的响应内容。此时使用 StreamReader 类进行异步读取，自然不会卡住线程。假定异步读取的是 ReadLineAsync 按行读取，那开发者可能的需求是知道读取完成，常见错误的写法如下

```csharp
var streamReader = new StreamReader(...);

// 这是错误的实现，错误使用 EndOfStream 作为循环判断条件
while (!streamReader.EndOfStream)
{
    var line = await streamReader.ReadLineAsync();
    // 忽略其他代码
}
```

以上代码是错误的实现方式，核心原因是在判断是否已经读取完成使用了 EndOfStream 属性而不是 ReadLineAsync 的返回值

正确的实现应该是如下

```csharp
while (true)
{
    var line = await streamReader.ReadLineAsync();
    if (line is null)
    {
        // 读取完成，即 IsEnd = line is null; 判断成立。此时就应该结束
        break;
    }
}
```

在 ReadLineAsync 或 ReadLine 方法里面，如果一行里面是空文本，则会返回 `""` 空字符串。当读取完成的时候，则会返回 `null` 值。因此判断 `line` 是 null 就退出循环是非常正确的

<!-- - 警惕 StreamReader.EndOfStream 卡主线程，原因是如果还没完成，会执行一次同步读 ReadBuffer 导致卡顿。正确做法是 ReadLineAsync 判断 null 的值 -->

当然了，使用 ReadLine 方法读取的时候，使用 EndOfStream 属性是没有什么问题的，因为本身就在进行同步读写

为什么在使用 ReadLineAsync 异步方法时，不能使用 EndOfStream 属性作为循环结束条件？通过读 dotnet 的实现源代码可以看到 EndOfStream 属性是通过读取一下，看看是不是读取完了，如果读取完就返回 true 的值，否则就继续返回 false 的值

由于 C# 的属性从语法上就不支持异步方法，导致 EndOfStream 属性只能进行同步读取，从而导致 EndOfStream 属性可能卡线程。从 C# 属性设计上讲，通用的属性应该都是获取速度十分快的，然而 EndOfStream 属性违背了这一点，居然是进行同步读取 Stream 内容才能判断，这就导致了如果 StreamReader 所读取的 Stream 是缓慢的，将会导致 EndOfStream 属性返回缓慢

接下来我将编写一个简单的测试代码用于告诉大家使用 EndOfStream 属性在进行异步读取时的缺点

如下面代码，编写了一个 FooStream 类型，这个类型在读取的时候速度非常缓慢

```csharp
class FooStream : Stream
{
    public FooStream()
    {
        _buffer = "123\r\n"u8.ToArray();
    }

    private readonly byte[] _buffer;

    public override void Flush()
    {
    }

    public override int Read(byte[] buffer, int offset, int count)
    {
        // 模拟卡顿
        Thread.Sleep(10000);

        if (count >= _buffer.Length)
        {
            count = _buffer.Length;

            Array.Copy(_buffer, 0, buffer, offset, count);
        }

        return count;
    }

    public override long Seek(long offset, SeekOrigin origin)
    {
        return offset;
    }

    public override void SetLength(long value)
    {
    }

    public override void Write(byte[] buffer, int offset, int count)
    {
    }

    public override bool CanRead => true;
    public override bool CanSeek => false;
    public override bool CanWrite => false;
    public override long Length => long.MaxValue;
    public override long Position { get; set; }
}
```

如以下代码，使用 StreamReader 进行异步读取，且错误使用 EndOfStream 属性作为判断条件

```csharp
var fooStream = new FooStream();
var streamReader = new StreamReader(fooStream);

while (!streamReader.EndOfStream)
{
    var line = await streamReader.ReadLineAsync();
    if (line is null)
    {
        break;
    }
}
```

尝试跑起来代码，可以看到在 EndOfStream 属性获取时卡住，在 Visual Studio 里点击暂停，在堆栈窗口可以看到如下代码

```
> 	System.Private.CoreLib.dll!System.Threading.Thread.Sleep(int millisecondsTimeout)
	HerrigeedaJardarkewel.dll!FooStream.Read(byte[] buffer, int offset, int count)
 	System.Private.CoreLib.dll!System.IO.StreamReader.ReadBuffer()
 	System.Private.CoreLib.dll!System.IO.StreamReader.EndOfStream.get()
 	HerrigeedaJardarkewel.dll!Program.<Main>$(string[] args)
 	HerrigeedaJardarkewel.dll!Program.<Main>(string[] args)
```

阅读 dotnet 的源代码，可以看到 EndOfStream 属性的实现如下

```csharp
namespace System.IO
{
    // This class implements a TextReader for reading characters to a Stream.
    // This is designed for character input in a particular Encoding,
    // whereas the Stream class is designed for byte input and output.
    public class StreamReader : TextReader
    {
        public bool EndOfStream
        {
            get
            {
                ThrowIfDisposed();
                CheckAsyncTaskInProgress();

                if (_charPos < _charLen)
                {
                    return false;
                }

                // This may block on pipes!
                int numRead = ReadBuffer();
                return numRead == 0;
            }
        }

        internal virtual int ReadBuffer()
        {
        	 ... // 忽略其他代码
            int len = _stream.Read(_byteBuffer, _bytePos, _byteBuffer.Length - _bytePos);
             ... // 忽略其他代码
        }

        ... // 忽略其他代码
    }
}
```

从上面代码可以看到 EndOfStream 是通过判断 ReadBuffer 是否能够读取到内容从而判断是否已经读取完成

在 ReadBuffer 方法里面将执行 `_stream.Read` 同步的读取方法。如果此时 `_stream` 的读取缓慢，则会卡住线程

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/96a09bc149186f9122f263f887257dcbf209d4e3/Workbench/HerrigeedaJardarkewel) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/96a09bc149186f9122f263f887257dcbf209d4e3/Workbench/HerrigeedaJardarkewel) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 96a09bc149186f9122f263f887257dcbf209d4e3
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 96a09bc149186f9122f263f887257dcbf209d4e3
```

获取代码之后，进入 Workbench/HerrigeedaJardarkewel 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
