# C# 转换类型和字符串

有时候我们需要互转类型和字符串，把字符串转类型、把类型转字符串。


<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


<div id="toc"></div>

如果是基础类型，可以使用 x.Parse 这个方法，很多基础类型都支持。

那么我们可以使用 TypeDescriptor


```csharp
            string value = "123";
            var typeDescriptor = TypeDescriptor.GetConverter(typeof(int));
            int @int =(int) typeDescriptor.ConvertFromString(value);
            Console.WriteLine(typeDescriptor.ConvertToString(@int));
            typeDescriptor = TypeDescriptor.GetConverter(typeof(double));
            double @double = (double)typeDescriptor.ConvertFromString(value);
            Console.WriteLine(typeDescriptor.ConvertToString(@double));
```

参见：[http://www.jianshu.com/p/cdc8f5fe6405](http://www.jianshu.com/p/cdc8f5fe6405)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  