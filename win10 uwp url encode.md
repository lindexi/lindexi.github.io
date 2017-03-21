# win10 uwp url encode

有时候需要向网络传一些中文或其他不支持的东西，这时需要 url encode

有时候需要把`<` 转`&lt;` ，这些本文都会告诉你，如何转换

<!--more-->
<!-- csdn -->

可以使用的函数有三个

`Uri.EscapeDataString` `WebUtility.UrlEncode` `WebUtility.HtmlEncode`

对于中文，`Uri.EscapeDataString` `WebUtility.UrlEncode`转换得到是一样。


对于符号，`Uri.EscapeDataString` `WebUtility.UrlEncode` 有一点不同。


`WebUtility.HtmlEncode` 做的转换很少，是将html源文件中不容许出现的字符进行编码

`Uri.EscapeDataString` `WebUtility.UrlEncode` `WebUtility.HtmlEncode` 对应的是
`Uri.UnescapeDataString` `WebUtility.UrlDecode` `WebUtility.HtmlDecode`

对字符串：“~+ =!@$#^&*http://lindexi.oschina.io”

通过 Uri.EscapeUriString 返回 “~+%20=!@$#%5E&*http://lindexi.oschina.io”

  
WebUtility.UrlEncode 返回  “%7E%2B+%3D!%40%24%23%5E%26*http%3A%2F%2Flindexi.oschina.io”

他们都是标准，关于区别，请看 http://stackoverflow.com/a/11236038/6116637

对于中文，使用两个得到是一样

WebUtility.HtmlEncode 可以转 `&#` 。

如果希望继续看三个转换有哪些不同，请看下面


|　| Uri.EscapeUriString|Uri.EscapeDataString|WebUtility.UrlEncode|WebUtility.HtmlEncode|
|--|--|--|--|
| 空格 |%20|%20 |+ | 空格|
|! |!|%21 |! |!|
|+ |+|%2B |%2B |+|
|" |%22|%22 |%22 |&quot;|
|# |#|%23 |%23 |#|
|$ |$|%24 |%24 |$|
|* |*|%2A |* |*|
|( |(|%28 |( |(|
|) |)|%29 |) |)|
|~ |~|~ |%7E |~|
|1 |1|1 |1 |1|
|2 |2|2 |2 |2|
|3 |3|3 |3 |3|
|a |a|a |a |a|
|b |b|b |b |b|
|: |:|%3A |%3A |:|
|' |'|%27 |%27 |&#39;|
|< |%3C|%3C |%3C |&lt;|
|> |%3E|%3E |%3E |&gt;|
|[ |[|%5B |%5B |[|
|] |]|%5D |%5D |]|
|/ |/|%2F |%2F |/|
|{ |%7B|%7B |%7B |{|
|} |%7D|%7D |%7D |}|
|\| |%7C|%7C |%7C |\||
|, |,|%2C |%2C |,|
|? |?|%3F |%3F |?|
|中 |%E4%B8%AD|%E4%B8%AD |%E4%B8%AD |中|
|文 |%E6%96%87|%E6%96%87 |%E6%96%87 |文|

https://blogs.msdn.microsoft.com/yangxind/2006/11/08/dont-use-net-system-uri-unescapedatastring-in-url-decoding/



<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  