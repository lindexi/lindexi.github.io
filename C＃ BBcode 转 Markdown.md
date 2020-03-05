# C＃ BBcode 转 Markdown

本文告诉大家一个简单的方法从 BBcode 转为 Markdown

<!--more-->
<!-- CreateTime:2018/5/20 14:58:57 -->


本文的方法都是使用正则转换，现在支持的代码只有很少的常用标签，如果大家发现有转换失败的，请帮我修改代码，估计代码我不会进行修改。

最重要的就是转换 url 和 image，那么主要就告诉大家如何转换这两个

```csharp
        private static string ConvertUrl(string str)
        {
            var regex = new Regex(@"\[url=(.+?)\]((?:.|\n)+?)\[\/url\]");
            return regex.Replace(str, "[$2]($1)");
        }
```

这就是转换 url 的代码，里面用了正则。一般使用正则拿到的数据就是`(`里的数据，如上面的代码，可能看起来有些复杂，那么用下面的代码告诉大家。

例如 需要拿出 123lindexifoo中的 lindexi  ，那么正则可以这样写

```csharp
123(lindexi)foo
```

如果需要拿到 lindexi ，可以使用下面代码

```csharp
           var regex = new Regex(@"123(lindexi)foo");
            if (regex.Match(str).Groups[1].Value == "lindexi")
            {
                
            }
```

实际使用会在 regex.Match 之后判断是否成功，而不是直接拿出来。从上面代码可以看到我使用了`1`而不是0，那么下面的代码拿到的字符串？

```csharp
            var regex = new Regex(@"123(lindexi)foo");
            str = regex.Match(str).Groups[0].Value;
```

这个 str 拿到是 123lindexifoo 就是原来匹配到的所有字符串。

如果需要替换某个字符串，可以使用下面的代码

```csharp
            str = "123lindexifoo";
            var regex = new Regex(@"123(lindexi)foo");
            str = regex.Replace(str, "csdn");

            str == "csdn"
```

如果想把上面的字符串替换为 lindexi csdn ，那么可以使用下面代码

```csharp
         str = "123lindexifoo";
            var regex = new Regex(@"123(lindexi)foo");
            str = regex.Replace(str, "$1csdn");
```

可以看到上面的替换使用了 `$1` ，这个就是第一个匹配拿到的字符串。如果需要拿第二个，就是使用`$2`，所以做这个很简单

从上面的代码可以看到，转换 image 可以使用代码

```csharp
            var regex = new Regex(@"\[img\]((?:.|\n)+?)\[\/img\]");
            return regex.Replace(str, "![$1]($1)");
```

转换其他的代码上传到 github ，如果是在我博客可以直接看到

<script src="https://gist.github.com/lindexi/3ba74c909b23d6426c779d0a8eb435c3.js"></script>

![](https://i.loli.net/2018/04/08/5ac9ffc7abb33.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
