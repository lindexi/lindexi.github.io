
# win10 uwp 轻量级 MVVM 框架入门 2.1.5.3199

一个好的框架是不需要写教程大家看到就会用，但是本金鱼没有那么好的技术，所以需要写很长的博客告诉大家如何使用我的框架。

<!--more-->


<div id="toc"></div>
<!-- csdn -->
<!-- 标签：win10,uwp,mvvm -->
<!-- 草稿 -->

在本文开始之前，希望大家是有 UWP 基础而且熟悉 C#，因为本金鱼有很多认为是大家都知道的就没有在博客说。

## 安装

首先需要从 Nuget 安装两个库

 - lindexi.uwp.Framework

 - lindexi.MVVM.Framework

第一个库是使用 UWP 的封装，因为我还有 WPF 的封装，实际上在使用，用 WPF 或 UWP 是差不多的。只要存在 UWP 和 WPF 不相同的库，我就把这写封装在不同的库。

使用 WPF 项目只需要安装 lindexi.wpf.Framework 这个库。因为 Nuget 可以找到依赖库，所以只需要安装 lindexi.wpf.Framework 就会自动安装 lindexi.MVVM.Framework 。如果现在使用的是 Xarmain ，那么安装 lindexi.MVVM.Framework 就可以，这个库使用 dotnet framework 4.5 和 dotnet standard 2.0 ，所以在很多项目都可以使用。

## 项目要求

安装这个库的要求是 UWP 的最低版本是 16299 ，因为在 16299 才支持 dotnet standard 2.0，在之前的版本是不支持。

如果使用的是 WPF 项目，要求项目最低版本是 dotnet framework 4.5 

## 主界面

这个框架是适合有一个主界面和多个子页面的程序，而且适合多个子页面之间有通信，包括子页面让另一个页面跳转等的框架。

先创建一个 ViewModel 类，表示这是主界面。

```csharp
    public class ViewModel : NavigateViewModel
```

然后在 MainPage 添加 ViewModel ，因为需要做导航，所以需要在前台添加 Frame 用来做导航。







<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。