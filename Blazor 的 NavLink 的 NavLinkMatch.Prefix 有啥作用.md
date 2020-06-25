# Blazor 的 NavLink 的 NavLinkMatch.Prefix 有啥作用

默认的 NavLink 的 Match 的默认值就是 NavLinkMatch.Prefix 表示只要当前的链接的路由的前部分和 href 的相同，那么将匹配上，修改样式为高亮

<!--more-->
<!-- 发布 -->

本文记于 2020.06.25 也许后续也有点变更

在 NavLink 的取值上有两个可选的值，一个是 NavLinkMatch.All 另一个是 NavLinkMatch.Prefix 按照官方的定义如下

- NavLinkMatch.All：NavLink 在与当前整个 URL 匹配的情况下处于活动状态。
- NavLinkMatch.Prefix（默认）：NavLink 在与当前 URL 的任何前缀匹配的情况下处于活动状态。

官方这两句是什么意思，可以看到在 NavLink 有一个 href 属性，如下面代码

```xml
<NavLink class="nav-link" href="counter">
    <span class="oi oi-plus" aria-hidden="true"></span> Counter
</NavLink>
```

此时的 NavLink 没有添加 Match 的值，因此是默认 NavLinkMatch.Prefix 表示只要前缀是 counter 的就激活这一项

因此访问链接如 `http://127.0.0.1:5000/counter/123` 或 `http://127.0.0.1:5000/counter` 处于都满足前缀是 coutner 因此激活这一项

但如 `http://127.0.0.1:5000/counter123` 链接，此时是 `counter123` 不满足此前缀哦，不会激活这一项

而 NavLinkMatch.All 要求是完全匹配，如将上面代码修改如下

```xml
<NavLink class="nav-link" href="counter" Match="NavLinkMatch.All">
    <span class="oi oi-plus" aria-hidden="true"></span> Counter
</NavLink>
```

此时只有访问 `http://127.0.0.1:5000/counter` 才会激活，访问 `http://127.0.0.1:5000/counter/123` 不会激活

默认创建的 Blazor 的 WebAssembly 项目可以在 `Shared\NavMenu.razor` 找到如上代码，默认只有 `href=""` 被设置为 `NavLinkMatch.All` 其他使用默认值

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/3cf2cdb488013b9022b23dde409e2cf23e393f20/HobilearnargurcalJudowokear) 欢迎小伙伴访问

官方文档请看 [ASP.NET Core Blazor 路由](https://docs.microsoft.com/zh-cn/aspnet/core/blazor/fundamentals/routing?view=aspnetcore-3.1)

特别推荐两个很好用的 Blazor 的 UI 库

- [wzxinchen/Blazui: Element的blazor版本，用 .NET 写前端的 UI 框架，无JS，无TS，非 Silverlight，非 WebForm，开箱即用](https://github.com/wzxinchen/Blazui)

- [ant-design-blazor/ant-design-blazor: 🌈A set of enterprise-class UI components based on Ant Design and Blazor WebAssembly.](https://github.com/ant-design-blazor/ant-design-blazor/)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
