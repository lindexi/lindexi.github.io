# VisualStudio 给任意字符串给定正则等格式

在写正则或 json 等字符串的时候，期望有智能提示和语法纠错，在 VisualStudio 可以在字符串上面添加一句注释表示这个字符串的功能是什么，然后 VisualStudio 对下一句字符串定义的代码将会执行智能提示，本文告诉大家可以如何给任意的字符串添加智能提示

<!--more-->
<!-- csdn -->

请试试使用 VisualStudio 2019 新建一个控制台项目，无论是 dotnet core 的项目还是 dotnet framework 的项目都可以

请尝试输入下面代码

```csharp
            // lang=json
            var str = "{ \"a\":\"\" }";
```

此时可以看到字符串高亮

<!-- ![](image/VisualStudio 给任意字符串给定正则等格式/VisualStudio 给任意字符串给定正则等格式0.png) -->

![](http://image.acmx.xyz/lindexi%2F2019912112531543)

在 VisualStudio 里面有这样的功能，在字符串定义的上面写上 `// lang=xx` 将会表示字符串格式

例如定义正则字符串

```csharp
            // lang=regex
            var str = @"\s";
```

不仅有高亮同时会有正则智能提示

<!-- ![](image/VisualStudio 给任意字符串给定正则等格式/VisualStudio 给任意字符串给定正则等格式1.png) -->

![](http://image.acmx.xyz/lindexi%2F2019912112717359)

除了在字符串定义的上一行写，还可以写在字符串前面

```csharp
            var str = /*lang=regex*/ @"\s";
```

上面这个写法更可读

如果有小伙伴不知道这个注释是做什么的，在代码审查可能会问你，可以在注释告诉代码审查小伙伴这个注释的用法

```csharp
var str = /*lang=regex 开启智能提示*/ @"\s";
```

将 `lang` 修改为 `language` 也是可以的

除了 json 和 regex 之外，还支持 xml 和 html 格式

[Code Analysis and Helpers for String Literals - Help ](https://www.jetbrains.com/help/resharper/Code_Analysis__String_Formatting_Methods.html )

[Visual Studio 2019 Preview .NET Productivity](https://devblogs.microsoft.com/dotnet/visual-studio-2019-net-productivity/ )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
