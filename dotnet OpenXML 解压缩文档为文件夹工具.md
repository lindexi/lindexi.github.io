# dotnet OpenXML 解压缩文档为文件夹工具

做 Office 解析，是需要进行不断的测试才能了解 OpenXML 里面的属性的作用。根据 Ecma 376 的定义，文档其实只是一个压缩文件，可以使用压缩工具进行解压缩。但是我需要不断进行修改文档里面的属性，然后用 Office 打开，测试属性的效果，此时就需要有一个工具用来提升效率

<!--more-->
<!-- CreateTime:2020/7/23 19:50:15 -->

<!-- 发布 -->

工具的作用就是将 Office 文档，包括 Word 的 docx 和 PPT 的 pptx 和 Excel 的 xlsx 文档，解压缩文档到指定的文件夹里面

此时就可以使用编辑工具，如 SublimeText 或 VisualStudio 或 NotPad++ 等工具编辑 Office 文档的文件

编辑完成之后，可以通过工具将文件夹压缩为 Office 文档，使用 Office 打开，此时就可以看到属性的效果

工具在 GitHub 上完全开源，请看 [https://github.com/dotnet-campus/dotnetCampus.OfficeDocumentZiper](https://github.com/dotnet-campus/dotnetCampus.OfficeDocumentZipper)

使用方法是通过 dotnet 工具安装

```csharp
dotnet tool isntall -g dotnetCampus.OfficeDocumentZipper
```

安装完成之后，可以使用命令行开启，如下面代码

```csharp
OfficeDocumentZipper
```

如果不想写一个命令行，想要通过一个批处理启动，可以新建一个 x.bat 文件，添加下面代码

```csharp
dotnet tool update -g dotnetCampus.OfficeDocumentZipper

OfficeDocumentZipper
```

注意在 bat 文件里面使用 update 代替 install 这样每次打开都是最新版本

这个工具的界面很简单，理论上你看界面就能使用

这个工具需要填写需要解压缩的 Office 文档的文件路径，以及解压缩到的文件夹路径。点击 UnZip 就是将 Office 文档解压缩到文件夹，如果文件夹存在，那么将会覆盖原有的文件夹。为什么这样设计？原因是我使用 SublimeText 打开了文件夹里面的 Part 文档，此时我不知道在 Office 上做对应的设置的行为，于是我就在工具里面点击 Open 打开 Office 文档，然后在 Office 里面编辑保存一下，此时文档的内容更改了。于是点击 UnZip 可以覆盖原有文件夹内容，我只需要让 SublimeText 自动刷新就可以看到在 Office 的更改的内容

在通过编辑工具修改了解压缩之后的文件之后，可以通过工具压缩为新的 Office 文档。每次都会创建新的 Office 文档文件

为什么每次都创建新的文档文件？原因是之前的 Office 文件也许在被打开，此时写入会失败，同时多个版本的 Office 文件方便对比，这样就知道多个版本更改的属性的作用

打开 Office 文档需要你本地安装了 Office 才能打开

这是一个 WPF 的 dotnet tool 工具，因此如果你不在 Windows 系统运行，你需要使用 Wine 才能运行

如果你有任何问题，都可以在 [github](https://github.com/dotnet-campus/dotnetCampus.OfficeDocumentZiper) 提 Issus 告诉我，当然，我也欢迎小伙伴贡献代码

我写了很多 Office 解析相关的博客，请看 [Office 使用 OpenXML SDK 解析文档博客目录](https://blog.lindexi.com/post/Office-%E4%BD%BF%E7%94%A8-OpenXML-SDK-%E8%A7%A3%E6%9E%90%E6%96%87%E6%A1%A3%E5%8D%9A%E5%AE%A2%E7%9B%AE%E5%BD%95.html )

在 SublimeText 打开 Office 的解压缩的 Part 文件，都是没有格式化的文件，可以安装 Indent XML 插件格式化

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
