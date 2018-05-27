
# Latex 去掉行号

本文主要讲如何去掉Latex的行号

<!--more-->


<!-- 标签：Latex -->

<div id="toc"></div>

删除`\modulolinenumbers`删除所有`\linenumbers`

删除`\usepackage{lineno,hyperref}`

modulolinenumbers就是每多少行显示行数，每一页重新开始

## latex引用多篇参考文献

尝试引用：`\usepackage[numbers,sort&compress]{natbib}`

因为自带的有了，我就修改

```csharp
\biboptions{numbers,sort&compress}

```

结果没有用。


使用了`\usepackage{cite}`编译错误，原因还没找

natbib 错误 `Option clash for package natbib.`是有多次使用

打开`elsarticle.cls`

修改

```csharp
\RequirePackage[\@biboptions]{natbib}

```

```csharp
\RequirePackage[numbers,sort&compress]{natbib}

```
在导言

```csharp
\newcommand{\ucite}[1]{\textsuperscript{\cite{#1}}}

```

引用用`\ucite{1,2,3}`







<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。