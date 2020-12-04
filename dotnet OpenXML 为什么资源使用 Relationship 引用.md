# dotnet OpenXML 为什么资源使用 Relationship 引用

在 OpenXML 文档格式里面，所有的资源以及页面之间的引用等，都是通过 Relationship 的引用，如资源需要通过 GetReferenceRelationship 的方法才能拿到。那为什么要这样设计呢

<!--more-->
<!-- CreateTime:7/8/2020 11:16:11 AM -->



在做 Office 解析，可以看到资源的引用，如图片的引用等，不是应用相对的文件路径，而是使用 `r:id="xx"` 的方式引用，而实际的引用文件需要在 xx.rels 文件里面才能找到引用的路径

尽管在 OpenXML SDK 里面这些细节已经被封装好了，只需要通过 GetReferenceRelationship 方法就可以拿到对应的资源，但我好奇为什么 Office 这样设计


在 [Office 文档解析 文档格式和协议](https://blog.lindexi.com/post/Office-%E6%96%87%E6%A1%A3%E8%A7%A3%E6%9E%90-%E6%96%87%E6%A1%A3%E6%A0%BC%E5%BC%8F%E5%92%8C%E5%8D%8F%E8%AE%AE.html ) 我和小伙伴讲了 Office 文档的格式，这里存储的方式使用的是 OPC (Open Package Convention) 协议

在 OPC 协议里面要求多个 Part 也就是文件之间不能相互引用，如果两个 Part 有引用，需要在 Part 的 `rels` 文件里面添加引用，而在 Part 里面只使用对应的 `rels` 文件的记录资源的 Id 的值

那 Part 的 `rels` 又是什么？在 OPC 里面规定的 Part 可以理解为文件，因为 OPC 是基于 Zip 的打包方法，而 Zip 里面都是文件。而 `rels` 其实就是 OPC 里面的 Relationship 概念，这个 Relationship 是一种特殊的 Part 文件，它描述了各 Part 之间的依赖关系。根据OPC协议的规定，所有的 Relationship 都必须存储在名为 `_rels` 的文件夹中，并且所有 Relationship 的文件名都必须以 `.rels` 为后缀。每个 Part 可以根据自身的业务需求有一个对应的 Relationship 文件，这个对应的 Relationship 文件必须存放在这个 Part 文件所在文件夹的 `_rels` 文件夹里面，同时要求使用 Part 文件加上 `.rels` 后缀，不能使用其他名字

如某个 PPT 页面 slide1.xml 引用了某个音频文件，那么这个页面不能直接写音频文件的相对路径，而是需要在 slide1.xml 所在文件夹新建一个 `_rels` 文件夹，在里面放一个 `slide1.xml.rels` 文件，如下

```csharp
ppt\slides\slide1.xml
ppt\slides\_rels\slide1.xml.rels
```

按照 OPC 的定义，在 Relationship 里面定义引用，假设音频文件存放在 `ppt\media\image1.png` 文件，那么对应的对应的 `slide1.xml.rels` 文件内容可以如下

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<Relationships xmlns="http://schemas.openxmlformats.org/package/2006/relationships">
  <Relationship Id="rId1" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/image" Target="../media/image1.png"/>
</Relationships>
```


使用 `Relationship` 定义 Id 的值，用来给 `slide1.xml` 引用，同时配置资源类型，通过 Type 定义，最后使用 Target 属性引用文件

此时在 `slide1.xml` 就可以根据 Id 引用资源，如以下代码

```xml
<a:blip r:embed="rId1"/>
```

此时通过 `rId1` 就可以在 `slide1.xml.rels` 找到对应的资源，然后通过资源的相对路径拿到文件

在 OpenXML SDK 里面将这部分都封装了，不需要咱自己去找对应的文件，通过 GetPartById 或 GetReferenceRelationship 传入资源的 Id 就可以拿到对应的资源。在 OPC 里面的定义，可以知道使用 Part 表示文件等。因此 GetReferenceRelationship 返回的是 ReferenceRelationship 类，根据对象转换为 DataPartReferenceRelationship 或 ExternalRelationship 等

在 2.11 版本的 DocumentFormat.OpenXml 库里面添加了我的代码，可以使用 TryGetPartById 方法在 OpenXmlPartContainer 尝试获取资源。因为默认的 GetPartById 将会在找不到资源的时候抛出 ArgumentOutOfRangeException 而如果文档是用户创建的，也许他用的是 WPS 等软件做的文档不遵守标准，此时就会炸了

```csharp
                if (Slide.SlidePart.TryGetPartById(id, out var part))
                {
                }
```

那为什么只有 GetPartById 添加了 TryGetPartById 方法，而 GetReferenceRelationship 没有？在获取不到资源的时候，会在 GetReferenceRelationship 里面抛出 KeyNotFoundException 提示

原因是使用 GetReferenceRelationship 时，一般都可以确定 Id 是否存在，因为有 HyperlinkRelationships 和 DataPartReferenceRelationships 等属性的存在，可以通过这些属性进行判断

关于 Relationship 的一个应用请看 [C# dotnet 使用 OpenXml 解析 PPT 里面的视频](https://blog.lindexi.com/post/C-dotnet-%E4%BD%BF%E7%94%A8-OpenXml-%E8%A7%A3%E6%9E%90-PPT-%E9%87%8C%E9%9D%A2%E7%9A%84%E8%A7%86%E9%A2%91.html)

更多请看 [Office 使用 OpenXML SDK 解析文档博客目录](https://blog.lindexi.com/post/Office-%E4%BD%BF%E7%94%A8-OpenXML-SDK-%E8%A7%A3%E6%9E%90%E6%96%87%E6%A1%A3%E5%8D%9A%E5%AE%A2%E7%9B%AE%E5%BD%95.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
