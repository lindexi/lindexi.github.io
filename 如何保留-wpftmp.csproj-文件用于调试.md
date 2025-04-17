
# 如何保留 wpftmp.csproj 文件用于调试

在构建 WPF 的过程，会生成 wpftmp.csproj 中间项目文件，用这个文件来辅助 XAML 构建过程。中间项目文件会在构建完成之后被删除，本文告诉大家如何保留 wpftmp.csproj 文件用于调试

<!--more-->


<!-- CreateTime:2024/11/23 07:13:58 -->

<!-- 发布 -->
<!-- 博客 -->

设置方法是添加 `<GenerateTemporaryTargetAssemblyDebuggingInformation>true</GenerateTemporaryTargetAssemblyDebuggingInformation>` 到项目属性里面，修改之后的 csproj 项目文件的代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net9.0-windows</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <UseWPF>true</UseWPF>
    <GenerateTemporaryTargetAssemblyDebuggingInformation>true</GenerateTemporaryTargetAssemblyDebuggingInformation>
  </PropertyGroup>

</Project>
```

设置此属性之后，在 VisualStudio 里面重新构建项目，就可以看到创建的 wpftmp.csproj 中间文件不被删除

保留这个中间项目文件可以帮助大家了解 WPF 构建过程，用于调试构建过程

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/6bc0a171f53f4da8a968257c621c9df7a6de77a3/WPFDemo/KeefearjerebuJuryeryochear) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/6bc0a171f53f4da8a968257c621c9df7a6de77a3/WPFDemo/KeefearjerebuJuryeryochear) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 6bc0a171f53f4da8a968257c621c9df7a6de77a3
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 6bc0a171f53f4da8a968257c621c9df7a6de77a3
```

获取代码之后，进入 WPFDemo/KeefearjerebuJuryeryochear 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。