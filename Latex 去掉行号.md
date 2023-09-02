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



