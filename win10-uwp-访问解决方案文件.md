#win10 uwp 访问解决方案文件

本文讲如何访问解决方案的资源。

我们经常会把一些图片资源放在我们的解决方案，那么从这里拿出来很简单。

我在 Assets 放了图片 1.jpg 那么我要把他拿出来可以

```
<Image Source="ms-appx:///Assets/logo.png" />
```

```
var file=await  StorageFile.GetFileFromApplicationUriAsync(new Uri("ms-appx:///Assets/1.png")); 
```

那么我们访问我们解决方案加的ms-appx就是解决方案的绝对路径，如果我们相对于我们xaml的路径也是可以

如果需要访问我们应用中的本地数据local，临时文件temp等，可以使用 ms-appdate:// 后面加/ local、temp

如果我们本地有1.png 放在local

我们可以使用

```
var file=await StorageFile.GetFileFromApplicationUriAsync(new Uri("ms-appdate:///local/1.png"));
```

如果提示找不到文件，因为是右击文件，属性，选生成操作：内容。记住要内容，其他可能出现找不到。

参见：[https://msdn.microsoft.com/zh-cn/library/windows/apps/xaml/hh965322(v=win.10).aspx](https://msdn.microsoft.com/zh-cn/library/windows/apps/xaml/hh965322\(v=win.10\).aspx)

关于git http://blog.csdn.net/marktheone/article/details/52062888

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

