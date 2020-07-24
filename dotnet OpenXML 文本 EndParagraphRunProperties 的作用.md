# dotnet OpenXML 文本 EndParagraphRunProperties 的作用

其实我想要记录的仅仅只是 a:endParaRPr 对文本布局的作用

<!--more-->
<!-- 发布 -->

按照 [ECMA 376](http://www.ecma-international.org/publications/standards/Ecma-376.htm ) 的说法，此属性的作用是指定用户在此段落后开始输入其他文本时要保留的属性。只有在此段落的样式与段落本身不同时，才应设置此属性

文本的属性会受到页面以及模版和所在段落的影响，而 `a:endParaRPr` 是一个放在 `<a:p>` 里面的标记，用来表示这一段的结束，在 OpenXML SDK 的表示是 EndParagraphRunProperties 类，这个类继承 TextCharacterPropertiesType 类

而 TextCharacterPropertiesType 类就包含了 TextRun 的字符属性，如字体字号等信息，详细请看 [dotnet OpenXML 简单聊聊 PPT 文本解析](https://blog.lindexi.com/post/dotnet-OpenXML-%E7%AE%80%E5%8D%95%E8%81%8A%E8%81%8A-PPT-%E6%96%87%E6%9C%AC%E8%A7%A3%E6%9E%90.html )

因此可以在 `<a:endParaRPr>` 添加一些文本属性

```xml
<a:endParaRPr lang="zh-CN" altLang="en-US" sz="2400" b="1" dirty="0">
  <a:latin typeface="+mn-ea"/>
</a:endParaRPr>
```

添加了属性之后的作用就是让用户在这一段继续输入的时候，可以找到可以继承的属性。例如用户的整个文本框默认的字体是宋体，而期望一个空段落之后输入的内容是楷体，此时就可以应用上了 `<a:endParaRPr>` 设置 typeface 属性。此时输入的内容就会继承 `<a:endParaRPr>` 的属性

当然，在很多不规范的排版软件里面，可以使用 endParaRPr 表示段落结束的功能藏一些文本，如下面代码

```xml
<a:p>
  <a:pPr fontAlgn="auto">
    <a:lnSpc>
      <a:spcPts val="3800"/>
    </a:lnSpc>
  </a:pPr>
  <a:r>
    <a:rPr lang="en-US" sz="2400" b="1" dirty="0" smtClean="0">
      <a:latin typeface="+mn-ea"/>
    </a:rPr>
    <a:t>1</a:t>
  </a:r>
  <a:endParaRPr lang="zh-CN" altLang="en-US" sz="2400" b="1" dirty="0">
    <a:latin typeface="+mn-ea"/>
  </a:endParaRPr>
  <a:r>
    <a:rPr lang="en-US" altLang="zh-CN" sz="2400" b="1" dirty="0" smtClean="0">
      <a:latin typeface="+mn-ea"/>
    </a:rPr>
    <a:t>2</a:t>
  </a:r>
  <a:r>
    <a:rPr lang="en-US" altLang="zh-CN" sz="2400" b="1" dirty="0" smtClean="0">
      <a:latin typeface="+mn-ea"/>
    </a:rPr>
    <a:t>3</a:t>
  </a:r>
</a:p>
```

可以看到文本里面有三个 TextRun 分别是 1 和 2 和 3 的文本，但是在 1 后面添加了 endParaRPr 标记，那么此时的 PPT 如何显示？其实 只会显示 1 而不会显示后续内容

也就是读取了 `a:endParaRPr` 将会忽略后续的文本内容，表示这一段结束

在 UWP 使用 XAML 的换行，请看 [win10 uwp 在 xaml 让 TextBlock 换行](https://blog.lindexi.com/post/win10-uwp-%E5%9C%A8-xaml-%E8%AE%A9-TextBlock-%E6%8D%A2%E8%A1%8C.html )

这个属性和 [Text Line Break](https://blog.lindexi.com/post/dotnet-OpenXML-%E6%96%87%E6%9C%AC-Text-Line-Break-%E7%9A%84%E4%BD%9C%E7%94%A8.html ) 有些不同，原因是默认一段就是包含一个换行符，而 `<a:br/>` 就是让一段内可以包含多个换行

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
