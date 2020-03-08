# C# 搜索算法

本文主要讲C#搜索算法。

<!--more-->
<!-- CreateTime:2019/12/24 9:27:49 -->


<div id="toc"></div>

<!--cdsn-->

## Bdf 算法

这算法是一个模糊的算法，用在用户在找一个他不确定的文本。

判断文本和匹配的字符是否有相同顺序，如果有，那么就是匹配。

假如我们有数据“abc”，匹配“abc”，那么，两个完全相对的字符串是匹配。

数据“aaacbc”，匹配“abc”，也是匹配，因为数据按照“abc”的顺序，算法不管数据有多长，只要数据存在和匹配相同的顺序，那么就匹配。

```csharp
        /// <summary>
        /// 
        /// </summary>
        /// <param name="text">数据</param>
        /// <param name="str">匹配</param>
        public static bool Bdt(string text, string str)
        {
            int i = 0;
            bool reu = false;
            foreach (var temp in str)
            {
                reu = false;
                for (; i < text.Length; i++)
                {
                    if (temp == text[i])
                    {
                        reu = true;
                        break;
                    }
                }
            }
            return reu;
        }

```


现在算法用在 https://github.com/tpxxn/JiHuangBaiKeCSharp

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  