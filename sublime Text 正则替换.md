# sublime Text 正则替换

<!--more-->

我遇到一个文章，需要把所有的 (数字)  换为 [数字]

于是我使用Sublime Text的替换

首先，我们需要打开正则使用“Alt+R” 或打开“Ctrl+h”选择正则。

然后我们开始输入正则，“ \((\d+\) ” 我们需要拿出的是数字，所有在数字加“()”。于是在替换写“\[$1\]”，其中$0就是所有的，$1就是第一个括号。

如何使用正则可以去看[正则表达入门](http://lindexi.oschina.io/lindexi/post/正则表达式30分钟入门教程/)。

Sumlime 还可以创建代码行，做法也很简单。

点击Tools   New Snippet 


![](http://7xqpl8.com1.z0.glb.clouddn.com/d021ae55-501f-4838-a9a0-f09ee95a83b82016121992723.jpg)


```
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

content是我们按下快捷键的内容，\$ {1:this} 就是第一个输入内容，其中，默认写This，所有的{1}都代换你输入的第一个。\$2就是第二个。

我们需要设置快捷键。

`<tabTrigger>hello</tabTrigger>`

就是按下hello，按下tab就会使用代码段。

写好，我们保存在`C:\Users\<Use>\AppData\Roaming\Sublime Text 2\Packages\User` 后缀`.sublime-snippet`