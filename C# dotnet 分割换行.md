# C# dotnet 分割换行

我在写一个 UWP 文本阅读器，我需要提升性能，需要将文本按行绘制但是文本里面的换行分割规则有点坑，本文写了一个辅助的方法用于分割换行

<!--more-->
<!-- CreateTime:2020/3/23 16:04:05 -->



虽然有默认字符串提供的 Split 分割方法很好用，在一些字符串里面只包含 `\r` 或 `\n` 很好用，但是如果在字符串里面同时包含了 `\r` 和 `\n` 就不好玩了，如下面字符串

```csharp
             var str = "123123\r123123\n123123\r\n123";
``` 

我需要按照只要有 `\r` 或 `\n` 就分割字符串，如果有连续的 `\r\n` 就分割一次

```csharp
            var newLineList = str.Split('\n', '\r').Select(text => text = text.Replace("\r", ""))
                .ToList();
```

上面代码将会多分割出一个空行，原因是 `\r\n` 被分割为两行

我自己写了一个辅助代码

```csharp
        private static List<string> SplitMultiLines(string str)
        {
            var lineList = new List<string>();
            var text = new StringBuilder(str.Length);
            for (var i = 0; i < str.Length; i++)
            {
                var c = str[i];
                if (c == '\r')
                {
                    lineList.Add(text.ToString());
                    text.Clear();

                    if (i < str.Length - 1)
                    {
                        if (str[i + 1] == '\n')
                        {
                            i++;
                        }
                    }
                }
                else if (c == '\n')
                {
                    lineList.Add(text.ToString());
                    text.Clear();

                    if (i < str.Length - 1)
                    {
                        if (str[i + 1] == '\r')
                        {
                            i++;
                        }
                    }
                }
                else
                {
                    text.Append(c);
                }
            }

            lineList.Add(text.ToString());
            return lineList;
        }
```

上面代码能符合我预期，如果小伙伴有更简单的方法，欢迎告诉我

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/0495ca07ac65af548810035628a4d565b26f1c91/BepirquwiKedoucawji) 欢迎小伙伴访问

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
