# C＃ 通配符转正则


<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

可以使用下面代码把通配符转正则字符串

```csharp
    public static class WildcardRegexString
    {
        /// <summary>
        /// 通配符转正则
        /// </summary>
        /// <param name="wildcardStr"></param>
        /// <returns></returns>
        public static string GetWildcardRegexString(string wildcardStr)
        {
            Regex replace = new Regex("[.$^{\\[(|)*+?\\\\]");
            return replace.Replace(wildcardStr,
                       delegate (Match m)
                       {
                           switch (m.Value)
                           {
                               case "?":
                                   return ".?";
                               case "*":
                                   return ".*";
                               default:
                                   return "\\" + m.Value;
                           }
                       }) + "$";
        }
    }
```

文件经常是不需要区分大小写，所以需要写一个函数告诉用户，不需要区分大小写。


```csharp
        /// <summary>
        /// 获取通配符的正则
        /// </summary>
        /// <param name="wildcarStr"></param>
        /// <param name="ignoreCase">是否忽略大小写</param>
        /// <returns></returns>
        public static Regex GetWildcardRegex(string wildcarStr, bool ignoreCase)
        {
            if (ignoreCase)
            {
                return new Regex(GetWildcardRegexString(wildcarStr));
            }
            return new Regex(GetWildcardRegexString(wildcarStr), RegexOptions.IgnoreCase);
        }
```

正则可以使用程序集方式，启动慢，但是运行快


```csharp
          private static Regex _regex = new Regex("[.$^{\\[(|)*+?\\\\]", RegexOptions.Compiled);
```

我的软件就需要重复使用，于是就使用这个。

代码：

<script src="https://gist.github.com/lindexi/2bd3bccb6de194aa40ad2e09a5413000.js"></script>

[https://gist.github.com/lindexi/2bd3bccb6de194aa40ad2e09a5413000](https://gist.github.com/lindexi/2bd3bccb6de194aa40ad2e09a5413000)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  