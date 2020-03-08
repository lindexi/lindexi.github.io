# 恢复 U 盘隐藏文件夹

在U盘放在打印的设备打印一个文件之后，发现U盘的文件夹都找不到了

<!--more-->
<!-- CreateTime:2018/12/14 19:24:56 -->

<!-- csdn -->

解决方法是在命令行输入下面命令，假设 U 盘的盘符是 H 盘

```csharp
attrib -h -r -s /s /d H:\*.*
```

替换上面的 `H:\*.*` 为你的盘符 `盘符:\*.*` 运行需要等待一会

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
