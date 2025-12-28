# jekyll 在博客添加流程图

本文告诉大家如何在博客使用流程图。

<!--more-->
<!-- CreateTime:2019/8/31 16:55:59 -->


如果你使用的是我博客的模板，那么就可以直接使用我说的文件，如果是自己的主题，就需要在自己文件对应的地方加上代码。

在我的博客里，需要添加下面的js到博客，可以打开 js.html 添加下面的代码。如果是自己定义的主题，需要在博客可以访问的地方添加下面的代码

```
<script src="//cdn.bootcss.com/raphael/2.2.0/raphael-min.js"></script>
<script src="//cdn.bootcss.com/flowchart/1.6.3/flowchart.js"></script>

<script>
	function flow(name,f)
	{
                    var chart = flowchart.parse(f);
                    chart.drawSVG(name, 
                    {
                       'x': 30,
                       'y': 50,
                      'line-width': 3,
                      'maxWidth': 3,//ensures the flowcharts fits within a certian width
                      'line-length': 50,
                      'text-margin': 10,
                      'font-size': 14,
                      'font': 'normal',
                      'font-family': 'Helvetica',
                      'font-weight': 'normal',
                      'font-color': 'black',
                      'line-color': 'black',
                      'element-color': 'black',
                      'fill': 'white',
                      'yes-text': 'yes',
                      'no-text': 'no',
                      'arrow-end': 'block',
                      'scale': 1,
                      'symbols': {
                        'start': {
                          'font-color': 'red',
                          'element-color': 'green',
                          'fill': 'yellow'
                        },
                        'end':{
                          'class': 'end-element'
                        }
                      },
                      'flowstate' : {
                        'past' : { 'fill' : '#CCCCCC', 'font-size' : 12},
                        'current' : {'fill' : 'yellow', 'font-color' : 'red', 'font-weight' : 'bold'},
                        'future' : { 'fill' : '#FFFF99'},
                        'request' : { 'fill' : 'blue'},
                        'invalid': {'fill' : '#444444'},
                        'approved' : { 'fill' : '#58C4A3', 'font-size' : 12, 'yes-text' : 'APPROVED', 'no-text' : 'n/a' },
                        'rejected' : { 'fill' : '#C45879', 'font-size' : 12, 'yes-text' : 'n/a', 'no-text' : 'REJECTED' }
                      }
                  });
	}

	window.onload = function () 
	{
		var cd = document.getElementsByClassName("flow");
		for (var i = 0; i < cd.length; i++) 
		{
			var t = cd[i].getElementsByClassName("flowcode")[0].value;
			var canvas = "canvas" + i;
			cd[i].innerHTML = "<div id=\"" + canvas + "\"></div>"
			flow(canvas, t);
		}
        
	}


</script>
```

然后就可以在博客使用流程图啦。

流程图使用的使用需要先定义，然后使用`->`表示连接

可以使用的定义有下面几个

 - start		开始
 - end			结束
 - operation	方法
 - subroutine	子程序
 - condition	条件 
 - inputoutput	输入

使用流程图和使用代码一样，需要使用`<div class="flow">` 放在一个地方，然后写 `flow` 请看下面代码

```csharp
 <div class="flow">
<textarea class="flowcode">

</textarea>
</div>
``` 

例如写一个简单的流程，注意把前面的空格删掉

```csharp
<div class="flow">
<textarea class="flowcode">
st=>start: Start 
e=>end           
ldata=>operation: 进入csdn 

st->ldata->e 
</textarea>
</div>
```

<div class="flow">
<textarea class="flowcode">
st=>start: Start 
e=>end           
ldata=>operation: 进入csdn
st->ldata->e 
</textarea>
</div>



`condition`条件需要添加是否条件，例如下面的代码


```csharp
<div class="flow">
<textarea class="flowcode">
   st=>start: Start
   e=>end
   ldata=>operation: 进入csdn
   c=>condition: 是否进入lindexi_gd
   l=>operation: 访问
   st->ldata->c
   c(yes)->l->e
   c(no)->e
</textarea>
</div>
 

```

必须删除空格才可以使用代码

<div class="flow">
<textarea class="flowcode">
 
st=>start: Start
e=>end
ldata=>operation: 进入csdn
c=>condition: 是否进入lindexi_gd
l=>operation: 访问
st->ldata->c
c(yes)->l->e
c(no)->e
</textarea>
</div>

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 