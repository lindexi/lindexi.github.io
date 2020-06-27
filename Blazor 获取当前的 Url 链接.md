# Blazor 获取当前的 Url 链接

在 Blazor 获取当前页面所在的 URL 链接可以通过 NavigationManager 类辅助获取，也可以通过此方法获取当前域名等信息

<!--more-->
<!-- 发布 -->

首先在页面添加依赖注入，如下面代码

```csharp
@inject NavigationManager NavigationManager
```

此时就注入了 `NavigationManager` 属性，获取当前页面所在链接的方法或域名可以采用 Uri 或 BaseUri 两个属性

```xml
<p>
	NavigationManager.Uri = @NavigationManager.Uri
</p>

<p>
	NavigationManager.BaseUri = @NavigationManager.BaseUri
</p>
```

详细请看 [ASP.NET Core Blazor 路由](https://docs.microsoft.com/zh-cn/aspnet/core/blazor/fundamentals/routing?view=aspnetcore-3.1)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
