# C＃ 6.0 字符串 String Interpolation

本文主要：C# 6.0 新特性 String Interpolation，一些比较少知道的知识。

本文内容有：字符串新特性的一般使用、格式化字符串、保留小数点、判断字符串


<!--more-->
<!-- CreateTime:2018/8/10 19:16:52 -->


<div id="toc"></div>

原文发在csdn http://blog.csdn.net/lindexi_gd/article/details/49716741

本文做了一些修改。

"hello $world"的格式化字符串是指把字符串中一个单词，以一个标示开头。可以代换为单词所指的变量。
这个在jq有，而C#string的格式只能用格式的字符占位符，格式的字符占位符都是数字，这样多了很容易混，好多我都出现了，拷贝代码，然后没有排好数字，漏了一个数字，这样出现了错误。
`string.Format("这里有很长字符串{0}{2}" , "Hello" , "csdn");`
![string.Format("这里有很长字符串{0}{2}" , "Hello" , "csdn");](http://img.blog.csdn.net/20151108094949397)
而看到一个大神实现了类似jQueryStringFormat的扩展string，觉得C#内置有一个方法：

```C#
            var csdn = "csdn";
            var result = $"Hello {csdn}";
            Console.Write(result);
```
会输出![输出Hello csdn](http://img.blog.csdn.net/20151108095202929)

通过$开头字符串，中间`{}`作为变量名，可以把字符串代换为变量的字符。

这就是 C# 6.0 新特性： String Interpolation

但是 String Interpolation 如何输入一定长度的字符串？


我看到了堆栈炸有大神问了一个[问题](http://stackoverflow.com/questions/37113595/c-sharp-6-how-to-format-double-using-interpolated-string)，刚好我在做的编码[工具](https://github.com/iip-easi/EncodingNormalior)也遇到命令行输入的不好看，需要格式化，所以就去找下方法。

后来发现，可以在ToString放参数的，把参数写在`:`后就可以传进去。


```csharp
        static void Main(string[] args)
        {
            string csdn = "csdn";
            double n = 1.1315;
            string str = $"Hello {csdn} 新特性";
            Console.WriteLine(str);

            str = $"Hello {csdn} 新特性 {n}";
            Console.WriteLine(str);

            str = $"Hello {csdn} 新特性 {n:1.##}";
            Console.WriteLine(str);

            //不生效
            str = $"Hello {csdn:10} 新特性 {n:1.##}";
            Console.WriteLine(str);
        }
```

如果要指定一个字符串显示的长度，不够长度填充空格，使用`,`


```csharp
            str = $"Hello {csdn,10} 新特性 {n:1.##}";
            Console.WriteLine(str);
```

![](http://image.acmx.xyz/8f464be7-2358-45f4-b6cd-eae32c47a87820172715475.jpg)

关于字符串长度的使用，参见：https://msdn.microsoft.com/en-us/library/dn961160.aspx


可以看到上面的长度是添加在左边，如果要添加在右边？可以使用字符串函数


```csharp
            str = $"Hello {csdn.PadRight(10)} 新特性 {n:1.##}";
            Console.WriteLine(str);
```


当然，可以在字符串使用判断。


```csharp
            str = $"Hello {(csdn =="csdn"?"csdn":"lindexi")} 新特性 {n:1.##}";
            Console.WriteLine(str);
```




代码传到了 csdn 下载：http://download.csdn.net/detail/lindexi_gd/9749065

还有代码放在 code.csdn

代码：https://code.csdn.net/lindexi_gd/lindexi_gd/tree/master/hellow_$csdn


参考：http://www.cnblogs.com/isaboy/p/4945045.html 

更多特性参见 [http://www.cnblogs.com/wolf-sun/p/5168217.html](http://www.cnblogs.com/wolf-sun/p/5168217.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  