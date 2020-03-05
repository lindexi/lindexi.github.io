# win10 uwp 读取文本GBK错误

本文讲的是解决UWP文本GBK打开乱码错误，如何去读取GBK，包括网页GBK。最后本文给出一个方法追加文本。
<!--more-->
<!-- CreateTime:2019/8/24 16:02:27 -->


<div id="toc"></div>

我使用NotePad记事本保存文件，格式ASCII，用微软示例打开文件方式读取，出现错误

“在多字节的目标代码页中，没有此 Unicode 字符可以映射到的字符”

英文 No mapping for the Unicode character exists in the target multi-byte code page

这个问题看来很简单，不就是编码错误，最后我就弄了一晚上

我先换个说法，让大家容易搜索到

 - UWP 读ASCII错误

 - UWP read ASCII
 
 - UWP GBK
 
 - UWP 读取记事本

 - UWP 访问GBK网页乱码

 - UWP 乱码
 
 - UWP GB2312 乱码 
 
 - UWP 网页乱码

 

其实不知道垃圾wr怎么想，现在没法读ASCII，把文本用文本编辑器打开，另存为的时候保存为 ASCII 格式，使用官方给的代码，直接错。

用了nos大神的代码[http://blog.csdn.net/nomasp/article/details/50310357](http://blog.csdn.net/nomasp/article/details/50310357 )，也是报错

用了我csdn博客置顶代码，就直接乱码 所有中文为 "?" ，但是我以前就是这样写，可以读取到？原因是因为文本保存为

查了一下WPF使用默认可以读，原因是默认的 WPF 的编码读取是 GBK 所以这时就可以读取，但是在 UWP 是没有 GBK 的，默认也不是。因为 UWP 是 .net core 程序，.net core 没有默认支持 GBK ，因为 .net core 是最小化的，

从[微软](https://docs.microsoft.com/en-us/dotnet/standard/base-types/character-encoding )的文档可以看到下面的说明

> By default, .NET Core does not make available any code page encodings other than code page 28591 and the Unicode encodings, such as UTF-8 and UTF-16. However, you can add the code page encodings found in standard Windows apps that target .NET to your app. For complete information, see the CodePagesEncodingProvider topic.

> 默认.net core 不包含除了 代码页为 28591 和 Unicode(utf-8,utf-16) 之外的其他编码，但是标准的 .net Framework 开发的程序中使用其他的编码，而且可以在标准的 windows 程序支持其他编码，详细请看[CodePagesEncodingProvider](https://msdn.microsoft.com/en-us/library/mt643901(v=vs.110).aspx)

也就是我们保存时GBK，查询到Encoding没有GBK，没有默认的，所以看起来这个问题不是简单就可以通过。

于是我就在网上找，很久没找到，但是找到 [http://www.cnblogs.com/yffswyf/p/4826207.html](http://www.cnblogs.com/yffswyf/p/4826207.html )，写到一半我就不想写，好难

在网上看到Encoding.GetEncoding（0）就是默认编码，于是我找了 GetEncoding，原来有string，那么`Encoding gbk = Encoding.GetEncoding("GBK");`是否就是可以，运行代码

报错
 'GBK' is not a supported encoding name. 
 
看来这个也不可以，我觉得我要写个转换
 
最后发现
https://bbs.uwp.ac.cn/?/article/43 有大神的方法，请看下面代码。

```csharp
//使用CodePagesEncodingProvider去注册扩展编码。
Encoding.RegisterProvider(CodePagesEncodingProvider.Instance);
//注册GBK编码
Encoding encodingGbk = Encoding.GetEncoding("GBK");
```

如果找不到 CodePagesEncodingProvider ，请在nuget搜索`System.Text.Encoding.CodePages`，如果找不到 GBK ，那么请使用 GB3212 代替。

我们在读取之前判断文件的编码，按照不同编码进行不同编码的读取，这个简单判断可以使用这段代码判断

```csharp
        private static Encoding AutoEncoding(byte[] bom)

        {

            if (bom.Length != 4)

            {

                throw new ArgumentException();

            }

            // Analyze the BOM

            if (bom[0] == 0x2b && bom[1] == 0x2f && bom[2] == 0x76) return Encoding.UTF7;

            if (bom[0] == 0xef && bom[1] == 0xbb && bom[2] == 0xbf) return Encoding.UTF8;

            if (bom[0] == 0xff && bom[1] == 0xfe) return Encoding.Unicode; //UTF-16LE

            if (bom[0] == 0xfe && bom[1] == 0xff) return Encoding.BigEndianUnicode; //UTF-16BE

            if (bom[0] == 0 && bom[1] == 0 && bom[2] == 0xfe && bom[3] == 0xff) return Encoding.UTF32;

            return Encoding.ASCII;

        }
```

这没有GBK所以我们只好通过一个垃圾方法。

用`Windows.Storage.FileIO.ReadTextAsync`如果错误了，就使用GBK读，还错误，那么就是文件错了。

编码的错报的ArgumentOutOfRangeException。我们可以Catch，用`GBK`读文本，这样如果读取错误就是文件错了。


全部代码

```csharp
        private async Task<string> Read(StorageFile file)
        {
            string str = "";
            try
            {
                str = await Windows.Storage.FileIO.ReadTextAsync(file);
            }
            catch (ArgumentOutOfRangeException)
            {
                //using(var stream =new StreamReader((await file.OpenReadAsync()).GetInputStreamAt(0).AsStreamForRead()))
                //{
                //    string text = stream.ReadToEnd();
                //    return text;
                //}


                IBuffer buffer = await FileIO.ReadBufferAsync(file);
                DataReader reader = DataReader.FromBuffer(buffer);
                byte[] fileContent = new byte[reader.UnconsumedBufferLength];
                reader.ReadBytes(fileContent);
                string text = "";

               // Encoding.ASCII.GetString(fileContent, 0, fileContent.Length);

               //text= Encoding.GetEncoding(0).GetString(fileContent, 0, fileContent.Length);

                Encoding.RegisterProvider(CodePagesEncodingProvider.Instance);
                Encoding gbk = Encoding.GetEncoding("GBK");

                text = gbk.GetString(fileContent);
                //string text = AutoEncoding(new byte[4] { fileContent[0], fileContent[1], fileContent[2], fileContent[3] }).GetString(fileContent);

                return text;
            }
            return str;
        }

        private static Encoding AutoEncoding(byte[] bom)
        {
            if (bom.Length != 4)
            {
                throw new ArgumentException();
            }
            // Analyze the BOM
            if (bom[0] == 0x2b && bom[1] == 0x2f && bom[2] == 0x76) return Encoding.UTF7;
            if (bom[0] == 0xef && bom[1] == 0xbb && bom[2] == 0xbf) return Encoding.UTF8;
            if (bom[0] == 0xff && bom[1] == 0xfe) return Encoding.Unicode; //UTF-16LE
            if (bom[0] == 0xfe && bom[1] == 0xff) return Encoding.BigEndianUnicode; //UTF-16BE
            if (bom[0] == 0 && bom[1] == 0 && bom[2] == 0xfe && bom[3] == 0xff) return Encoding.UTF32;
            return Encoding.ASCII;
        }
```

参考：http://stackoverflow.com/questions/35296213/read-unicode-string-from-text-file-in-uwp-app/38299563#38299563

http://www.cnblogs.com/loyieking/p/5617508.html

[难道.NET Core到R2连中文编码都不支持吗？ - Artech - 博客园](http://www.cnblogs.com/artech/p/encoding-registeration-4-net-core.html )

文本还有一个坑，我们如何在文本追加？uwp追加文本其实换了类，在FileIO。

对于一个需要追加的文件，UWP追加文件写入其实可以使用`await FileIO.AppendTextAsync(StorageFile,"追加文本");`

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。        

