# C# 判断系统版本

本文告诉大家如何判断系统是 win7 还是 xp 系统

<!--more-->
<!-- CreateTime:2019/10/18 15:02:00 -->


使用下面代码可以判断

```csharp
        private static readonly Version _osVersion = Environment.OSVersion.Version;
 
        internal static bool IsOSVistaOrNewer
        {
            get { return _osVersion >= new Version(6, 0); }
        }
 
        internal static bool IsOSWindows7OrNewer
        {
            get { return _osVersion >= new Version(6, 1); }
        }
 
        internal static bool IsOSWindows8OrNewer
        {
            get { return _osVersion >= new Version(6, 2); }
        }
```

上面方法不能判断是win10系统

[关于C#中Environment.OSVersion判断系统及Win10上的问题 - 夏至千秋 - 博客园](https://www.cnblogs.com/chihirosan/p/5139078.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
