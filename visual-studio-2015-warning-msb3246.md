# visual studio 2015 warning MSB3246

在我们很高兴的按下 本地计算机运行 按钮，希望看到我们程序运行的时候，垃圾vs就告诉我们，你的程序出现了问题，问题就是：

warning MSB3246: 解析的文件包含错误图像、无元数据或不可访问。未能加载文件或程序集“`*.dll`”或它的某一个依赖项。试图加载格式不正确的程序。

其中`*.dll`就是一个dll名称，那么遇到这个问题，不是修复vs就能做好的，我找了好久，在堆栈炸了找到一个可以用的方法
<!--more-->
<!-- CreateTime:2019/9/2 12:57:38 -->


<div id="toc"></div>

1. 检查`.nuget\packages`的包，假如报错的`*.dll`是`System.Numerics.Vectors.WindowsRuntime.dll`那么找到`System.Numerics.Vectors.WindowsRuntime`文件夹

 ![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=5d6484b1ed6be44a36183ba6eca782db)
 `.nuget\packages`在`C:\Users\用户名\`

2. 尝试修改找到的文件夹名称，或移动到别的地方
可能在运行，有程序占用文件，这时可以使用软媒的文件大师取消占用。

 修改名称的做法是让vs找不到文件夹，然后重新生成，如果生成失败，我们可以通过把文件夹名称改回去，快速恢复。

3. 选择项目清理项目，然后生成项目
 可以看到nuget安装被删掉的文件夹
 ![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=d9233c9b6583d2543378ae15fbea5bc4)



<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。





