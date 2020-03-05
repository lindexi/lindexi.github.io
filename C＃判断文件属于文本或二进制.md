
# C＃判断文件属于文本或二进制

其实标题说错了，所有的文件储存都是二进制，但我们想区别的是那些可以当做文本来读的，如 .txt,.cs,.c 的文件和一般的不是文字的文件。
我们有什么办法可区分文件是文本或二进制？

<!--more-->



<div id="toc"></div>

于是我找到下面的文章，发现了好多个方法。

http://stackoverflow.com/questions/567757/how-do-i-distinguish-between-binary-and-text-files

http://magic.codeplex.com/

https://stackoverflow.com/questions/4744890/c-sharp-check-if-file-is-text-based

https://stackoverflow.com/questions/910873/how-can-i-determine-if-a-file-is-binary-or-text-in-c

http://blog.csdn.net/cherylnatsu/article/details/6412898

最后发现Git的方法是判断一个文件中是否存在 '\0' 如果存在，那么判断为二进制，不是文本，当然对于 Utf-16 这个方法容易就炸了，显然没有一个好用的方法。

我在项目：https://github.com/lindexi/EncodingNormalior   遇到这个问题，我希望让用户自己添加规则，如果遇到规则之外的，那么判断使用上面的检测 '\0' 方法，当然，遇到了 Utf-16 编码，还是没有发现好的解决办法。


另外还发现一个判断文件的方法是读文件的前两个字节，但是这个方法很多都无法判断。

[在C#中如何确定一个文件是不是文本文件，以及如何确定一个文件的类型](https://fresky.github.io/2014/04/21/how-to-determine-the-file-type-in-csharp/)




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。