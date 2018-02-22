# C# 不能用于文件名的字符

在 Windows 有一些字符是不能作为文件名，尝试重命名一个文件，输入`/` 就可以看到windows 提示的不能作为文件名的字符

<!--more-->
<!-- csdn -->

<div id="toc"></div>

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018222143910.jpg)

那么具体是包括哪些符号不能作为文件名？

 - Tilde (`~`)
 - Number sign (`#`)
 - Percent (`%`)
 - Ampersand (`&`)
 - Asterisk (`*`)
 - Braces (`{` `}`)
 - Backslash (\\)
 - Colon (`:`)
 - Angle brackets (`<` `>`)
 - Question mark (`?`)
 - Slash (`/`)
 - Plus sign (`+`)
 - Pipe (`|`)
 - Quotation mark (`"`)

上面这些字符不能用来文件名和文件夹名

而且在 windows 的要求，文件是需要有文件名的，虽然你也可以创建`.file`这样的文件，但是用户是难以自己输入这样的文件名。

参见：[Information about the characters that you cannot use in site names, folder names, and file names in SharePoint](https://support.microsoft.com/en-us/help/905231/information-about-the-characters-that-you-cannot-use-in-site-names-fol )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
