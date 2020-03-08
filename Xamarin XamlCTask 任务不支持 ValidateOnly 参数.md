# Xamarin XamlCTask 任务不支持 ValidateOnly 参数

使用 Xamarin 项目，添加一个额外的库项目，被 Xamarin.Form 引用，构建时提示“XamlCTask”任务不支持“ValidateOnly”参数。请确认该参数存在于此任务中，并且是可设置的公共实例属性

<!--more-->
<!-- CreateTime:2020/3/2 8:23:13 -->

<!-- 发布 -->

英文的提示如下

```csharp
MSB4064: The "ValidateOnly" parameter is not supported by the "XamlCTask" task
```

这个算 Xamarin 4.3 的坑，解决方法是在 NuGet 升级到 4.5 就可以了

[[Bug] Mixed Xamarin.Forms versions causes error MSB4064: The "ValidateOnly" parameter is not supported by the "XamlCTask" task · Issue #8209 · xamarin/Xamarin.Forms](https://github.com/xamarin/Xamarin.Forms/issues/8209 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
