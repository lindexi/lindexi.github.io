# C# 简单读取文件

本文告诉大家如何使用最少的代码把一个文件读取二进制，读取为字符串

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


现在写了一些代码，想使用最少代码来写简单的读文件，所以我就写了这个文章

## 读取文件为二进制

```csharp
        private byte[] ReadFile(FileInfo file)
        {
            var memoryStream = new MemoryStream();
            using (var stream = file.OpenRead())
            {
                stream.CopyTo(memoryStream);
            }

            return memoryStream.GetBuffer();
        }
```

这个方法性能比较差，但是代码很简单

## 读取文件为字符串

```csharp
            string str;
            using (var stream = new StreamReader(file.OpenRead()))
            {
                str = stream.ReadToEnd();
            }
```

第二个方法只需要一句话

```csharp
string str = System.IO.File.ReadAllText(path);
```

这里的 path 就是文件的路径

如果有找到比我上面代码更少的方法请告诉我，这里不用安装第三方的库，是在快速创建新的项目进行测试

我的博客即将搬运同步至腾讯云+社区，邀请大家一同入驻：https://cloud.tencent.com/developer/support-plan?invite_code=19bm8i8js1ezb

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
