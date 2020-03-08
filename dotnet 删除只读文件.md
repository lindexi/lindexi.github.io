# dotnet 删除只读文件

如果直接通过 File.Delete 删除只读文件会出现没有权限提示，可以先设置文件不是只读然后删除文件

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


```csharp
            try
            {
                File.Delete(file);
            }
            catch (UnauthorizedAccessException)
            {
                File.SetAttributes(file, FileAttributes.Normal);
                File.Delete(file);
            }
```

上面的代码是先尝试删除文件，删除失败再设置文件不是只读，然后尝试删除文件

为什么需要先尝试删除，原因是如果要删除一个文件之前还尝试去修改他，那么性能不好。

在 Try 里面如果没有出现异常，那么进入 Try 的代码和没有进入 Try 一样的速度

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
