这几天在开始写论文，准备发的是elsevier，这个网站的instruction有问题，下载的东西基本上好多的错误，所以我就写博客记录。

<!--more-->

首先看下：https://www.elsevier.com/authors/author-schemas/latex-instructions

这里需要我们先安装Latex，我安装的是Ctex

CTex的下载可以在网上搜索或使用我网盘，如果我网盘没法使用请和我说：[lindexi_gd@163.com](mailto:lindexi_gd@163.com)

我们首先安装Ctex，一路下一步，注意可以安装到D盘，不一定需要C盘。

然后就是`basic-miktex-2.9.6069-x64`这里你不一定安装64位，下载可以到[http://miktex.org/](http://miktex.org/) 下载，也可以到我网盘下载，如果不能下载请和我说

安装完CTeX如果使用报错

>CTeX 系统找不到指定的文件

大概的方法是打开Execution Modes在Options

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=cbd5295a82030a015ea0a318299d1b4e)

然后修改Tex System的路径这样很简单我们的CTex就好了

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=9e6cf40a898f22bda6a8ebd342644c23)

看到CTEX的MikTex不存在Bin，那么需要安装miktex，这个安装必须在C盘，默认位置，记住**C盘**

安装完，我们可以安装TeXStudio，这个软件做的很好，看下界面

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=471b24d1b321541a9cb98d5081c57c20)

是不是感觉比WinEdt好，如果感觉好的话，那么就去下载，可以到官网[www.texstudio.org](www.texstudio.org)如果上不了就在我这里下。

我们开始使用官方的模板，官方下载位置 [https://www.elsevier.com/__data/assets/file/0007/56842/elsarticle-template.zip](https://www.elsevier.com/__data/assets/file/0007/56842/elsarticle-template.zip)可以看到里面没有`ecrc.sty`在我们编译的时候就出错

```csharp
File `ecrc.sty' not found.
```

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=0d2dc427f5df19c6fd3d5c26480a3eab)

可以去下载我的`ecrc.sty`，下载放在最后，大家如果发现无法下载，就联系我

这个在于我们打开的模板是`ecrc-template.tex`，在官网有下载，[https://www.elsevier.com/__data/assets/text_file/0007/56878/ecrc-template.tex](https://www.elsevier.com/__data/assets/text_file/0007/56878/ecrc-template.tex)

打开之后发现还需要Elsevier-logo-3p.pdf, SDlogo-3p.pdf, Elsevier-logo-5p.pdf and SDlogo-5p.pdf ，都可以在官网下载，下载后需要下载elsarticle.cls，同样官方，[https://www.elsevier.com/__data/assets/text_file/0005/56903/elsarticle.cls](https://www.elsevier.com/__data/assets/text_file/0005/56903/elsarticle.cls)

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=af60a44f45f5a6a31bac45018e8f42ef)

把这些放在一个文件夹，这样就好啦。

我在TeXStudio打开，发现没有错误。

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=7263d9f243b82dbd0ddb75b2d7ffa77d)

顺便推荐一个软件：[Tickeys](http://www.yingdev.com/projects/tickeys)这个软将可以让我们打字有声音，晚上打字用这个软件感觉好。

开始写的是`\documentclass[3p,times]{elsarticle}`

就是引用elsarticle，`\documentclass[]{elsarticle}` 选项可以使用参见：https://www.elsevier.com/__data/assets/pdf_file/0009/56844/elsdoc2.pdf

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=8345e12176ee74d31704d5702819e9ec)

我们开始就是题目，我们的题目是乱写

首先题目`\title{A paper csdn and Mircrosoft is laji \tnoteref{t1,t2}}`，其中`\tnoteref`是引用脚注，一般写的就是t1,t2标识，这个随意。其中写了t1，在题目后面就有一个星，在写一个就出现一个逗号，后面加两个星，规则是第n个就有n个星。

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=47006b8ad7f2cc53a29d35d4674f3f98)

然后我们在外面写`\tnotetext[标识]{Content}`，也就是页面下的内容，这里我写的是垃圾微软，可以看到Content可以很长，而我们还有t3没写，没写的没显示

```csharp
\tnotetext[t1]{This document is a collaborative effort.}
\tnotetext[t2]{The second title footnote which is a longer
	longer than the first one and with an intention to fill
	in up more than one line while formatting.But it's to small,I write Mircosoft is laji.}
```

接着我们开始写作者，假如我们两个作者，第一个是`lindexi_gd`第二个是`Microsoft`

我们需要知道在Latex的注释是`%%`

作者的格式是

```csharp
%% \author[label1,label2]{<author name>}
%% \address[label1]{<address>}
%% \address[label2]{<address>}
```

每个label就是标签，可以在地址写，可以在别的地方写作者的地址

Latex用`\\`换行，在我们的地址比较长，可以用这换行

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=a1b9c47f89679369883919fc05be1d6f)

```csharp
\author[lindexiAddress]{lindexi\_gd \corref{cor1} \fnref{fn1} }
\ead{lindexi\_gd@163.com}

%%作者的\corref{lable}就是在脚注，用的是*，第n个有n*
%%\fnref{lable} 脚注，这个才是真的，但是没显示
%%\author[lindexiAddress]{lindexi\_gd \corref{cor1} \fnref{fn1} }
%%地址写在下面\address[lindexiAddress]{The lindexi's address }
%%地址是a,b,c  label不区分大小写

\author[Address1,Address2]{Mircrosoft \corref{cor2} \fnref{fn2}}
\ead{kaveh@river-valley.com}

\cortext[cor1]{Corresponding author}
\cortext[cor2]{Principal corresponding author}

\fntext[fn1]{This is the specimen author footnote.}
\fntext[fn2]{Another author footnote, but a little more longer.}
\fntext[fn3]{Yet another author footnote. Indeed, you can have
	any number of author footnotes.}

\address[lindexiAddress]{The lindexi's address }

\address[Address1]{River Valley Technologies, 9, Browns Court,
	Kennford, Exeter, United Kingdom}
\address[Address2]{Central Application Management,
	Elsevier, Radarweg 29, 1043 NX\\
	Amsterdam, Netherlands}
```

复制我这个放到代码就可看到
![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=a1b9c47f89679369883919fc05be1d6f)

然后就是我们的摘要

```csharp
\begin{abstract}
摘要写在这
\end{abstract}
```

然后我们开始写关键字

关键字写在

```csharp

\begin{keyword}

\end{keyword}
```
不同的关键字使用`\sep`分开

```csharp
\begin{keyword}

CSDN \sep lindexi \sep windows.sc

\end{keyword}
```

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=831ef3dfb69b9b72c6c699b55bd476ab)

我们就在正文开始写，遇到问题是如何插入图片，插入算法，图片位置不对，这写我都找了连接

### 输入列表

```csharp
\begin{enumerate}[标号]

```
标号可以使用数字或字，其中可以加`[]`,`()`

我们可以在列表嵌套

```csharp
\begin{enumerate}[a)]
	\item This item has roman numeral counter.
	\item Another one before we close the third level.
	\begin{enumerate}[a)]
		\item This item has roman numeral counter.
		\item Another one before we close the third level.
	\end{enumerate}
\end{enumerate}
```

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=b47139339692da2ae2ae17767f35a117)

### 双栏

需要双栏，在`\documentclass[3p,times]{elsarticle}`改5p

还有其他选项，这些我们现在不需要，直接改5p就好`\documentclass[5p,times]{elsarticle}`

### 引用

我们的论文会参见很多人的，那么添加一个人或删会让我们修改很多，Latex给我们简单方法，我们可以使用这个来添加我们的参考

首先在页面最下，`\end{document}`上面写参考，首先要删除`\bibliography{<your-bib-database>}`

然后写参考的

```csharp
\begin{thebibliography}{00}
	
	\bibitem[每次参考自动写的内容，作者，如果遇到括号不写]{label} 参见lindexi
	
\end{thebibliography}
```

在文章遇到我们需要参考

```csharp
\citet{label}
```
这样就好，如果我们没写作者看起来不好

如果没写作者，只要一个[1]那么在正文

```csharp
\citep{label}

另一方式

\cite{label} 
```

要写一个上标我们应该用`^`

假如我们的一个引用

```csharp
\begin{thebibliography}{00}    
    
    \bibitem{lindexi 1} lindexi_gd csdn.
	
\end{thebibliography}
```

在引用的写`  $^{\cite{lindexi 1}}$`

注意`$$`就是使用公式

### 标题

我们需要一级标题，我们还需要二级，那么一级标题`\section{标题名}`

二级标题`\subsection{标题名}`

### 斜体

```csharp
\emph{内容}
```

### 粗体

```csharp
\textbf{}
```

### 排列

左对齐

```csharp
  \begin{flushleft}
  	
  \end{flushleft}
```
居中

```csharp
 \begin{center}
 	
 \end{center}
```

右对齐

```csharp
  \begin{flushright}
	
  \end{flushright}
```

### 分数

```csharp
\frac{上}{下}
```

### 无序列表

```csharp
\begin{itemize}
	\item 
	\item 
\end{itemize}
```




### 根号

```csharp
\sqrt{}

```



### 插入图片

插入图片的位置不对，我们可以使用`[!htb]`来取消latex的美学，latex的美学很烂。

我们需要先引用 float

```csharp
\usepackage{graphicx}


\usepackage{float}
```

在引用图片写上

```csharp
\begin{figure}这里用取消美学[!htb]
	\includegraphics{fig1.png}
	\caption{Problem Description }
	\label{fig1}
\end{figure}
```

图片位置可以使用`figure=`

如果是esp，需要`\epsfig`

```csharp
\begin{figure}[h]
  \centerline
  {
    \epsfig
     {
      figure=images/fig, 
      height=9cm, 
      angle=-90
     }
  }
  \caption{标题}
  \label{Fig1}
\end{figure}
```

参见：http://blog.sciencenet.cn/blog-400681-886697.html

http://blog.csdn.net/bingfengxiao/article/details/6650096

多图并排

使用\vfill换行，\hfill 并排

```csharp
\begin{figure*} 
	 \centering 
	 
	 \begin{minipage}{0.48\linewidth} 0.48大小
	 	\centering 图在中间
	    \includegraphics[width=5cm,height=5cm]{f1} 图，如果使用eps需要转换
    	\caption{图1  }
        \label{fig:1}
     \end{minipage}
     \hfill 并排
     \begin{minipage}{0.5\linewidth}
     	\centering
     	\includegraphics[width=5cm,height=5cm]{f2}
        \caption{图2  }
     	\label{fig:2}
     \end{minipage}
\end{figure*}
```

每一张图有自己的`Figure`

如果需要用的是一张图包含小图

``` 
\begin{figure}
	\centering 
	\subfigure[(a)]
	{
	
		\includegraphics[width=5cm,height=3cm]{1a}
	}
	\subfigure[(b)]
	{
	
		\includegraphics[width=5cm,height=3cm]{1b}
	}
	
	\caption{ fig }
	\label{f1}
\end{figure}

```


http://blog.csdn.net/lsg32/article/details/8121417

http://www.ctex.org/documents/latex/graphics/node109.html

### 使用eps

引用epsfig

```csharp
\usepackage{graphicx}

\usepackage{epsfig} 
```

打开cmd，进入图片路径， 使用epstopdf 

```csharp
epstopdf 图.eps
```

就会生成图.pdf

不需要写后缀名

```csharp
\includegraphics[width=7cm,height=7cm]{f1}
```

把生成的pdf也放进去，这样生成就好

重新为图片编号：http://www.52yfjc.com/2014/show.asp?id=887

### Visio转eps

在线wmf转eps https://cloudconvert.com/wmf-to-eps

svg转eps http://cn.office-converter.com/SVG-to-EPS

一个简单方法是选择要转换的图，新建一个visio，然后在页面大小，设为适应，然后保存为pdf就好，一般不需要在转


### 公式

``` stylus
\begin{equation}
公式
\label{g1}
\end{equation}
```

插入公式：https://www.kancloud.cn/thinkphp/latex/41806

http://mohu.org/info/symbols/symbols.htm

http://blog.csdn.net/garfielder007/article/details/51646604

#### 公式空格

|latex|空格|
|--|--|
|a \qquad b| 两个quad空格|
|a \quad b | quad空格|
|a\ b | 大空格|
|a\;b | 中等空格|
|a\,b | 小空格|







### 插入算法

http://blog.csdn.net/lqhbupt/article/details/8723478

```csharp
\begin{algorithm}
	\caption{} 
	\label{a}
	\begin{algorithmic}
	\STATE ) $ $
	\end{algorithmic}
\end{algorithm}  
```

### 页开始

如果需要设置期刊的名

```csharp
\journalname{期刊名}
```

下载：
basic-miktex-2.9.6069-x64
<!-- https://yunpan.cn/OcvKmfknLnEsnJ  访问密码 0943 -->
链接：http://pan.baidu.com/s/1slnnoPB 密码：wcaw

Btsync:B74YC6AIP6J2CNK2CNTCXGG3NVGGMTTXO

CTeX
<!-- https://yunpan.cn/OcvKmJ8EINEN22  访问密码 7fa9 -->
http://pan.baidu.com/s/1jHB7LIm


TeXStudio
<!-- https://yunpan.cn/OcvKmVsRytDq7Z  访问密码 5ef5 -->
链接：http://pan.baidu.com/s/1pLCZL5h 密码：hb29

Btsync:BBXDB6T3LBPYJ6CVXTV7V6226FGXHXOFP

https://sourceforge.net/projects/texstudio/?source=typ_redirect

ecrc.sty
<!-- https://yunpan.cn/OcvKHgsSHJMaFc  访问密码 2cef -->
http://download.csdn.net/download/lindexi_gd/9646187


ecrc-template.tex
<!-- https://yunpan.cn/OcvKHKAiBNIZDi  访问密码 1100 -->
http://pan.baidu.com/s/1mi3CzJq

Btsync: BQ2XFET5YROHGWVN2NZNQ4X5VTKEKLO4C

参见：https://www.elsevier.com/authors/author-schemas/preparing-crc-journal-articles-with-latex

http://hubl82.blog.163.com/blog/static/1267694852013459412617/





