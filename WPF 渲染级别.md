# WPF 渲染级别

很少人会知道 WPF 也可以知道当前的显卡能支持的渲染级别。 根据显卡的不同，包括显存、纹理等的支持是否打到要求，指定渲染级别。

<!--more-->
<!-- CreateTime:2018/7/24 18:46:27 -->

<!-- csdn -->
<!-- 标签：WPF,渲染 -->

使用 System.Windows.Media.RenderCapability 可以拿到 WPF 的渲染级别

```csharp
                var renderingTier = System.Windows.Media.RenderCapability.Tier >> 16;

```

因为直接拿到 RenderCapability.Tier 是不能用的，参见[WPF 渲染级别 (Tier)](http://www.cnblogs.com/andrew-blog/p/WPF_Tier.html )

通过 renderingTier 的值就可以判断当前显卡渲染级别

 - 0 没有显卡加速功能

 - 1 只有部分有显卡加速

 - 2 所有功能由显卡加速

可以通过这个判断方式决定是否加载某些动画。

但是不能通过这个方式判断当前是否存在独立显卡，因为很多时候有集显都是返回 2 ，大家都知道，集显的性能一般都不是很好。

参见：[Graphics Rendering Tiers](https://docs.microsoft.com/en-us/dotnet/framework/wpf/advanced/graphics-rendering-tiers )

[RenderCapability.Tier Property (System.Windows.Media)](https://docs.microsoft.com/en-us/dotnet/api/system.windows.media.rendercapability.tier?view=netframework-4.7.2#System_Windows_Media_RenderCapability_Tier )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
