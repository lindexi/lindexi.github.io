# sublime Text 正则替换

<!--more-->
<!-- CreateTime:2018/8/10 19:16:52 -->


<div id="toc"></div>

我遇到一个文章，需要把所有的 (数字)  换为 [数字]

于是我使用 Sublime Text的替换

首先，我们需要打开正则使用“Alt+R” 或打开“Ctrl+h”选择正则。

然后我们开始输入正则，“ `\((\d+\)` ” 我们需要拿出的是数字，所有在数字加“()”。于是在替换写“`\[$1\]`”，其中`$0`就是所有的 `$1`就是第一个括号。

如何使用正则可以去看[正则表达入门](https://blog.lindexi.com/post/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F30%E5%88%86%E9%92%9F%E5%85%A5%E9%97%A8%E6%95%99%E7%A8%8B.html)。

Sumlime 还可以创建代码行，做法也很简单。

点击 Tools   New Snippet 


![](http://image.acmx.xyz/d021ae55-501f-4838-a9a0-f09ee95a83b82016121992723.jpg)


```xml
< snippet>
	< content><![CDATA[
Hello, ${1:this} is a ${2:snippet}.
]]>< /content>
	<!-- Optional: Set a tabTrigger to define how to trigger the snippet -->
	<!-- <tabTrigger>hello</tabTrigger> -->
	<!-- Optional: Set a scope to limit where the snippet will trigger -->
	<!-- <scope>source.python</scope> -->
< /snippet>

```

content 是我们按下快捷键的内容，\$ {1:this} 就是第一个输入内容，其中，默认写This，所有的{1}都代换你输入的第一个。\$2就是第二个。

我们需要设置快捷键。

`<tabTrigger>hello</tabTrigger>`

就是按下 hello，按下 tab 就会使用代码段。

写好，我们保存在`C:\Users\<Use>\AppData\Roaming\Sublime Text 2\Packages\User` 后缀`.sublime-snippet`


我们有时打开中文会乱码，我们可以 ctrl+shift+p

输入 Package  control:install 安装 CovertToUTF8


<alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 