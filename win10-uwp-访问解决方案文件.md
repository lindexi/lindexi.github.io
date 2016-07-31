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

参见：[https:\/\/msdn.microsoft.com\/zh-cn\/library\/windows\/apps\/xaml\/hh965322\(v=win.10\).aspx](https://msdn.microsoft.com/zh-cn/library/windows/apps/xaml/hh965322(v=win.10).aspx\)

关于git http:\/\/blog.csdn.net\/marktheone\/article\/details\/52062888

