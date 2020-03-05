# dotnet core 使用 GBK 编码

本文告诉大家如何在 .NET Core 中使用 GBK 编码

<!--more-->
<!-- CreateTime:2019/11/29 8:32:11 -->


默认的 .NET Core 框架不包含 GBK 编码，不包含除了代码页为 28591 和 Unicode(utf-8,utf-16) 之外的其他编码，需要安装 System.Text.Encoding.CodePages 才能使用

先通过 NuGet 安装 System.Text.Encoding.CodePages 库

然后在使用之前调用下面方法，注意下面的方法只需要在程序集内调用一次

```csharp
//使用CodePagesEncodingProvider去注册扩展编码。
Encoding.RegisterProvider(CodePagesEncodingProvider.Instance);
```

这个代码是做注册，可以在启动的时候调用

获取 GBK 可以使用下面方法

```csharp
//注册GBK编码
Encoding encodingGbk = Encoding.GetEncoding("GBK");
```

[C＃ 判断文件编码](https://blog.lindexi.com/post/C-%E5%88%A4%E6%96%AD%E6%96%87%E4%BB%B6%E7%BC%96%E7%A0%81.html)

[dotnet core 使用 GBK 编码](https://blog.lindexi.com/post/dotnet-core-%E4%BD%BF%E7%94%A8-GBK-%E7%BC%96%E7%A0%81.html )

[VisualStudio 编码规范工具 2.6 修改当前文件编码](https://blog.lindexi.com/post/visualstudio-%E7%BC%96%E7%A0%81%E8%A7%84%E8%8C%83%E5%B7%A5%E5%85%B7-2.6-%E4%BF%AE%E6%94%B9%E5%BD%93%E5%89%8D%E6%96%87%E4%BB%B6%E7%BC%96%E7%A0%81 )

[win10 uwp 读取文本GBK错误](https://blog.lindexi.com/post/win10-uwp-%E8%AF%BB%E5%8F%96%E6%96%87%E6%9C%ACGBK%E9%94%99%E8%AF%AF.html )

![](https://i.loli.net/2019/08/29/H7N41mpAM2T6QBg.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。        
