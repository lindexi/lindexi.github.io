
# C# 命令行如何静默调用 del 删除文件

如果在 C# 命令行调用 del 删除文件，很多时候会提示是否需要删除，本文告诉大家如何调用命令行的时候静默删除

<!--more-->


<!-- csdn -->

在[C# 命令行](https://gist.github.com/lindexi/f2868a0d02f2197fbb62368514ed6f99) 调用 del 删除文件的时候，会提示是否删除，通过在命令行加上 `\Q` 可以静默删除

```csharp
del /F /Q 文件
```

这里的 `/F` 是删除只读文件

[C# 使用相同权限调用 cmd 传入命令](https://lindexi.gitee.io/post/C-%E4%BD%BF%E7%94%A8%E7%9B%B8%E5%90%8C%E6%9D%83%E9%99%90%E8%B0%83%E7%94%A8-cmd-%E4%BC%A0%E5%85%A5%E5%91%BD%E4%BB%A4.html )

[How to skip "are you sure Y/N" when deleting files in batch files - Stack Overflow](https://stackoverflow.com/questions/7160342/how-to-skip-are-you-sure-y-n-when-deleting-files-in-batch-files )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。