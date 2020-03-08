# C＃ 相对路径转绝对路径


<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>


如果是路径相对路径，使用 Path 转换


```csharp
  System.IO.Path.Combine(文件夹, relativePath);
```

文件夹就是相对的文件夹。

这样就可以把相对路径转绝对。


参见：http://stackoverflow.com/questions/4796254/relative-path-to-absolute-path-in-c

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  