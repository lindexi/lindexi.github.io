# win10 uwp 读取文本ASCII错误

【】

我使用NotePad记事本保存文件，格式ASCII，用微软示例打开文件方式读取，出现错误

在多字节的目标代码页中，没有此 Unicode 字符可以映射到的字符

英文 No mapping for the Unicode character exists in the target multi-byte code page

这个问题看来很简单，不就是编码错误，可以我就弄了一晚上

我先换个说法，让大家容易搜索到

 - UWP 读ASCII错误

 - UWP read ASCII
 
 - UWP GBK
 
 - UWP 读取记事本

 - UWP 访问GBK网页乱码

 - UWP 乱码
 
 - UWP GB2312 乱码 
 
 - UWP 网页乱码

 

其实不知道垃圾wr怎么想，现在没法读ASCII，官方给的，直接错

用了nos大神的代码http://blog.csdn.net/nomasp/article/details/50310357，也是报错

用了我csdn博客置顶代码，就直接乱码 所有中文代为 "?" 

查了一下WPF使用默认可以读，也就是我们保存时GBK，查询到Encoding没有GBK，没有默认

于是我就在网上找，很久没找到，但是找到http://www.cnblogs.com/yffswyf/p/4826207.html，写到一半我就不想写，好难

在网上看到Encoding.GetEncoding（0）默认，于是我找了GetEncoding，原来有string，那么`Encoding gbk = Encoding.GetEncoding("GBK");`

报错
 'GBK' is not a supported encoding name. 
 
 看来这个也不可以，我觉得我要写个转换
 
最后发现
https://bbs.uwp.ac.cn/?/article/43

```
//使用CodePagesEncodingProvider去注册扩展编码。
Encoding.RegisterProvider(CodePagesEncodingProvider.Instance);
//注册GBK编码
Encoding encodingGbk =Encoding.GetEncoding("GBK");
```

我们在读取之前判断我们的编码，这个简单可以使用

```
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

用Windows.Storage.FileIO.ReadTextAsync如果错误了，就使用GBK读，还错误，那么就是文件错了。

编码的错报的ArgumentOutOfRangeException。我们可以Catch，用GBK就是上面写的。


全部代码

```
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

对于一个需要追加的文件，UWP追加文件写入其实可以使用`await FileIO.AppendTextAsync(StorageFile,"追加文本");`





        
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。        