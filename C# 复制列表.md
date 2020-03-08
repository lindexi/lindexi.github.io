# C# 复制列表

本文：如何复制一个列表

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

最简单的方法是 foreach


```csharp
   foreach(var temp in a)
   {
       b.Add(temp);

   }
```

有没一个简单的方法？


```csharp
            using System.Linq;

            var a = new List<Fex>()
            {
                new Fex() {F = true,},
                new Fex() {F = true,},
                new Fex() {F = false,},

            };
            List<Fex> b = a.ToList();

            b.RemoveAt(0);

            Console.WriteLine(a.Count);

            b.Add(new Fex());
            b.Add(new Fex());

            Console.WriteLine(a.Count);
```

            
`List<Fex> b = a.ToList();` 可以把列表a到列表b，对b进行删除、添加，不会对a造成元素改变。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  