# 解决 vs 出现Error MC3000 给定编码中的字符无效

在 xaml 写中文注释，发现编译失败 Error MC3000 给定编码中的字符无效

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


我的 xaml 写了一句代码

```csharp
<Grid>
<!--林德熙-->
</Grid>
```

然后 vs 告诉我，给定编码中的字符无效，让我以为是我的名字是无法编译。

我尝试删掉了我的名字，发现可以编译，注释居然可以让 vs 无法编译？我问了小伙伴，他说是编码问题，于是我就去修改文件编码，但是发现还是无法编译。

最后发现在文件最前添加`<?xml version="1.0" encoding="utf-8"?>`就好了。

下面我来告诉大家如何解决这个坑。

先检查文件编码，如果文件编码不是 utf-8 那么先修改文件编码。如果还无法编译，那么在文件最前添加 `<?xml version="1.0" encoding="utf-8"?>`

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。