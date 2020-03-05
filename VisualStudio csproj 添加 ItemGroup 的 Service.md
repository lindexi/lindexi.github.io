# VisualStudio csproj 添加 ItemGroup 的 Service 

本文告诉大家，如果发现在自己的工程文件出现了一个新的 ItemGroup 里面是 Service 加 GUID 是在做什么。

<!--more-->
<!-- CreateTime:2018/3/7 8:54:04 -->

<!-- csdn -->
<!-- 标签：VisualStudio -->

如果在你看到工程文件多了下面代码，里面的 GUID 可能不相同，那么就是 VisualStudio 添加了 UNIT 等单元测试

```csharp
<ItemGroup>
  <Service Include="{82A7F48D-3B50-4B1E-B82E-3ADA8210C358}" />
</ItemGroup>
```

微软：这是故意添加的，为了支持第三方的单元测试框架，如NUnit、XUnit，VisualStudio 2012 会在工程打开的时候加载测试项目，不管这时用户有没有单元测试框架，所以 VisualStudio 2012 的启动速度就会很慢。在VisualStudio 2013，微软改变了加载的算法，只有用户存在最少一个单元测试框架才会加载测试项目。但是这样就很难知道哪个项目就是单元测试项目，于是使用两个方法去定义测试项目，其中一个就是添加`<Service/>`和GUID。另一个方法是使用单元测试项目模板创建使用项目类型的GUID判断是否单元测试。

参见：https://stackoverflow.com/q/18614342/6116637

[Automatic Project Check Outs After Installing Visual Studio SDK](https://www.richard-banks.org/2007/12/automatic-project-check-outs-after.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。