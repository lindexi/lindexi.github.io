# win10 uwp release 因为 Entry Point Not Found 无法启动

本文告诉大家如果在使用 release 编译时，无法启动应用，出现 Entry Point Not Found 如何让应用运行。

<!--more-->
<!-- CreateTime:2018/12/25 9:39:42 -->


```csharp
程序“[30760] xx.exe”已退出，返回值为 -1073741511 (0xc0000139) 'Entry Point Not Found'。
激活 Windows 应用商店应用“43179.1161685EE70AE_ajj8jc175maf4!App”失败，错误为“应用未启动”。
```

![](https://i.loli.net/2018/04/06/5ac6f00b7e2f2.jpg)

如果在 DEBUG 可以运行，但是在 Release 运行就退出，而且看到输出`-1073741511 (0xc0000139) 'Entry Point Not Found'` 那么请使用下面的方法

右击项目的属性，在生成页面取消使用.net 本机工具链编译。

如果你的VisualStudio是英文的，那么就右击项目属性，在 Build 页面取消 Complie with .net native tool chain

![](https://i.loli.net/2018/04/06/5ac6f01c404e2.jpg)

![](https://i.loli.net/2018/04/06/5ac6f076ab669.jpg)

如果遇到每次都需要重新部署，那么点击 x86 或 x64 的配置管理器，打开部署

![](https://i.loli.net/2018/04/06/5ac6f0655b88e.jpg)

[UWP app fails to start because of 'Entry Point Not Found' exception · Issue #267 · xamarin/Xamarin.Auth](https://github.com/xamarin/Xamarin.Auth/issues/267 )

[Deploy UWP on release: error 0xC0000139: Entry Point Not Found - Stack Overflow](https://stackoverflow.com/questions/49672036/deploy-uwp-on-release-error-0xc0000139-entry-point-not-found/49684955#49684955 )

但是如果不加上Release的本机工具那么就无法把包上传到商店，所以我还是更新了 VisualStudio 2017 15.6.5 然后重新创建一个空白工程，把图床的所有类放进去，居然重新生成就可以了。

我对比了文件的不同，也没有找到哪个地方会编译不通过。所以建议大家升级 VisualStudio ，如果还是无法使用，就重新创建一个空白项目，如果空白项目可以使用。那么继续把类放进来。

![](https://i.loli.net/2018/04/08/5ac9ff7657fb2.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
