# dotnet OpenXML 简单聊聊 PPT 文本解析

在 Office 里面的文本解析最全的范围是 Word 文本，就是属性数量本身就特别多。本文只是简单和大家聊聊 Office 里面的 PPT 的文本的解析入门。尽管 PPT 的文本也是采用 DrawingDL 的文本属性为主，不过会用到的属性将比 Word 少很多。本文将和小伙伴介绍 PPT 的文本存放的方式

<!--more-->
<!-- 发布 -->

在 PPT 中的文本框也是形状，只是形状里面添加了特殊的设置。而没有添加特殊设置的形状也可以添加文本，在 PPT 的文本使用 `<p:txBody>` 包含，这就是本文主要和大家介绍的内容

阅读本文，你将能大概了解如何开始入手 PPT 的文本解析，以及了解相应的工作量

在开始之前，我期望你是了解 PPT 的整个元素的存放格式的，请看 [dotnet OpenXML 解析 PPT 页面元素文档格式](https://blog.lindexi.com/post/dotnet-OpenXML-%E8%A7%A3%E6%9E%90-PPT-%E9%A1%B5%E9%9D%A2%E5%85%83%E7%B4%A0%E6%96%87%E6%A1%A3%E6%A0%BC%E5%BC%8F.html )

最简单的文本元素，当然，我这里的简单说的是文本解析层的。此时的文本将不引用全局的和占位符等的样式信息，此时的文本将包含自己的所有信息，此时的文本解析是最简单的

简单的文本在 `<p:txBody>` 里面将会包含一个 `<a:bodyPr>` 的值，这个值对应在 OpenXML SDK 的 DocumentFormat.OpenXml.Drawing.BodyProperties 类。也就是在 `<a:bodyPr>` 有啥属性和值都可以在 BodyProperties 这个类里面找到，当然有些可选的值只能通过 GetFirstChild 方法获取了，这部分就不细说了

需要科普的，放在 `DocumentFormat.OpenXml.Drawing` 的内容意味着是 DrawingDL 的定义，也就是将是 Office 多个格式如 PPT 和 Word 等共同使用的定义，因此对 DrawingDL 的解析完成基本上对 Office 的多个格式的这部分解析也是算完成。关于 DrawingDL 等 Office 多个格式的关系，请小伙伴看 [Office 文档解析 文档格式和协议](https://blog.lindexi.com/post/Office-%E6%96%87%E6%A1%A3%E8%A7%A3%E6%9E%90-%E6%96%87%E6%A1%A3%E6%A0%BC%E5%BC%8F%E5%92%8C%E5%8D%8F%E8%AE%AE.html )

在 PPT 的 `<p:txBody>` 的 `<a:bodyPr>` 表示整个文本本身的属性，包括文本是横排还是竖排，文本是否自适宽度高度等信息

在 PPT 的文本是富文本，可以对文本的文本段落进行设置，同时可以对文本的每个字符进行设置样式。因此在 PPT 的文档会给每一段添加段落属性。给每一段里面的某些样式相同的字符添加文本属性，为什么不是给每个字符单独一个属性？原因是这样做会让文本属性太多了。因此连续的文本如果有相同的样式，那么就可以使用相同的一个文本属性

在 `<p:txBody>` 标准的文档将会在 `<a:bodyPr>` 的下方添加 `<a:p>` 段落信息，一个文本框会包含多个段落信息，大概的格式如下

```xml
<p:txBody>
  <a:bodyPr>
  </a:bodyPr>

  <a:p></a:p>
  <a:p></a:p>
</p:txBody>
```

在 `<a:p>` 包含一个段落信息，在 OpenXML SDK 里面使用 DocumentFormat.OpenXml.Drawing.Paragraph 类。在段落里面，会包含很多 `<a:r>` 文本，以及段落属性

段落属性使用 `<a:pPr>` 表示，包含段落的行距等段落信息，具体是哪些属于段落信息？打开 PPT 软件，选择一个文本，可以看到段落的设置，这些就是段落信息。在 OpenXML SDK 里面使用 DocumentFormat.OpenXml.Drawing.ParagraphProperties 类

<!-- ![](image/dotnet OpenXML 简单聊聊 PPT 文本解析/dotnet OpenXML 简单聊聊 PPT 文本解析0.png) -->

![](http://image.acmx.xyz/lindexi%2F2020722178122775.jpg)

小伙伴可以看到在 PPT 软件的段落设置旁边有字体设置，这是对应放在 `<a:r>` 文本的内容，格式如下

```xml
<p:txBody>
  <a:bodyPr></a:bodyPr>
  <a:p>
    <a:pPr fontAlgn="auto">
    </a:pPr>
    <a:r>
      <a:rPr b="1" dirty="0" lang="en-US" sz="2400">
        <a:latin typeface="+mn-ea"/>
      </a:rPr>
      <a:t>欢迎</a:t>
    </a:r>
    <a:r>
      <a:rPr altLang="en-US" b="1" dirty="0" lang="zh-CN" sz="2400">
        <a:latin typeface="+mn-ea"/>
      </a:rPr>
      <a:t>访问我博客 https://blog.lindexi.com 里面有大量 UWP WPF 博客</a:t>
    </a:r>
  </a:p>
</p:txBody>
```

在 `<a:r>` 里面包含 `<a:rPr>` 文本属性，也就是上面图片 PPT 软件的 字体 的设置的内容，包含具体的字体字号等属性。这里的 `<a:rPr>` 在 OpenXML SDK 里面使用 DocumentFormat.OpenXml.Drawing.RunProperties 类。上面仅是例子，关于属性的作用和含义请忽略

同时在 `<a:r>` 里面将包含 `<a:t>` 也就是 DocumentFormat.OpenXml.Drawing.Text 的值，从上面的格式可以看到，这就是纯文本的字符串

因此最简单的解析 PPT 的文本，其实就是需要先拿到整个本文的属性，也就是 `<a:bodyPr>` 的值，然后分段解析每个 `<a:p>` 的值。在解析 `<a:p>` 的值包括解析段落的属性 `<a:rPr>` 的值和段落里面包含的文本 `<a:r>` 的值，而文本本身包含纯文本 `<a:t>` 和文本属性 `<a:rPr>` 的值

也就是 PPT 的文本排版其实就是拿出纯文本，然后将纯文本先按照文本的富文本属性的不同分组，如不同的文本有不同的字号和颜色等。接着按照文本的段落，也就是换行符将文本分为多个段落，再添加每个段落的段落属性，如行距等

最后的文本需要包含这些段落和文本框的属性，如横排竖排显示等

从本文上面的描述，其实 PPT 用到的文本格式大部分都是 DrawingDL 的定义，也就是其实上面的规则不只是适用于 PPT 一个格式，对 Excel 等也适合

那么解析的工作量大的在哪里？第一点是在收集这些属性上面，在 PPT 里面最简单的文本才会将所有的属性都添加上，而基本上的 PPT 文档里面是不会将所有的属性添加的，那么这些没有添加的属性使用的是默认属性？其实不一定，因为在 PPT 中的属性是有继承的，从页面继承 SlideLayout 属性，从 SlideLayout 继承 SlideMaster 属性。又有文本的样式等级，如 Level1ParagraphProperties 和 Level2ParagraphProperties 等，收集对的属性的工作量很大

等等，什么是 SlideLayout 和 SlideMaster 呀，请看 [dotnet OpenXML 的 Slide Master 和 Slide Layout 是什么](https://blog.lindexi.com/post/dotnet-OpenXML-%E7%9A%84-Slide-Master-%E5%92%8C-Slide-Layout-%E6%98%AF%E4%BB%80%E4%B9%88.html )

第二点就是属性的数量本身，在 OpenXML 里面定义了大量的文本可以被设置的属性，光是定义这些属性就需要大量的代码。而在定义之后还需要了解属性的含义和作用，有些属性本身会相互影响的，就更坑了。好在几乎所有的属性能做出来的效果，都能在 WPF 里面实现，我还没有遇到只有 PPT 能实现的渲染效果而在 WPF 不能实现的。当然使用 WPF 的富本文控件是做不出效果的，需要自己写一个文本库

因此整个 PPT 的文本解析里面的工作量都在属性上面，也就是了解 PPT 的纯文本在加上这些属性之后会有啥的呈现就是解析文本的主要工作

我写了很多 Office 解析相关的博客，请看 [Office 使用 OpenXML SDK 解析文档博客目录](https://blog.lindexi.com/post/Office-%E4%BD%BF%E7%94%A8-OpenXML-SDK-%E8%A7%A3%E6%9E%90%E6%96%87%E6%A1%A3%E5%8D%9A%E5%AE%A2%E7%9B%AE%E5%BD%95.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
