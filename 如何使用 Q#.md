# 如何使用 Q#

Q# 是微软的量子语言，很厉害，所以本文告诉大家如何入门，如何配置。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:51 -->


<!-- csdn -->

## 介绍

很多新的计数机技术都在很多年前就有人提出，量子计算就是其中一个。量子计算在 1980 年就被 Richard Feynman 和 Yuri Manin 提出，因为量子比较特殊，所以难以被人们的容易接受，而且因为计算机的价格比较高所以大家比较少可以测试和开发。现在微软弄了Q#和模拟器，大家可以开始学习如何使用量子计算，这是一个简单的语言，可以很多的使用 C# 的方式。不过还是有一些不同的地方，在使用之前，需要了解很多量子的东西。

## 下载环境

首先需要下载一些东西，打开[Microsoft Quantum Development Kit - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=quantum.DevKit )下载扩展。扩展无法下载，请到我上传的[csdn](http://download.csdn.net/download/lindexi_gd/10155909)下载

需要确定自己的 VisualStudio 是 VisualStudio 2017

需要关闭所有的 VisualStudio 才可以安装，在安装完成之后，先 clone 一下微软的项目，这个项目可以测试是否可以使用。

项目的地址：[Microsoft/Quantum: Microsoft Quantum Developer Kit Samples and Libraries](https://github.com/microsoft/quantum ) ，可以使用下面的代码克隆

```csharp
git clone https://github.com/Microsoft/Quantum.git
```

如果发现自己无法访问，那么可以使用 gitee 进行克隆，新建一个项目选择远程链接，这样就好。

打开 QsharpLibraries.sln 可以发现提示缺少库，一般是没有 F# ，需要安装一下。

但是这时会发现，项目无法编译，需要还原Nuget，微软虽然提高了在中国的 Nuget 速度，但是实际还原的时间还是可以去弄一些咖啡，回来差不多就还原好了。注意这时需要关闭博客园的 Nuget ，他没有这些东西，如果使用了他了，就需要清理文件夹。

因为垃圾 VS 不能帮你把库引用，所以需要在还原成功打开每个项目的引用，点击一下找不到的引用，这时会自动引用。

## 测试

如果发现所有库都安装，那么就可以开始编译，选择 TeleportationSample 项目，右击他为启动。
编译的时候注意关闭 Resharper 的编译，这时需要使用 VS 的编译，虽然我小伙伴使用 Resharper 可以成功，但是我自己失败了。

按下 F5 就可以看到输出，如果可以看到下面的输出，那么你的环境就弄好了

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201712122104.jpg)

请看 [Setting up the Q# development environment ](https://docs.microsoft.com/zh-cn/quantum/quantum-installconfig?view=qsharp-preview )

本文只是告诉大家如何搭建环境，实际没有告诉大家如何写Q#，在我后面有时间在写

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  