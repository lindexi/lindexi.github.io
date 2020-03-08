# C# AddRange 添加位置

有没人想知道， AddRange 添加位置 是哪？

是添加到数组的开始，还是数组的末尾？

<!--more-->
<!-- CreateTime:2018/10/19 9:03:08 -->


<div id="toc"></div>

假如有一个 代码，看起来是下面的，很简单，把一个 list b 放进list a


```csharp
                List<int> a=new List<int>(){1,2,};
            List<int> b = new List<int> {5, 6};
            a.AddRange(b);
            foreach (var temp in a)
            {
                Console.WriteLine(temp);
            }
```

那么打印的是 1 2  5 6

还是 5 6 1 2

答案是自己去跑。

不要打我，答案是 1256， AddRange 添加数组的末尾。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017917111552.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 