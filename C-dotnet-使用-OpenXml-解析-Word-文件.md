
# C# dotnet 使用 OpenXml 解析 Word 文件

本文聊的 Word 是 docx 格式，这个格式遵循 ECAM 376 全球标准，使用的格式是 OpenXML 格式，在 2013 微软开源了 OpenXml 解析库。这个库里面包含了海量代码，可以使用 MB 计算的代码量，通过这个解析库，咱可以使用几行代码完成对 Word 文件的解析，从文件到内存模型

<!--more-->


<!-- csdn -->

本文通过一个简单的 WPF 程序告诉大家如何解析，这个简单的 WPF 程序简单到仅一个拖放功能，将 Word 文件拖入应用，就可以自动解析 Word 里面的内容

先新建一个简单的 Word 文件

![](http://image.acmx.xyz/lindexi%2F2020952036365930.jpg)

然后新建一个 WPF 程序，在这个程序里面添加简单的界面

```xml
    <Grid>
        <Border Background="Gray" AllowDrop="True" DragEnter="UIElement_OnDragEnter"></Border>
    </Grid>
```

在 WPF 中通过设置 AllowDrop="True" 就可以让控件支持接收拖放的文件

接着通过 NuGet 安装 [Openxml](https://www.nuget.org/packages/DocumentFormat.OpenXml) 库，这个库支持跨平台。我新建的是 WPF 的 .NET Core 版本，此时可以在 csproj 添加下面代码进行安装

```xml
  <ItemGroup>
    <PackageReference Include="DocumentFormat.OpenXml" Version="2.11.3" />
  </ItemGroup>
```

这个版本的 DocumentFormat.OpenXml 库包含了我的垃圾代码，这是一个在 GitHub 上开源的库，所有的小伙伴都可以参与开发

在 WPF 的后台代码添加 UIElement_OnDragEnter 方法，在这个方法里面可以使用下面代码拿到拖放的文件

```csharp
        private void UIElement_OnDragEnter(object sender, DragEventArgs e)
        {
            e.Handled = true;
            var fileList = (string[]) e.Data.GetData("FileDrop");

        }
```

在解析 Word 文档，可以使用下面代码就可以输出 Word 文档里面的内容

```csharp
            using (FileStream fs = new FileStream(fileList[0], FileMode.Open, FileAccess.Read, FileShare.ReadWrite))
            {
                using (var doc = WordprocessingDocument.Open(fs, false))
                {
                    var mainDocumentPart = doc.MainDocumentPart;
                    var body = mainDocumentPart.Document.Body;
                    Console.WriteLine(body.InnerText);
                }
            }
```

![](http://image.acmx.xyz/lindexi%2F202095203446551.jpg)

可以看到代码非常简单，但是如果想要将整个 Word 的文档的内容解析出来，这个就复杂一些

我写了一些解析的文档，其实 Word 主要就是文本解析部分复杂，请看 [Office 使用 OpenXML SDK 解析文档博客目录](https://blog.lindexi.com/post/Office-%E4%BD%BF%E7%94%A8-OpenXML-SDK-%E8%A7%A3%E6%9E%90%E6%96%87%E6%A1%A3%E5%8D%9A%E5%AE%A2%E7%9B%AE%E5%BD%95.html )



代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/f647bd45/JallweawhairnoFairwherenajajal) 欢迎小伙伴访问






<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。