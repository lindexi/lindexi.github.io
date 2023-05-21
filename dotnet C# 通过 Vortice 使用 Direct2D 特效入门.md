# dotnet C# 通过 Vortice 使用 Direct2D 特效入门

本文将告诉大家如何通过 Vortice 使用 D2D 的特效

<!--more-->
<!-- 标签：C#,D2D,DirectX,Vortice,Direct2D, -->
<!-- 博客 -->
<!-- 发布 -->

本文属于 DirectX 系列博客，更多 DirectX 和 D2D 以及 Vortice 库的博客，请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

上一篇： [DirectX 使用 Vortice 从零开始控制台创建 Direct2D1 窗口修改颜色](https://blog.lindexi.com/post/DirectX-%E4%BD%BF%E7%94%A8-Vortice-%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%88%9B%E5%BB%BA-Direct2D1-%E7%AA%97%E5%8F%A3%E4%BF%AE%E6%94%B9%E9%A2%9C%E8%89%B2.html )

在上一篇博客里面，咱创建了一个 Win32 空窗口，接着给他挂上了 DirectX 交换链。使用以下代码从交换链里面拿到了 DXGI 平面，拿到的的 DXGI 平面即可被绘制 2D 内容在上面，从而将内容绘制输出到窗口上



<!-- 特效： 


 -->


本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/615c235ce34b8c38abe1e99e65a5e34ddc9addb0/VorticeD2DEffect1) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/615c235ce34b8c38abe1e99e65a5e34ddc9addb0/VorticeD2DEffect1) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 615c235ce34b8c38abe1e99e65a5e34ddc9addb0
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 615c235ce34b8c38abe1e99e65a5e34ddc9addb0
```

获取代码之后，进入 VorticeD2DEffect1 文件夹

有伙伴好奇为什么我最近写的是通过 Vortice 调用 DirectX 的博客，而不是通过 SharpDx 或 Silk.NET 调用 DirectX 的博客。这不是我收了 Vortice 的钱或者是和 Vortice 有什么 py 交易哈。其原因是 SharpDx 不维护了，作为 SharpDx 的接任者 Vortice 的行为和 API 都会靠近 SharpDx 许多，我编写起来比较顺手。而 Silk.NET 是对 DirectX 的底层封装，由于是直接底层封装，导致使用 Silk.NET 比较繁琐。尽管使用 Silk.NET 的性能从理论分析上能够比 Vortice 和 SharpDx 更好，但从定量上说，其实好不了多少。我所遇到的几乎所有性能问题，基本都卡在渲染上，而不是调用上，调用上的损耗基本可以忽略。那 Silk.NET 是不是就无用武之地？其实不然，在一些情况下，机器的性能不够业务的需求情况下，能省多少就应该省多少。而且在熟悉整个过程之后，即使将 Vortice 换成 Silk.NET 也只不过是一个体力活而已，将各个 API 进行替换即可。而且有趣的是，可以混合着 Vortice 和 Silk.NET 一起用，只有某些模块才使用 Silk.NET 编写

我创建了专门聊 Vortice 的 QQ 群： 622808968 欢迎加入交流技术