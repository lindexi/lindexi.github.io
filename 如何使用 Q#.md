# 如何使用 Q#

Q# 是微软的量子语言，很厉害，所以本文告诉大家如何入门，如何配置。

<!--more-->

<!-- csdn -->

首先需要下载一些东西，打开[Microsoft Quantum Development Kit - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=quantum.DevKit )下载扩展，无法扩展无法下载，请到我上传的[csdn](http://download.csdn.net/download/lindexi_gd/10155909)下载

需要确定自己的 VisualStudio 是 VisualStudio 2017

需要关闭所有的 VisualStudio 才可以安装，在安装完成之后，先 clone 一下微软的项目，这个项目可以测试是否可以使用。

项目的地址：[Microsoft/Quantum: Microsoft Quantum Developer Kit Samples and Libraries](https://github.com/microsoft/quantum ) ，可以使用下面的代码克隆

```csharp
git clone https://github.com/Microsoft/Quantum.git
```

如果发现自己无法访问，那么可以使用 gitee 进行克隆，新建一个项目选择远程链接，这样就好。


打开 QsharpLibraries.sln 可以发现提示缺少库，一般是没有 F# ，需要安装一下。

然后还原nuget，需要一些时间，这时关闭博客园的nuget他没有这些。然后打开每个项目的引用，这样点击一下找不到的引用，就可以自动引用。

接着选择 TeleportationSample 项目，右击他为启动。

按下 F5 就可以看到输出

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201712122104.jpg)

请看 [Setting up the Q# development environment | Microsoft Docs](https://docs.microsoft.com/zh-cn/quantum/quantum-installconfig?view=qsharp-preview )

本文只是告诉大家如何搭建环境，实际没有告诉大家如何写Q#，在我后面有时间在写

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  