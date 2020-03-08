# 使用 Pandoc 把 Markdown 转 Docx

最近在写文档，但是有小伙伴比较渣，他只会使用 Word 为了照顾这些比较渣的小伙伴，我需要把我的 Markdown 文件转换为 Word 给他们。

<!--more-->
<!-- CreateTime:2018/10/23 10:56:18 -->

<!-- csdn -->
<!-- 标签：pandoc -->

首先需要下载 Pandoc ，可以从我的网盘下载 打开 <http://lindexi.ys168.com/> 点击 UWP 文件夹里面就可以下载

或者到 [Pandoc 转换 Markdown 为 pdf-CSDN下载](https://download.csdn.net/download/lindexi_gd/10437151 )

然后使用命令行进入 Pandoc 解压出来的路径，例如我需要把 `E:\lindexi\win10 uwp 如何开始开发.md`转换为`win10 uwp 如何开始开发.docx`，那么就需要使用下面的代码

```csharp
pandoc.exe -s -o "E:\lindexi\win10 uwp 如何开始开发.docx" "E:\lindexi\win10 uwp 如何开始开发.md" --mathjax
```

这里的代码意思是 `-o` 输出路径，通过输出路径的后缀，可以知道需要转换为什么。除了转换 word 还可以转换 pdf 。

加上空格就是 Markdown 文件，后面`--mathjax`表示添加数学公式的支持。

我自己尝试了转换，感觉不错

实际上 pandoc 是强大的文档转换工具，可以相互转换下面的格式

 - pdf

 - word

 - markdown

 - tex

 - html

如果需要做 Latex 转 pdf 也可以使用这个工具，参见 [You got LaTeX in my Markdown!](https://kesdev.com/you-got-latex-in-my-markdown/ )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
