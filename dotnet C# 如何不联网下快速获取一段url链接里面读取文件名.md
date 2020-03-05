# dotnet C# 如何不联网下快速获取一段url链接里面读取文件名

在不联网情况下，从 url 获取文件名也就是只能靠 url 本身了，如果是 get 的链接很多链接后面都是带上了文件名，那么如何用最短的代码获取链接里面的文件名

<!--more-->
<!-- CreateTime:2020/2/16 14:44:30 -->

<!-- 发布 -->

如我有链接 https://blog.lindexi.com/lindexi.html 我想要拿到 lindexi.html 这个文件名可以如何写代码最少

请看下面方法

```csharp
        private string GetFileName(string url)
        {
            FileInfo fileInfo = new FileInfo(url);
            return fileInfo.Name;
        }
```

是不是看错代码了，居然用了 FileInfo 的方法，小伙伴请试试用下面代码跑起来

```csharp
using System;
using System.IO;

        static void Main(string[] args)
        {
            Program program = new Program();
            Console.WriteLine(program.GetFileName("https://blog.lindexi.com/lindexi.html"));
        }
```

此时可以看到输出了 lindexi.html 就这样返回了 url 链接里面的文件名，那么还有更短的方法

```csharp
        private string GetFileName(string url)
        {
            return System.IO.Path.GetFileName(url);
        }
```

没错用 System.IO.Path.GetFileName 就可以了，不需要做任何的字符串修改，直接一个库方法就解决了

在个人的小项目或工具项目，或脚本里面，可以使用上面方法解析 url 的文件名

我可以做到不开 vs 不用任何 ide 只需要记事本加上命令行就完成了上面的玩法

```
dotnet new console -o KugaybafalJerekunaycerecha
cd KugaybafalJerekunaycerecha
notepad Program.cs

>using System;
>using System.IO;
>
>namespace KugaybafalJerekunaycerecha
>{
>    class Program
>    {
>        static void Main(string[] args)
>        {
>            Console.WriteLine(Path.GetFileName("https://blog.lindexi.com/lindexi.html"));
>        }
>    }
>}

保存
dotnet run
```

我建议在设备上面安装 dotnet 工具，从[官网 https://dotnet.microsoft.com/](https://dotnet.microsoft.com/)下载安装 dotnet 工具，就可以在命令行玩起来，在熟悉之后可以提高逗比技能

等等还有一个问题，如果添加了参数呢，如 https://blog.lindexi.com/doubi/lindexi.html?13216546 上面代码返回的是什么？

试试下面代码

```csharp
        private string GetFileName(string url)
        {
            Uri uri = new Uri(url);
            return System.IO.Path.GetFileName(uri.AbsolutePath);
        }
```

可以看到上面的 AbsolutePath 就可以去掉了链接参数，不过用 uri 需要小心一些问题，请看 [C#/.NET Framework 使用 Uri 类型表示文件路径时需要注意的问题 - J.晒太阳的猫 - 博客园](https://www.cnblogs.com/jasongrass/p/12284564.html )

等等，我拿到的链接里面如果有中文名呢，请看 [win10 uwp url encode](https://blog.lindexi.com/post/win10-uwp-url-encode.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
