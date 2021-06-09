
# dotnet 忽略输出文件夹的正则表达式

本文告诉大家在 dotnet 里面忽略 obj 和 x86 等输出文件夹的正则表达式内容

<!--more-->


<!-- CreateTime:2021/6/7 20:31:06 -->


<!-- 发布 -->

正则表达式如下

```csharp
        private static readonly Regex IgnoreIdentifierRegex = new Regex(@"^([Bb]in|[Oo]bj|[Dd]ebug|[Rr]elease|[Xx]86|[Xx]64|net[\.\w]*\d+)$");
```

最后的 `net[\.\w]*\d+` 表示忽略 `netcoreapp3.1` 等文件夹

欢迎加入正则群 108425797





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。