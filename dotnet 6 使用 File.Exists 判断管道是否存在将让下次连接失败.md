# dotnet 6 使用 File.Exists 判断管道是否存在将让下次连接失败

我尝试在 dotnet 6 使用 File.Exists 判断管道是否存在，如果管道存在再进行连接。然而这个逻辑将会接下来的 NamedPipeClientStream 调用 Connect 连接失败

<!--more-->
<!-- CreateTime:2022/5/23 20:17:28 -->

<!-- 发布 -->
<!-- 博客 -->

这个问题似乎是 CLR 底层的问题，我将问题报告给官方，请看 [Using File.Exists to check the pipe created will make the NamedPipeClientStream connect fail · Issue #69604 · dotnet/runtime](https://github.com/dotnet/runtime/issues/69604 )

使用 File.Exists 判断管道是否存在的代码如下

```csharp
File.Exists(@"\\.\pipe\" + PipeName);
```

然而以上代码将会在 dotnet 6 下让 NamedPipeClientStream 调用 Connect 连接失败。这个逻辑如果放在 .NET Framework 下运行，是能成功的

最简复现步骤如下，先使用 NamedPipeServerStream 启动管道服务

```csharp
void StartServer()
{
    var server = new NamedPipeServerStream(PipeName,
        PipeDirection.InOut, 1, PipeTransmissionMode.Byte, PipeOptions.Asynchronous, 1024, 1024);

    server.BeginWaitForConnection(OnWaitForConnection, server);
}
```

接下来使用 File.Exists 判断管道是否存在

```csharp
File.Exists(@"\\.\pipe\" + PipeName);
```

再使用 NamedPipeClientStream 进行连接

```csharp
void StartClient()
{
    var localServer = ".";
    var pipeDirection = PipeDirection.InOut;
    var client = new NamedPipeClientStream(localServer,
        PipeName, pipeDirection, PipeOptions.Asynchronous);

    var timeout = 1000 * 5;
    client.Connect(timeout);
}
```

运行代码，可以看到 Connect 方法抛出 TimeoutException 错误

核心原因是在 .NET 6 通过 GetFileAttributesW 去判断管道是否存在，然而根据[堆栈网](https://stackoverflow.com/questions/28769237/calling-getfileattributesw-removes-a-pipe) 的描述，通过 GetFileAttributesW 去判断一个非文件系统的对象，会有非预期的行为。这也就是管道连接失败的原因。那为什么 .NET Framework 没问题？因为 .NET Framework 是先调用 FindFirstFile 进行判断

因此一个解决方法是采用和 .NET Framework 一样的 FindFirstFile 方法进行判断管道是否存在，代码如下

```csharp
        private static bool IsPipeExists(string pipeName)
        {
            try
            {
                // 不要用 File.Exists 判断，内部会调用 GetFileAttributes 导致管道无法被连接

                unsafe
                {
                    // 这里是一个结构体，但是不关心内容，直接栈上分配点空间给它
                    var findFileData = stackalloc byte[604];

                    var file = FindFirstFile(@"\\.\pipe\" + pipeName, (IntPtr)findFileData);

                    const nint INVALID_HANDLE_VALUE = -1;

                    if (file != INVALID_HANDLE_VALUE)
                    {
                        FindClose(file);
                        return true;
                    }
                }
            }
            catch
            {
                
            }
            return false;
        }


        [DllImport("kernel32.dll", CharSet = CharSet.Unicode, EntryPoint = "FindFirstFileW", ExactSpelling = true)]
        private static extern SafeFileHandle FindFirstFile([In] string lpFileName, [In] IntPtr lpFindFileData);

        [DllImport("kernel32.dll", CharSet = CharSet.Unicode, EntryPoint = "FindClose", ExactSpelling = true)]
        [return: MarshalAs(UnmanagedType.Bool)]
        private static extern bool FindClose([In] IntPtr hFindFile);
```

如果不想碰不安全代码，也可以采用判断文件夹里面的文件是否存在的方法判断管道是否存在

```csharp
Directory.EnumerateFiles(@"\\.\pipe\", PipeName).Any();
```

只是文件夹的判断方法会比使用 FindFirstFile 的速度慢一点点，我测试大概是 3-5 毫秒左右

更多请看 [Detecting that NamedPipe exists - Andrii Snihyr](https://berserkerdotnet.github.io/blog/detecting-namedpipe-exists/ )

本文以上的测试代码放在[github](https://github.com/lindexi/lindexi_gd/tree/e26bdd0bced93db8edf8be18f1062e571f3d1861/HallwhallkernarbafejaNakeldibi) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/e26bdd0bced93db8edf8be18f1062e571f3d1861/HallwhallkernarbafejaNakeldibi) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin e26bdd0bced93db8edf8be18f1062e571f3d1861
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 HallwhallkernarbafejaNakeldibi 文件夹

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
