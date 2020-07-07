# C# dotnet 使用 FileStream 随机文件读写

本文说的随机文件读写的随机的反义词是顺序，这里的随机文件读写对应顺序文件读写。表示文件可以不按照顺序进行读写

<!--more-->
<!-- CreateTime:7/6/2020 10:38:13 AM -->

<!-- 发布 -->

进行文件读写的时候，基本上读是几乎不存在问题的，而写的话就稍微坑了一点，在 dotnet 里面默认没有提供 RandomAccessStream 类，这个 RandomAccessStream 类仅在 UWP 中可以使用

如果在不引用 UWP 的 WPF 里面，或者在 ASP.NET Core 以及 Xamarin 里面，也可以通过 FileStream 的 Seek 方法做到进行随机的读写

在随机读写文件的时候使用 FileStream 的 Seek 方法设置当前的文件 Stream 所在的点，此时就可以从 Stream 的这个点开始进行读写。在 Stream 的 Seek 方法会在 FileStream.Windows.cs 调用 SeekCore 方法，在 SeekCore 会调用 [Kernel32.SetFilePointerEx](https://docs.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-setfilepointerex) 的方法设置到文件的读写

此时使用 Position 属性也能完成，在 FileStream.cs 里面可以看到 Position 的 Set 方法本质也是调用 Seek 方法

```csharp
        public override long Position
        {
            get {/*忽略代码*/}
        
            set
            {
            	// 忽略代码
                Seek(value, SeekOrigin.Begin);
            }
        }
```

比较推荐使用 Seek 的方法，因此这个方法功能比较强大，可以设置相对或者从前开始等

大概的做法是如移动到某个字节处开始读写，可以使用如下代码

```csharp
        private async Task WriteFile(long fileStartPoint, byte[] data, int dataLength)
        {
            Stream.Seek(fileStartPoint, SeekOrigin.Begin);

            await Stream.WriteAsync(data, offset: 0, dataLength);
        }
```

注意这里的 WriteAsync 使用的第二个参数 `offset` 指的是第一个参数 `byte[]` 的偏移而不是写入到 Stream 的偏移。通过 Seek 的方法就能做到让文件支持进行随机读写

另外，如果想要比较大的提升随机文件读写性能，我推荐在知道文件长度的时候通过 SetLength 方法设置文件长度，这样能减少文件碎片分配

如果需要进行多线程读写，此时读可以采用创建多个 FileStream 的方法，注意设置读共享。但如果存在多线程写入，我推荐是使用一个 FileStream 然后其他多个线程委托到一个线程里面进行写入，而不是多个线程同时写入。原因是多个线程同时写入的时候冲突不好处理，加上文件写入有磁盘延迟，此时的写入特别是有长度变化的时候会写出空值

我通过 [AsyncQueue](https://github.com/dotnet-campus/AsyncWorkerCollection) 做到多个线程不断写入队列，而一个线程不断从队列取出待写入的数据，写入到文件。这样做的优势在于能做到在一个线程里面写入文件，而其他线程只是委托这个写入文件线程写入，其他线程不访问文件

这部分多线程进行文件随机写入代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/f27a3701825ce8dd0f7171d4bcfe45dabab5c7d9/FileDownloader) 欢迎小伙伴访问，代码放在 RandomFileWriter.cs 文件

更多 dotnet 底层源代码请看 [官方开源代码](https://github.com/dotnet/runtime) 本文用到的代码放在 `\src\libraries\System.Private.CoreLib\src\System\IO\FileStream.cs` 和 `\src\libraries\System.Private.CoreLib\src\System\IO\FileStream.Windows.cs` 文件

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

