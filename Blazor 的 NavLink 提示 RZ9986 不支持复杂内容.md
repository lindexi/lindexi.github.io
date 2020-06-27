# Blazor 的 NavLink 提示 RZ9986 不支持复杂内容

在使用 Blazor 做动态的跳转的时候，如果在 NavLink 的 href 添加了包含 C# 代码，那么将会提示 RZ9986 组件的属性不支持复杂的内容，如混合 C# 代码和标记等。解决方法是通过按钮加上事件代替

<!--more-->
<!-- 发布 -->

在使用如下面代码在循环里面写跳转逻辑

```
@for (int i = 1; i < PageCount + 1; i++)
{
        <li>
            <NavLink href="/blog/page/@(i)">@(i)</NavLink>
        </li>
}
```

此时尝试运行代码将会提示如下错误

```
严重性	代码	说明	项目	文件	行	禁止显示状态
错误	RZ9986	Component attributes do not support complex content (mixed C# and markup). Attribute: 'href', text: '/blog/page/i'	StaticBlog	g:\lindexi\Blog\StaticBlog\RAZORGENERATE	1	
```

如 `RZ9986	Component attributes do not support complex content (mixed C# and markup). Attribute: 'href', text:` 所说，这里的坑就是 `NavLink` 的 href 只支持静态的字符串，不支持拼接，因此如[官方文档](https://docs.microsoft.com/zh-CN/aspnet/core/blazor/fundamentals/routing?view=aspnetcore-3.1) 所说，可以使用 [NavigationManager](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.components.navigationmanager) 配合按钮解决此问题

在添加按钮之前，需要知道当前是第几次循环进入，此时需要用到再定义一个变量。如果了解闭包问题的小伙伴应该很好理解，如果不了解的话，就请听我吹一下水。咱需要再写一个变量去获取当前是 `for` 循环的第几次循环进来，用于传入按钮点击时的事件，如[官方文档](https://docs.microsoft.com/zh-CN/aspnet/core/blazor/components/event-handling?view=aspnetcore-3.1)所说，假设咱在事件的表达式里面使用了循环迭代的变量，因此变量只有一个，因此这个变量的值会被变更，因此做不到让每次循环创建的按钮都知道自己是第几个被创建的

```csharp
@for (int i = 1; i < PageCount + 1; i++)
{
        // 必须放在标签之前，如下面代码放在标签下面是不对的
        var currentPage = i;
        <li>
            @* 下面这一行代码将会显示为 HTML 内容 *@
            @*var currentPage = i;*@
            <button @onclick="@(e => GotoPage(e, currentPage))">@i</button>
            
            @*<NavLink href="/blog/page/@(i)">@(i)</NavLink>*@
        </li>
}

@code
{
	private void GotoPage(MouseEventArgs e, int currentPage)
    {
        NavigationManager.NavigateTo($"/blog/page/{currentPage}");
    }
}
```

上面代码有两个注意的地方，第一个地方就是需要创建 `currentPage` 这个变量，这个变量能固定当前循环进入的值。此外需要将 `currentPage` 的定义放在标签之前，如上面代码

这样玩就能做到跳转了，对比使用 `a` 的跳转标签的优势在于用此方法依然是单页应用，而不是重新进入一个新的页面。这句话不理解的小伙伴，请自己修改为 `a` 跳转对比试试就知道了

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
