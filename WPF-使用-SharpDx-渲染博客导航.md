
# WPF 使用 SharpDx 渲染博客导航

我写了很多在 WPF 使用 SharpDx 渲染的博客，但是有小伙伴吐槽说这些博客没有一个好看的顺序，我将所有的 SharpDx 的博客按照顺序放在下面

<!--more-->


<!-- CreateTime:2019/10/23 21:10:13 -->

使用 SharpDx 必然会涉及很多底层的渲染知识，和 DirectX 相关的知识将会有很多，一旦开始入手就是复杂的大项目，将很好被劝退。我希望大家在开始前，先看如何在 C# 用一个控制台创建一个 SharpDx 程序，以下是系列博客

- [C# 从零开始写 SharpDx 应用 控制台创建 Sharpdx 窗口](https://blog.lindexi.com/post/C-%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E5%86%99-SharpDx-%E5%BA%94%E7%94%A8-%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%88%9B%E5%BB%BA-Sharpdx-%E7%AA%97%E5%8F%A3.html )

<!-- - [C# 从零开始写 SharpDx 应用 初始化dx修改颜色](https://blog.csdn.net/lindexi_gd/article/details/82114907 ) -->
- [C# 从零开始写 SharpDx 应用 初始化dx修改颜色](https://blog.lindexi.com/post/C-%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E5%86%99-SharpDx-%E5%BA%94%E7%94%A8-%E5%88%9D%E5%A7%8B%E5%8C%96dx%E4%BF%AE%E6%94%B9%E9%A2%9C%E8%89%B2.html )

- [C# 从零开始写 SharpDx 应用 画三角](https://blog.lindexi.com/post/C-%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E5%86%99-SharpDx-%E5%BA%94%E7%94%A8-%E7%94%BB%E4%B8%89%E8%A7%92.html )

- [C# 从零开始写 SharpDx 应用 绘制基础图形](https://blog.lindexi.com/post/C-%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E5%86%99-SharpDx-%E5%BA%94%E7%94%A8-%E7%BB%98%E5%88%B6%E5%9F%BA%E7%A1%80%E5%9B%BE%E5%BD%A2.html )

一些 SharpDx 杂项知识如下：

- [C# 从零开始写 SharpDx 应用 聊聊功能等级](https://blog.lindexi.com/post/C-%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E5%86%99-SharpDx-%E5%BA%94%E7%94%A8-%E8%81%8A%E8%81%8A%E5%8A%9F%E8%83%BD%E7%AD%89%E7%BA%A7.html )
- [C# 从零开始写 SharpDx 应用 从 D3DDevice 获取 DXGIDevice 方法](https://blog.lindexi.com/post/C-%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E5%86%99-SharpDx-%E5%BA%94%E7%94%A8-%E4%BB%8E-D3DDevice-%E8%8E%B7%E5%8F%96-DXGIDevice-%E6%96%B9%E6%B3%95.html )

其实 SharpDx 只是一个封装，在没有封装的时候可以直接使用 Direct2D1 在 WPF 画出界面

- [WPF 使用 Direct2D1 画图入门](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Direct2D1-%E7%94%BB%E5%9B%BE%E5%85%A5%E9%97%A8.html )
- [WPF 使用 Direct2D1 画图 绘制基本图形](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Direct2D1-%E7%94%BB%E5%9B%BE-%E7%BB%98%E5%88%B6%E5%9F%BA%E6%9C%AC%E5%9B%BE%E5%BD%A2.html)

但是使用Direct2D1写起来很难写，需要有一些封装，下面告诉大家如何在 WPF 使用 SharpDx 做界面

- [WPF 使用 SharpDX](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-SharpDX.html )
- [WPF 使用 SharpDX 在 D3DImage 显示](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-SharpDX-%E5%9C%A8-D3DImage-%E6%98%BE%E7%A4%BA.html )
- [WPF 使用封装的 SharpDx 控件](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8%E5%B0%81%E8%A3%85%E7%9A%84-SharpDx-%E6%8E%A7%E4%BB%B6.html )
- [WPF 使用 SharpDx 异步渲染](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-SharpDx-%E5%BC%82%E6%AD%A5%E6%B8%B2%E6%9F%93.html )
- [SharpDx D3D9 的颜色 Format 和 WPF 的 PixelFormat 转换方法](https://blog.lindexi.com/post/SharpDx-D3D9-%E7%9A%84%E9%A2%9C%E8%89%B2-Format-%E5%92%8C-WPF-%E7%9A%84-PixelFormat-%E8%BD%AC%E6%8D%A2%E6%96%B9%E6%B3%95.html )

一些应用方法如下

- [SharpDx 进入全屏模式](https://blog.lindexi.com/post/SharpDx-%E8%BF%9B%E5%85%A5%E5%85%A8%E5%B1%8F%E6%A8%A1%E5%BC%8F.html )
- [C# 从零开始写 SharpDx 应用 禁止 Alt + Enter 进入全屏](https://blog.lindexi.com/post/C-%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E5%86%99-SharpDx-%E5%BA%94%E7%94%A8-%E7%A6%81%E6%AD%A2-Alt-+-Enter-%E8%BF%9B%E5%85%A5%E5%85%A8%E5%B1%8F.html )

值得一说的是 SharpDx 当前官方不维护了，可以选择的代替请看 [SharpDx 的代替项目](https://blog.lindexi.com/post/SharpDx-%E7%9A%84%E4%BB%A3%E6%9B%BF%E9%A1%B9%E7%9B%AE.html )



其中一个可代替 SharpDx 的是 Vortice 库，以下是使用 Vortice 库的博客

- [DirectX 使用 Vortice 从零开始控制台创建 Direct2D1 窗口修改颜色](https://blog.lindexi.com/post/DirectX-%E4%BD%BF%E7%94%A8-Vortice-%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%88%9B%E5%BB%BA-Direct2D1-%E7%AA%97%E5%8F%A3%E4%BF%AE%E6%94%B9%E9%A2%9C%E8%89%B2.html )

- [dotnet DirectX 通过 Vortice 控制台使用 ID2D1DeviceContext 绘制画面](https://blog.lindexi.com/post/dotnet-DirectX-%E9%80%9A%E8%BF%87-Vortice-%E6%8E%A7%E5%88%B6%E5%8F%B0%E4%BD%BF%E7%94%A8-ID2D1DeviceContext-%E7%BB%98%E5%88%B6%E7%94%BB%E9%9D%A2.html )

- [dotnet C# 使用 Vortice 支持 Direct2D1 离屏渲染](https://blog.lindexi.com/post/dotnet-C-%E4%BD%BF%E7%94%A8-Vortice-%E6%94%AF%E6%8C%81-Direct2D1-%E7%A6%BB%E5%B1%8F%E6%B8%B2%E6%9F%93.html )

- [dotnet C# 使用 Vortice 创建 Direct2D1 的 ID2D1SolidColorBrush 纯色画刷](https://blog.lindexi.com/post/dotnet-C-%E4%BD%BF%E7%94%A8-Vortice-%E5%88%9B%E5%BB%BA-Direct2D1-%E7%9A%84-ID2D1SolidColorBrush-%E7%BA%AF%E8%89%B2%E7%94%BB%E5%88%B7.html )

- [WPF 对接 Vortice 调用 D2D 使用 IWICBitmap 离屏渲染](https://blog.lindexi.com/post/WPF-%E5%AF%B9%E6%8E%A5-Vortice-%E8%B0%83%E7%94%A8-D2D-%E4%BD%BF%E7%94%A8-IWICBitmap-%E7%A6%BB%E5%B1%8F%E6%B8%B2%E6%9F%93.html ) [博客园](https://www.cnblogs.com/lindexi/p/16774416.html )

- [WPF 对接 Vortice 调用 WIC 加载图片](https://blog.lindexi.com/post/WPF-%E5%AF%B9%E6%8E%A5-Vortice-%E8%B0%83%E7%94%A8-WIC-%E5%8A%A0%E8%BD%BD%E5%9B%BE%E7%89%87.html )

- [WPF 对接 Vortice 在 Direct2D 绘制从 WIC 加载的图片](https://blog.lindexi.com/post/WPF-%E5%AF%B9%E6%8E%A5-Vortice-%E5%9C%A8-Direct2D-%E7%BB%98%E5%88%B6%E4%BB%8E-WIC-%E5%8A%A0%E8%BD%BD%E7%9A%84%E5%9B%BE%E7%89%87.html )

- [dotnet C# 通过 Vortice 使用 Direct2D 特效入门](https://blog.lindexi.com/post/dotnet-C-%E9%80%9A%E8%BF%87-Vortice-%E4%BD%BF%E7%94%A8-Direct2D-%E7%89%B9%E6%95%88%E5%85%A5%E9%97%A8.html )

- [dotnet C# 通过 Vortice 使用 Direct2D 的 ID2D1CommandList 入门](https://blog.lindexi.com/post/dotnet-C-%E9%80%9A%E8%BF%87-Vortice-%E4%BD%BF%E7%94%A8-Direct2D-%E7%9A%84-ID2D1CommandList-%E5%85%A5%E9%97%A8.html )

- [dotnet C# 通过 Vortice 将 ID2D1CommandList 作为特效的输入源](https://blog.lindexi.com/post/dotnet-C-%E9%80%9A%E8%BF%87-Vortice-%E5%B0%86-ID2D1CommandList-%E4%BD%9C%E4%B8%BA%E7%89%B9%E6%95%88%E7%9A%84%E8%BE%93%E5%85%A5%E6%BA%90.html )

- [使用 Silk.NET 调用 DirectWrite 获取字体 Font Metrics 信息](https://blog.lindexi.com/post/%E4%BD%BF%E7%94%A8-Silk.NET-%E8%B0%83%E7%94%A8-DirectWrite-%E8%8E%B7%E5%8F%96%E5%AD%97%E4%BD%93-Font-Metrics-%E4%BF%A1%E6%81%AF.html )
<!-- [使用 Silk.NET 调用 DirectWrite 获取字体 Font Metrics 信息 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/19001918 ) -->

在 4k 下的性能测试请看 [WPF 模拟 WPFMediaKit 的 D3D 配置用来测试4k性能](https://blog.lindexi.com/post/WPF-%E6%A8%A1%E6%8B%9F-WPFMediaKit-%E7%9A%84-D3D-%E9%85%8D%E7%BD%AE%E7%94%A8%E6%9D%A5%E6%B5%8B%E8%AF%954k%E6%80%A7%E8%83%BD.html )

如果有部分逻辑是通过其实平台渲染的，可以通过 SharedHandle 在不同的 Direct3D 设备之间共享资源，详细请看下面博客

- [使用 Direct3D11 的 OpenSharedResource 方法渲染来自其他进程/设备的共享资源（SharedHandle） - walterlv](https://blog.walterlv.com/post/direct3d11-open-shared-resource.html#sharedhandle )

在遇到界面不渲染时，可以保存渲染图片到本地查看，请看 [将 Direct3D11 在 GPU 中的纹理（Texture2D）导出到内存（Map）或导出成图片文件 - walterlv](https://blog.walterlv.com/post/map-directx-surface-to-bitmap.html )

更多请看 [SharpDX 系列](https://blog.lindexi.com/post/sharpdx.html )

我在 CSDN 创建 [WPF 底层渲染](https://blog.csdn.net/lindexi_gd/column/info/24324 ) 系列博客，欢迎大家访问

如果想了解[渲染相关](https://blog.lindexi.com/post/%E6%B8%B2%E6%9F%93 )如 Win2d 和 WPF 渲染原理等请看 [渲染相关](https://blog.lindexi.com/post/%E6%B8%B2%E6%9F%93 )

以下是我收藏的一些博客

- [DirectX11 With Windows SDK--00 目录 - X_Jun - 博客园](https://www.cnblogs.com/X-Jun/p/9028764.html?tdsourcetag=s_pctim_aiomsg )
- [现代图形API对比-CSDN博客](https://yangshuohao.blog.csdn.net/article/details/115834816 )

有任何建议和吐槽都欢迎通过邮件联系我，或加入[dotnet 职业技术学院](https://t.me/dotnet_campus) 交流

注意[dotnet 职业技术学院](https://t.me/dotnet_campus) 是一个电报群，加群需要一些技术




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。