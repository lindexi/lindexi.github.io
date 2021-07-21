
# dotnet core 和 .NET 5 不支持 Prefer32Bit 首选 32 位的功能

我尝试在 dotnet core 和 dotnet 5 的应用上，右击项目属性，在生成界面勾选首选 32 位的功能，然而在 x64 下没有生成 PE32+ 的应用

<!--more-->


<!-- 发布 -->

这是 .NET Core 以及以上版本不支持的功能，勾选了无效，在 csproj 上添加如下代码也无效

```xml
<PropertyGroup Condition="'$(Platform)'=='AnyCPU'">
  <PlatformTarget>AnyCPU</PlatformTarget>
  <Prefer32Bit>true</Prefer32Bit>
</PropertyGroup>
```

没有什么非常特别的理由，就是不支持而已

更多细节请看 

[.NET Assembly “Cross-Bitness” Loading – Mihai-Albert.com](https://mihai-albert.com/2019/03/10/net-assembly-cross-bitness-loading/ )

["Prefer32Bit" property remains true when PlatformTarget is x64, but checkbox is empty and disabled. · Issue #5933 · dotnet/project-system](https://github.com/dotnet/project-system/issues/5933 )

[Startup Sequence of a .NET Core App – Mihai-Albert.com](https://mihai-albert.com/2020/03/08/startup-sequence-of-a-dotnet-core-app/#bitness )

[c# - 'Prefer 32-bit' in Visual Studio Generates 64-bit Code Under Console App (.NET Core) - Stack Overflow](https://stackoverflow.com/questions/60324529/prefer-32-bit-in-visual-studio-generates-64-bit-code-under-console-app-net-c )

[Remove "Prefer 32bit" Property from .NET Core Projects · Issue #5074 · dotnet/project-system](https://github.com/dotnet/project-system/issues/5074 )

[dotnet 5 wpf did not respect "Prefer32Bit" setting · Issue #4872 · dotnet/wpf](https://github.com/dotnet/wpf/issues/4872 )

[WPF 编译为 AnyCPU 和 x86 有什么区别](https://blog.lindexi.com/post/WPF-%E7%BC%96%E8%AF%91%E4%B8%BA-AnyCPU-%E5%92%8C-x86-%E6%9C%89%E4%BB%80%E4%B9%88%E5%8C%BA%E5%88%AB.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。