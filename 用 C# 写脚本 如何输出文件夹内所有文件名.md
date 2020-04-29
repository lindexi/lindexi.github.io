# 用 C# 写脚本 如何输出文件夹内所有文件名

大部分在 Windows 下的脚本都是使用 bat 或 cmd 写的，这部分的脚本对我来说可读性不好。这个可读性也是很主观的，对我来说用 C# 写脚本的可读性很强，但是换个小伙伴就不是了。在 .NET Core 下的 C# 可以通过 dotnet run 运行代码起来，此时特别适合用来写脚本

<!--more-->
<!-- CreateTime:4/28/2020 8:06:43 AM -->

<!-- 发布 -->

我需要输出一个文件夹里面的根目录的所有文件，输出一个文件夹的顶层文件的方法可以使用 Directory.GetFiles 拿到文件夹的顶层文件，然后遍历输出。此时注意需要引用 System.IO 命名空间

```csharp
using System.IO;

        static void Main(string[] args)
        {
            var folder = @"e:\lindexi\";
            foreach (var file in Directory.GetFiles(folder))
            {
                Console.WriteLine(file);
            }

            Console.Read();
        }
```

这就是整个脚本的核心代码了，十分简单，通过 dotnet run 命令就可以跑起来

额外的，在 Directory.GetFiles 可以输入通配符进行选择输出的文件

```csharp
            foreach (var file in Directory.GetFiles(folder, "*.enbx"))
            {
                Console.WriteLine(file);
            }
```

如上面代码就输出文件夹里面后缀是 `.enbx` 的文件

如果想要遍历所有的子文件夹的文件，可以通过加上 SearchOption.AllDirectories 参数

```csharp
            foreach (var file in Directory.GetFiles(folder, "*.enbx", SearchOption.AllDirectories))
            {
                Console.WriteLine(file);
            }
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/c4dd7a59cd2c45b5ca0d53438964ac9af0d439d1/BerjearnearheliCallrachurjallhelur) 欢迎小伙伴访问


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
