# WPF 程序生成类库错误

把WPF程序输出改为类库，报错


```csharp
    库项目文件无法指定 ApplicationDefinition 元素 项目文件包含无效的属性值
```
<!--more-->
<!-- CreateTime:2018/8/10 19:16:53 -->

<!-- csdn -->

出现这个错误是因为 app.xaml 的生成是 ApplicationDefinition ，所以可以右击项目的  app.xaml 文件，在属性生成选择 page

![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F201753104937.jpg)

参见：http://blog.csdn.net/hefeng_aspnet/article/details/17245205

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  