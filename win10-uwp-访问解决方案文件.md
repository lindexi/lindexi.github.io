# win10 uwp 访问解决方案文件

本文讲如何访问解决方案的资源。
<!--more-->
<!-- CreateTime:2019/10/31 9:24:18 -->


<div id="toc"></div>
<!-- 标签：uwp，uwp文件 -->

我们经常会把一些图片资源放在我们的解决方案，那么从这里拿出来很简单。

我在 Assets 放了图片 1.jpg 那么我要把他拿出来可以

```xml
<Image Source="ms-appx:///Assets/logo.png" />
```

```csharp
var file = await StorageFile.GetFileFromApplicationUriAsync(new Uri("ms-appx:///Assets/1.png")); 
```

那么我们访问我们解决方案加的 ms-appx 就是解决方案的绝对路径，如果我们相对于我们xaml的路径也是可以

如果需要访问我们应用中的本地数据 local ，临时文件 temp 等，可以使用 ms-appdate:// 后面加/ local、temp

如果我们本地有 1.png 放在 local

我们可以使用

```csharp
var file=await StorageFile.GetFileFromApplicationUriAsync(new Uri("ms-appdate:///local/1.png"));
```

如果提示找不到文件，但是自己确定位置是对的，那么一个可能是，你创建了生成不对。
试试右击文件，属性，选生成操作：内容。记住要内容，其他可能出现找不到。

参见：[https://msdn.microsoft.com/zh-cn/library/windows/apps/xaml/hh965322(v=win.10).aspx](https://msdn.microsoft.com/zh-cn/library/windows/apps/xaml/hh965322\(v=win.10\).aspx)

关于git http://blog.csdn.net/marktheone/article/details/52062888

## WPF 访问解决方案文件

WPF 使用 Application.GetResourceStream 获得，注意 URL 和UWP不同，WPF使用 `pack://application:,,,/` 的方法 参见 https://msdn.microsoft.com/en-us/library/aa970069%28v=vs.110%29.aspx?f=255&MSPPError=-2147217396

具体请看 [WPF 使用 VisualStudio 2017 项目文件](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-VisualStudio-2017-%E9%A1%B9%E7%9B%AE%E6%96%87%E4%BB%B6.html )

## C# 访问解决方案文件

如果是命令行，那么可以使用 Resource 放文件，然后读取

在项目右击新建资源文件，注意这个文件的后缀是 resw 格式，创建之后，点击添加文件，选取项目里面的现有文件。此时将会自动生成代码，包括添加的文件的二进制数组属性

读取到的文件是 byte[] 数组 ，如果需要转 stream 请看下面代码，其中资源文件名称为 resource ，资源文件里的文件是 Res 属性，资源文件是没有后缀，所以打开不能通过后缀判断


```csharp
    new MemoryStream(resource.Res)
```

也就是将 byte 数组转换为 MemoryStream 从而转为 Stream 作为参数

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。



