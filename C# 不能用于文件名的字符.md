# C# 不能用于文件名的字符

在 Windows 有一些字符是不能作为文件名，尝试重命名一个文件，输入`/` 就可以看到windows 提示的不能作为文件名的字符

<!--more-->
<!-- CreateTime:2018/8/10 19:16:52 -->


<div id="toc"></div>

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018222143910.jpg)

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

之外还有一些文件名是保留，不能创建这样的文件名

```csharp
CON, PRN, AUX, CLOCK$, NUL
COM0, COM1, COM2, COM3, COM4, COM5, COM6, COM7, COM8, COM9
LPT0, LPT1, LPT2, LPT3, LPT4, LPT5, LPT6, LPT7, LPT8, and LPT9.
```

尝试新建一个文本，然后把他文件名命名为上面的任意一个，基本windows会说不能把文件命名

那么是不是把这些字符串拿出来判断？实际上微软已经做了这个了，因为在不同的系统，可能之后会添加新的字符串，所以最好不要自己写。

可以使用微软给的函数`System.IO.Path.GetInvalidFileNameChars` ，下面的代码可以直接使用

```csharp
        public static string MakeValidFileName(string text, string replacement = "_")
        {
            StringBuilder str=new StringBuilder();
            var invalidFileNameChars = System.IO.Path.GetInvalidFileNameChars();
            foreach (var c in text)
            {
                if (invalidFileNameChars.Contains(c))
                {
                    str.Append(replacement??"");
                }
                else
                {
                    str.Append(c);
                }
            }

            return str.ToString();
        }

```

如果确实需要显示文件名，请使用下面代码

```csharp
public static string GetSafeFilename(string arbitraryString)
{
    var invalidChars = System.IO.Path.GetInvalidFileNameChars();
    var replaceIndex = arbitraryString.IndexOfAny(invalidChars, 0);
    if (replaceIndex == -1) return arbitraryString;

    var r = new StringBuilder();
    var i = 0;

    do
    {
        r.Append(arbitraryString, i, replaceIndex - i);

        switch (arbitraryString[replaceIndex])
        {
            case '"':
                r.Append("''");
                break;
            case '<':
                r.Append('\u02c2'); // '˂' (modifier letter left arrowhead)
                break;
            case '>':
                r.Append('\u02c3'); // '˃' (modifier letter right arrowhead)
                break;
            case '|':
                r.Append('\u2223'); // '∣' (divides)
                break;
            case ':':
                r.Append('-');
                break;
            case '*':
                r.Append('\u2217'); // '∗' (asterisk operator)
                break;
            case '\\':
            case '/':
                r.Append('\u2044'); // '⁄' (fraction slash)
                break;
            case '\0':
            case '\f':
            case '?':
                break;
            case '\t':
            case '\n':
            case '\r':
            case '\v':
                r.Append(' ');
                break;
            default:
                r.Append('_');
                break;
        }

        i = replaceIndex + 1;
        replaceIndex = arbitraryString.IndexOfAny(invalidChars, i);
    } while (replaceIndex != -1);

    r.Append(arbitraryString, i, arbitraryString.Length - i);

    return r.ToString();
}
```

上面的代码是[jnm2](https://stackoverflow.com/a/30126118/6116637) 大神写的

参见：[Information about the characters that you cannot use in site names, folder names, and file names in SharePoint](https://support.microsoft.com/en-us/help/905231/information-about-the-characters-that-you-cannot-use-in-site-names-fol )

[c# - How to make a valid Windows filename from an arbitrary string](https://stackoverflow.com/questions/620605/how-to-make-a-valid-windows-filename-from-an-arbitrary-string )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
