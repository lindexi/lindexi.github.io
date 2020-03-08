# Resharper 如何把类里的类移动到其他文件

有时候，看到一个类里有很多类，需要把他移动其他文件

<!--more-->
<!-- CreateTime:2018/8/14 17:34:39 -->


<div id="toc"></div>

<!-- 标签：Resharper -->

假如有一个类


```csharp
    class A
    {
        class B
        {

        }
       
    }
```

如何把 B 移动文件 B里？

一般使用 快捷键是 Resharper 的快捷键，如果不是的话，打开设置选择快捷键是 Resharper

然后选择 B ，按 ctrl+shift+R

![](http://image.acmx.xyz/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F201732420813.jpg)

移动到其他文件，第一个

这样输入文件名称就可以移动类到其他文件

这个快捷键可以把类移到其他命名空间，安全删除，提取属性做接口

如果需要Resharper多行注释，请用`ctrl+shift+/`

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。