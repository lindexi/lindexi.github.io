
# dotnet 警惕判断文件是否存在因为检查网络资源造成超长等待

在使用 System.IO.File.Exists 方法时，绝大部分的情况下都是一个非常快捷且没有成本的，但是如果判断的文件是否存在，是从非自己完全控制的逻辑下进入的，那就需要警惕是否判断的文件路径属于一个网络资源。判断一个网络资源是否存在，是一个耗时不可确定行为，很有可能造成主线程卡顿

<!--more-->


<!-- CreateTime:2022/9/30 15:12:55 -->

<!-- 博客 -->
<!-- 发布 -->

如果是传入了一个 url 路径，此路径是采用 `//` 或者 `\\` 开头的，那可能这将会是一个网络路径，或者是一个 UNC 格式的路径。如果是前者，那确实很有可能让 System.IO.File.Exists 方法判断需要等待超时，导致了这个同步的判断文件是否存在的方法卡住当前线程。如果刚好当前的线程是主线程，那就更加不好玩了

因此，在判断一个非自己完全控制的传入参数，判断此参数表示的文件是否存在时，那谨慎在主线程调用

详细的关于文件的路径表示格式，请参阅：

[File path formats on Windows systems Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/standard/io/file-path-formats )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。