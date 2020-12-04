
# Office 文档解析 文档格式和协议

本文讨论的 Office 文档指的是 Office 2007 及以后的 PPTX 和 xlsx 等格式的文件。在 Office 2007 之前使用的不公开标准的二进制格式定义。在 Office 2007 之后的文档格式使用 OOXML 国际标准定义，本文将告诉大家这个标准的协议和格式

<!--more-->


<!-- CreateTime:7/7/2020 10:56:35 AM -->



在 Office 2007 之后的 Office 文档格式采用的是 OOXML 标准格式。那什么是 OOXML 标准？这里的 OOXML 的全称是 Office Open XML File Formats 或被称为 OpenXML 格式，这是一个基于 zip+xml 定义的文档格式。这个标准最初是由 [ECMA-376](http://www.ecma-international.org/publications/standards/Ecma-376.htm ) 定义的，后来 [ISO/IEC 29500](https://www.iso.org/standard/71691.html) 也开始掺和 OOXML 格式的定义，不过可以认为从 ECMA-376 的第2版开始，这两个标准是一样的

微软的 Office 实现了 OOXML 格式，但仅实现其中一部分，详细描述请看 [官方文档](https://docs.microsoft.com/en-us/openspecs/office_standards/ms-offstandlp/d5784a8b-7070-466b-befa-b7bf3724c6f0?redirectedfrom=MSDN)

在 OOXML 格式里面，如上文所说是基于 zip+xml 定义的，这里的 Zip 提供文件的支持，而 xml 提供内容的支持。不过 OOXML 使用的 zip 也是有规范的，这里使用 OPC (Open Package Convention) 中文名叫 `开放打包协定` 作为文件存储格式。当然，这并非说 OPC 使用特殊的 zip 格式，而是 OPC 规定了文件存放的存储格式，然后将这些文件使用 zip 打包为一个文件。因此 一个 OPC 文件（不管其文件后缀是什么）本质上就是一个 zip 文件，你可以用任何常见的解压软件进行解压，解压后你看到的那些文件的组织结构，就是以 OPC 定义的方式存储的

在 [ECMA-376,Fourth Edition,Part 2](http://www.ecma-international.org/publications/standards/Ecma-376.htm) 详细定义了 OPC 开放打包协定。在 OPC 里面有三个重要的概念，分别是 Part 和 Relationship 和 ContentTypes 这三个

什么是 `Part` 简单理解就是 zip 的文件，每一个文件都是一个 Part 可以是任何格式，比如图片和xml文件等。在 Office 文件中，各种 Markup Language 定义的内容就作为 XML 存储在 Part 中

而 Relationship 是一种特殊的 Part 文件，它描述了各 Part 之间的依赖关系。根据OPC协议的规定，所有的 Relationship 都必须存储在名为 `_rels` 的文件夹中，并且所有 Relationship 的文件名都必须以 .rels 为后缀。每个 Part 可以根据自身的业务需求有一个对应的 Relationship 文件，这个对应的 Relationship 文件必须存放在这个 Part 文件所在文件夹的 `_rels` 文件夹里面，同时要求使用 Part 文件加上 `.rels` 后缀，不能使用其他名字

如有一个 PPTX 的页面是 `ppt\slides\slide1.xml` 此时这个页面的 Relationship 必须是 `ppt\slides\_rels\slide1.xml.rels` 文件，不能使用其他命名

最后一个 ContentTypes 相信小伙伴也不陌生，这是放在zip压缩包的根目录下的 `[Content_Types].xml` 文件，这是基本上每个 NuGet 包都会带的内容（不认识NuGet的小伙伴请点击右上角关闭按钮，因为你不要妄想玩转Office解析了），在 `[Content_Types].xml` 文件记录了该 OPC 压缩文件中除了他自己以外的所有文件的类型。这个文件作用是解决后缀名判断问题，在 OPC 定义里面不建议使用后缀名判断文件类型，而是根据 `[Content_Types].xml` 文件记录判断文件类型

讲完了文件存储方式，剩下的就是 XML 表示文件内容。在 OOXML 格式里面，可以认为是 `OPC + *ML` 的组合，这里的 `*ML` 表示的是各个标记语言，如 PML（Presentation Markup Language PPT 中各种数据的描述）等，这部分定义可以在 [ECMA-376,Fifth Edition,Part 1](http://www.ecma-international.org/publications/standards/Ecma-376.htm) 找到

解析 Office 文档用的比较多的包含 `PML (Presentation Markup Language pptx, PPT 中各种数据的描述)` 和 `WML (Wordprocessing Markup Language docx, Word 中数据的描述)` 和 `SML (Spreadsheet Markup Language xlsx, Excel 中数据的描述)` 和 `DML(Drawing Markup Language, Office 所有格式中都可以使用，用来描述矢量图形，图表等)` 和 `SharedML(Shared Markup Language, 描述了文档属性，音视频，图片，文档主题等内容，它被所有Office文件使用)` 等

这将会对应在 OpenXML SDK 的各个命名空间里面

[OfficeTalk: Essentials of the Open Packaging Conventions](https://docs.microsoft.com/en-us/previous-versions/office/office-12/ee361919(v=office.12)?redirectedfrom=MSDN )

[Open XML SDK](https://docs.microsoft.com/en-us/office/open-xml/open-xml-sdk )

[ECMA-376](http://www.ecma-international.org/publications/standards/Ecma-376.htm )

[ISO/IEC 29500](https://www.iso.org/standard/71691.html)

更多请看 [Office 使用 OpenXML SDK 解析文档博客目录](https://blog.lindexi.com/post/Office-%E4%BD%BF%E7%94%A8-OpenXML-SDK-%E8%A7%A3%E6%9E%90%E6%96%87%E6%A1%A3%E5%8D%9A%E5%AE%A2%E7%9B%AE%E5%BD%95.html )

 		
 	 	
 	 	




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。