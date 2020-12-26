# WPF 框架开发 加入 InternalsVisibleToAttribute 特性让其他程序集可以访问 internal 权限成员

在 WPF 框架开发中，其实很少有开发者有足够的勇气去更改现有的框架逻辑，因为 WPF 的功能十分庞大，很难测试全。更多的开发都是加功能以及开放已有功能。整个 WPF 框架的大体设计是十分好的，可以在框架里面遵循对修改关闭的原则，定制化更多的是做注入，调用 internal 权限成员

本文告诉大家如何给 WPF 框架加入 InternalsVisibleToAttribute 特性让其他程序集可以访问 internal 权限成员

<!--more-->
<!-- CreateTime:2020/12/24 17:38:25 -->


<!-- 发布 -->

如果我不新加入一个程序集，而是在原有的程序集开发，加上新功能，是否可行？肯定是可以的，但是这样做的开发效率不够好。因为我为了在 VisualStudio 上构建 WPF 框架，我加入了一些构建黑科技，此时的 WPF 框架丢失了增量构建的功能。而一次 WPF 框架的构建需要 20 分钟

因为我在 WPF 框架的定制开发中，更多的是访问 internal 权限成员添加新的类等，我几乎没有想去更改现有的逻辑。所以最简方法就是添加一个程序集，让整个 WPF 框架的 internal 权限成员可以被我添加的程序集访问。实现的方法是 InternalsVisibleToAttribute 特性

因为 WPF 是一个带签名的框架，大家都知道，一个带签名的程序集如果设置 InternalsVisibleToAttribute 特性，要求被设置的程序集也是被签名的，详细请看 [dotnet 强签名下使用 InternalsVisibleToAttribute 给程序集加上友元](https://blog.lindexi.com/post/dotnet-%E5%BC%BA%E7%AD%BE%E5%90%8D%E4%B8%8B%E4%BD%BF%E7%94%A8-InternalsVisibleToAttribute-%E7%BB%99%E7%A8%8B%E5%BA%8F%E9%9B%86%E5%8A%A0%E4%B8%8A%E5%8F%8B%E5%85%83.html )

接下来我需要新建一个程序集，我这里叫 dotnetCampus.WPF 程序集，在这个程序集里面自己新建签名，然后配置到 WPF 框架各个项目

做法就是先通过 [dotnet 强签名下使用 InternalsVisibleToAttribute 给程序集加上友元](https://blog.lindexi.com/post/dotnet-%E5%BC%BA%E7%AD%BE%E5%90%8D%E4%B8%8B%E4%BD%BF%E7%94%A8-InternalsVisibleToAttribute-%E7%BB%99%E7%A8%8B%E5%BA%8F%E9%9B%86%E5%8A%A0%E4%B8%8A%E5%8F%8B%E5%85%83.html ) 拿到签名，然后拼接 InternalsVisibleToAttribute 的内容

```csharp
[assembly:InternalsVisibleTo("dotnetCampus.WPF, PublicKey=0024000004800000940000000602000000240000525341310004000001000100256f5cb79140dbc25623807d6823ca4b5b602209eaaf71f064e5926a7039c24351c1e2ad3130e194631307ed36a76ad4b832e237a467fefbd693428c7ecc5d4cc26796f6f8b705311948e00f2be5fa2db52ddff50a5b3eb0acc715b45618c1a92532ae2326529fb9e0f58a44abf31e9b5701994464186d3b9f52169b6e0f80b9")]
```

将上面代码放在 WPF 的每个项目，此时就可以完成了 WPF 的配置了。下一步就是将这个 WPF 仓库构建一下，可以使用命令行方式构建，详细请看 [手把手教你构建 WPF 框架的私有版本](https://blog.lindexi.com/post/%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E6%9E%84%E5%BB%BA-WPF-%E6%A1%86%E6%9E%B6%E7%9A%84%E7%A7%81%E6%9C%89%E7%89%88%E6%9C%AC.html )

在构建完成之后，从 WPF 的 artifacts 文件夹里面，可以在 `artifacts\packaging\Release\Microsoft.DotNet.Wpf.GitHub` 文件夹找到构建输出的所有内容

有两个方法，一个是打包为 NuGet 包，另一个方法是作为 Dll 引用。我推荐使用 Dll 引用的方式，这个方式使用起来更简单

做出 Dll 引用的方法需要先做一些准备

先创建一个空白的 WPF 引用，然后使用 `self-contained` 独立方式发布，发布的时候小心 x86 和 x64 的不同。使用 x86 和 x64 需要和 WPF 打包关联，我当前用的都是 x86 下的

拿出来空白 WPF 的发布输出内容，放在 `CustomWPF\Lib` 文件夹

将 `artifacts\packaging\Release\Microsoft.DotNet.Wpf.GitHub\lib\netcoreapp5.0` 的文件内容拷贝替换 `CustomWPF\Lib` 文件夹的文件，上面的 `netcoreapp5.0` 是需要根据你具体构建的 WPF 框架而更改

接着还需要将 `artifacts\packaging\Release\Microsoft.DotNet.Wpf.GitHub\runtimes\win-x86\native` 文件夹里面的 Native 部分内容也拷贝到 `CustomWPF\Lib` 文件夹。其实在 WPF 中使用 x86 或 x64 构建的不同就是 runtimes 文件夹内的文件而已

现在就构建完成了 Lib 自己定制版本的依赖文件了，接下来就是将 dotnetCampus.WPF 程序集拷贝出来，放在 CustomWPF 文件夹里面。在 dotnetCampus.WPF.csproj 添加引用

```xml
  <ItemGroup>
    <Reference Include="..\Lib\DirectWriteForwarder.dll" />
    <Reference Include="..\Lib\Microsoft.VisualBasic.Forms.dll" />
    <Reference Include="..\Lib\PresentationCore.dll" />
    <Reference Include="..\Lib\PresentationFramework-SystemCore.dll" />
    <Reference Include="..\Lib\PresentationFramework-SystemData.dll" />
    <Reference Include="..\Lib\PresentationFramework-SystemDrawing.dll" />
    <Reference Include="..\Lib\PresentationFramework-SystemXml.dll" />
    <Reference Include="..\Lib\PresentationFramework-SystemXmlLinq.dll" />
    <Reference Include="..\Lib\PresentationFramework.Aero.dll" />
    <Reference Include="..\Lib\PresentationFramework.Aero2.dll" />
    <Reference Include="..\Lib\PresentationFramework.AeroLite.dll" />
    <Reference Include="..\Lib\PresentationFramework.Classic.dll" />
    <Reference Include="..\Lib\PresentationFramework.dll" />
    <Reference Include="..\Lib\PresentationFramework.Luna.dll" />
    <Reference Include="..\Lib\PresentationFramework.Royale.dll" />
    <Reference Include="..\Lib\PresentationUI.dll" />
    <Reference Include="..\Lib\ReachFramework.dll" />
    <Reference Include="..\Lib\System.Design.dll" />
    <Reference Include="..\Lib\System.Drawing.Common.dll" />
    <Reference Include="..\Lib\System.Drawing.Design.dll" />
    <Reference Include="..\Lib\System.Drawing.dll" />
    <Reference Include="..\Lib\System.Printing.dll" />
    <Reference Include="..\Lib\System.Private.DataContractSerialization.dll" />
    <Reference Include="..\Lib\System.Private.Uri.dll" />
    <Reference Include="..\Lib\System.Private.Xml.dll" />
    <Reference Include="..\Lib\System.Private.Xml.Linq.dll" />
    <Reference Include="..\Lib\System.Security.Cryptography.OpenSsl.dll" />
    <Reference Include="..\Lib\System.Windows.Controls.Ribbon.dll" />
    <Reference Include="..\Lib\System.Windows.Forms.Design.dll" />
    <Reference Include="..\Lib\System.Windows.Forms.Design.Editors.dll" />
    <Reference Include="..\Lib\System.Windows.Forms.dll" />
    <Reference Include="..\Lib\System.Windows.Forms.Primitives.dll" />
    <Reference Include="..\Lib\System.Windows.Input.Manipulations.dll" />
    <Reference Include="..\Lib\System.Windows.Presentation.dll" />
    <Reference Include="..\Lib\System.Xaml.dll" />
    <Reference Include="..\Lib\UIAutomationClient.dll" />
    <Reference Include="..\Lib\UIAutomationClientSideProviders.dll" />
    <Reference Include="..\Lib\UIAutomationProvider.dll" />
    <Reference Include="..\Lib\UIAutomationTypes.dll" />
    <Reference Include="..\Lib\WindowsBase.dll" />
    <Reference Include="..\Lib\WindowsFormsIntegration.dll" />
  
  </ItemGroup>
```

现在尝试在 dotnetCampus.WPF 程序集内写代码，这里的代码可以访问 WPF 框架的 internal 成员

我将上面的制作完成的内容放在 [CSDN 下载](https://download.csdn.net/download/lindexi_gd/13769715) 欢迎小伙伴下载来试试

这部分的 WPF 代码我也放在 [GitHub](https://github.com/dotnet-campus/wpf/tree/bf90f3b21856fbe833cc72b9ed94cbaa5ec4dac0) 欢迎小伙伴访问

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
