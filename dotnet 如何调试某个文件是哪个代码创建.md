# dotnet 如何调试某个文件是哪个代码创建

我发现了自己的软件，会在桌面创建一个 1.txt 文件，但是我不知道是哪个代码创建的，那么如何进行快速的调试找到是哪个代码创建的

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


最简单的方法是使用 VisualStudio 全局搜 `1.txt` 看是否存在，但是这个方法存在两个问题，一个问题是可能这个文件名是拼出来的，如下面代码

```csharp
            for (int i = 1; i < 2; i++)
            {
                var file = $"{i}.txt";
            }
```

这样就无法通过搜 `1.txt` 找到这个代码

之外可能这个文件的写入是在库里面做的，如引用了 `林德熙是逗比.dll` 在这个库里面写入了文件

那么有什么有效的方法？

可以通过这样的方法，先删除这个文件，然后创建一个文件夹，将这个文件夹命名为和这个文件相同的名。如删除 `1.txt` 文件，然后创建 `1.txt` 文件夹，然后获取全局异常，现在就可以发现有文件读写异常的代码应该就是写入 `1.txt` 文件的代码

除了创建文件夹之外，还可以写一个 `FileStream` 占用这个 `1.txt` 文件，找到一个文件由另一个进程占用的异常就可以找到代码在哪

不过本文提供的这些方法都做不到解决在 C++ 等库里面写文件的问题

特别感谢[walterlv](https://blog.walterlv.com/ )这位写出了 `a[b[c[e[g[a1[a2[a3]]]]]]].Foo` 的小伙伴提供的方法

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
