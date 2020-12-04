# asp dotnet core 提供大文件下载的测试

本文仅仅是提供测试使用的代码

<!--more-->
<!-- CreateTime:2020/8/1 10:04:52 -->



提供文件下载只需要返回 PhysicalFile 方法，如下面代码

```csharp
        [HttpGet]
        public IActionResult Get()
        {
            var folder = Path.GetDirectoryName(Assembly.GetExecutingAssembly().Location);
            var file = Path.Combine(folder, "big file");
            const string mime = "application/octet-stream";

            return PhysicalFile(file, mime);
        }
```

本文的 big file 是程序运行创建的垃圾文件

```csharp
        private void WriteBigFile()
        {
            var folder = Path.GetDirectoryName(Assembly.GetExecutingAssembly().Location);
            var file = Path.Combine(folder, "big file");

            if (!File.Exists(file))
            {
                using (var stream = new FileStream(file, FileMode.Create))
                {
                    for (int i = 0; i < 100000000; i++)
                    {
                        stream.WriteByte(0);
                    }
                }
            }
        }
```

测试代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/e237082b643c86cd15124f201c82f46955b9ab84/Gaaweeealjrdwrebiny) 欢迎小伙伴访问

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
