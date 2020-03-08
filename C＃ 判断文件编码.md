# C＃ 判断文件编码

我们的项目中会包含有很多文件，但是可能我们没有注意到的，我们的文件的编码不一定是utf-8，所以可能在别人电脑运行时出现乱码。最近在做一个项目，这个项目可以把我们的文件夹里的所有文本，判断他们是什么编码，如果不是用户规定的编码，那么就告诉用户，是否要把它规范为设置的编码。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:52 -->


<div id="toc"></div>

## 故事

编码问题是历史带来的，所以需要说下文件编码的历史。

一开始程序员认为 Ascii 就可以表达一切，于是一开始就只有 Ascii 编码。后来 中国 这样强大的国家加入 IT 于是就需要表达自己国家的编码，于是中国就出了GBK，这个一个伟大的编码，因为他最难判断。

最开始我们不客气地把那些127号之后的符号们直接取消掉, 取消之后规定：小于127的和原来一样，但两个大于127的字符连在一起时，就表示一个汉字，于是使用两个字节前面的一个字节（0xA1- 0xF7），后面一个字节（0xA1-0xFE），这样我们就可以组合出大约7000多个简体汉字了。在这些编码里，我们还把数学符号、罗马希腊的字母、日文的假名们都编进去了，连在 ASCII 里本来就有的数字、标点、字母都统统重新编了两个字节长的编码，这就是常说的”全角”字符，而原来在127号以下的那些就叫 “半角” 字符了。 中国人民看到这样很不错，于是就把这种汉字方案叫做 “GB2312“。GB2312 是对 ASCII 的中文扩展。

但是汉字是很多的，所以 GB2312 不够用了，于是就规定了，在后面一个字节，不再是大于127的了，直接是一个大于127的字符加上另一个字符就表示一个字，这个叫 GBK 。GBK兼容 GB2312 ，添加好多新汉字和字符，支持很多民族的字符。

在中国做出自己的编码的时候，中国台湾地区也做出自己的编码， 这就是 BIG-5 ，但是历史原因，大陆把 GBK 包含了 繁体字，于是两个方案就无法准确识别了。现在没有一个方法可以识别 一个文件是 GB2312 还是 Big-5 。

中国都除了两个编码，不能识别的编码，其他国家更是出了好多编码，于是程序员无法识别其他国家的编码。

于是这时，出现了 unicode ，他是一个国际标准，但是没有人使用它。后来在 网络的发展，人们为可以在世界使用自己的网站，就使用了 unicode ，但是他的传输性能比较差，好在有了 Utf-8 。

关于字符编码的故事，可以去看：http://www.jianshu.com/p/326795dab773

因为文件编码太多，最好是让文件自己说自己是什么编码，于是 WR 就说，在文件开始使用两个字节来说明文件是什么编码。于是这就叫文件带签名，这样可以根据文件自己描述，来读取文件。

下面来说下如何识别各种字符。

我们常用的编码有 UTF-8 和 GBK ，所以这就是我们的重点关注编码，可惜现在没有一个好的办法区别 UTF-8 和 GBK 。

如果是带 BOM 的文件，带 BOM 就是带签名，我们可以看到在 VisualStudio 的 文件-高级保存 有 UTF-8 带签名和 UTF-8 编码。

![](http://image.acmx.xyz/fce442cf-cf99-421e-8fd2-768b101f2bcd201711094251.jpg)

![](http://image.acmx.xyz/fce442cf-cf99-421e-8fd2-768b101f2bcd201711094313.jpg)

那么带签名的意思是什么，这个和历史有关，我们做出了太多编码，有时无法解析文件的编码，如我们在记事本写上联通，再次打开会是乱码的原因一样，为了让文件自己告诉是什么编码，我们就取文件的前四个 byte ，用于让文件说出自己的编码。

对带签名文件，我们可以简单得到他的编码。See：http://lindexi.oschina.io/lindexi/post/win10-uwp-%E8%AF%BB%E5%8F%96%E6%96%87%E6%9C%ACGBK%E9%94%99%E8%AF%AF/


```csharp
        private static Encoding AutoEncoding(byte[] bom)
        {
            if (bom.Length != 4)
            {
                throw new ArgumentException("EncodingScrutator.AutoEncoding 参数大小不等于4");
            }

            // Analyze the BOM

            if (bom[0] == 0x2b && bom[1] == 0x2f && bom[2] == 0x76)
                return Encoding.UTF7; //85 116 102 55    //utf7 aa 97 97 0 0
            //utf7 编码 = 43 102 120 90

            if (bom[0] == 0xef && bom[1] == 0xbb && bom[2] == 0xbf)
                return Encoding.UTF8; //无签名 117 116 102 56
            // 130 151 160 231
            if (bom[0] == 0xff && bom[1] == 0xfe)
                return Encoding.Unicode; //UTF-16LE

            if (bom[0] == 0xfe && bom[1] == 0xff)
                return Encoding.BigEndianUnicode; //UTF-16BE

            if (bom[0] == 0 && bom[1] == 0 && bom[2] == 0xfe && bom[3] == 0xff) return Encoding.UTF32;

            return Encoding.ASCII; //如果返回ASCII可能是GBK、无签名 utf8
        }
```

那么对于没有带签名的文件，我们如何判断？其实我找了现在很多大神的博客，他们都认为这个是没有一个可行的方法，精确判断。所以我们只能通过一个近似的方法来判断。

找了很久，发现了一个很好的算法，对于文件长度不是3的倍数，和包含有中文、ASCII字符的 GBK 编码文件，几乎不会与UTF8混淆。

我们统计属于 GBK 的 byte 个数和属于UTF8的byte个数，比较两个个数，如果countGBK 大于 countUtf8 那么编码就是 GBK，否则是 UTF8。如果相同，gg，所以我们需要一个置信度变量。

看起来我们需要好多变量，于是写一个类


```csharp
  using System.IO;
  using System.Text;

namespace EncodingNormalior.Model
{
    /// <summary>
    ///     包括文件和编码
    /// </summary>
    public class EncodingScrutatorFile
    {
        /// <summary>
        ///     文件
        /// </summary>
        public FileInfo File { set; get; }

        /// <summary>
        ///     编码
        /// </summary>
        public Encoding Encoding { set; get; }

        /// <summary>
        ///     置信度
        ///     范围0-1,1表示确定，0表示不确定，注意：ASCII编码的置信度为0
        /// </summary>
        public double ConfidenceCount { set; get; } = 0;
    }
}
```

那么如何统计文件中属于 GBK的 byte 个数

我们需要知道 GBK 的编码，对于一般的 ASCII 字符，使用一个 byte 和ASCII一样，如果一个文件都是 ASCII 字符，那么GBK 编码和 ASCII 的都一样，我们统计得到属于 GBK的byte个数为0。对于其他的字符，使用两个 byte 表示。

我找到了一个大神写的判断，https://gist.github.com/neesenk/956765

<script src="https://gist.github.com/neesenk/956765.js"></script>

实际上试过了，不如使用`firstByte >= 161 && firstByte <= 247 && secondByte >= 161 && secondByte <= 254`判断。

那么知道了如何判断一个字符是属于GBK，那么我们可以开始写函数CountGbk

```csharp
        /// <summary>
        /// 统计文件属于 GBK 的 byte数
        /// </summary>
        /// <returns></returns>
        private int CountGbk()
        {
            var count = 0; //存在GBK的byte
            if (CountBuffer == null)
            {
                ReadStream();
            }
            var length = CountBuffer.Length; //总长度

            var buffer = CountBuffer;
            const char head = (char) 0x80; //小于127 通过 &head==0

            for (var i = 0; i < length; i++)
            {
                var firstByte = buffer[i]; //第一个byte，GBK有两个
                if ((firstByte & head) == 0) //如果是127以下，那么就是英文等字符，不确定是不是GBK
                {
                    continue; //文件全部都是127以下字符，可能是Utf-8 或ASCII
                }
                if (i + 1 >= length) //如果是大于127，需要两字符，如果只有一个，那么文件错了，但是我也没法做什么
                {
                    break;
                }
                var secondByte = buffer[i + 1]; //如果是GBK，那么添加GBK byte 2
                if (firstByte >= 161 && firstByte <= 247 &&
                    secondByte >= 161 && secondByte <= 254)
                {
                    count += 2;
                    i++;
                }
            }
            return count;
        }
```


统计文件中属于 utf8 的byte个数

```csharp
        /// <summary>
        /// 属于 UTF8 的 byte 数
        /// </summary>
        /// <returns></returns>
        private int CountUtf8()
        {
            var count = 0;
            if (CountBuffer == null)
            {
                ReadStream();
            }

            var length = CountBuffer.Length;


            var buffer = CountBuffer; // new byte[length];
            const char head = (char) 0x80;
            //while ((n = stream.Read(buffer, 0, n)) > 0)
            {
                for (var i = 0; i < length; i++)
                {
                    var temp = buffer[i];
                    if (temp < 128) //  !(temp&head)
                    {
                        //utf8 一开始如果byte大小在 0-127 表示英文等，使用一byte
                        //length++; 我们记录的是和CountGBK比较
                        continue;
                    }
                    var tempHead = head;
                    var wordLength = 0; //单词长度，一个字使用多少个byte

                    while ((temp & tempHead) != 0) //存在多少个byte
                    {
                        wordLength++;
                        tempHead >>= 1;
                    }

                    if (wordLength <= 1)
                    {
                        //utf8最小长度为2
                        continue;
                    }

                    wordLength--; //去掉最后一个，可以让后面的 point大于wordLength
                    if (wordLength + i >= length)
                    {
                        break;
                    }
                    var point = 1; //utf8的这个word 是多少 byte
                    //utf8在两字节和三字节的编码，除了最后一个 byte 
                    //其他byte 大于127 
                    //所以 除了最后一个byte，其他的byte &head >0
                    for (; point <= wordLength; point++)
                    {
                        var secondChar = buffer[i + point];
                        if ((secondChar & head) == 0)
                        {
                            break;
                        }
                    }

                    if (point > wordLength)
                    {
                        count += wordLength + 1;
                        i += wordLength;
                    }
                }
            }
            return count;
        }
```

我们判断如果是不带签名的文件，判断为 UTF8 或GBK，可以使用判断属于 GBK 的 byte 多还是 UTF8 多。


```csharp
                var countUtf8 = CountUtf8();
                if (countUtf8 == 0)
                {
                    encoding = Encoding.ASCII;
                }
                else
                {
                    var countGbk = CountGbk();
                    if (countUtf8 > countGbk)
                    {
                        encoding = Encoding.UTF8;
                        EncodingScrutatorFile.ConfidenceCount = (double) countUtf8/(countUtf8 + countGbk);
                    }
                    else
                    {
                        encoding = Encoding.GetEncoding("GBK");
                        EncodingScrutatorFile.ConfidenceCount = (double) countGbk/(countUtf8 + countGbk);
                    }
                }
```

下面是我看到的大神的博客，如果希望了解编码的问题，可以参见下面的博客。

我把项目开源，希望能帮到大家。

https://github.com/lindexi/EncodingNormalior

听说项目的名字拼错了，大家不要笑。

参见：http://blog.csdn.net/wwlhsgs/article/details/45641997

http://blog.csdn.net/wgw335363240/article/details/41700045

http://www.ruanyifeng.com/blog/2010/02/url_encoding.html

http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html

http://blog.codingnow.com/2010/06/detect_utf-8_gbk.html

https://msdn.microsoft.com/en-us/library/dd374101(VS.85).aspx

http://www.jianshu.com/p/326795dab773

http://blog.sina.com.cn/s/blog_63426ff90100he20.html

https://my.oschina.net/1pei/blog/390663

最近发现我的方法在国内可以使用，如果使用的软件会在很多个国家使用，还需要去想如何识别他们国家的编码，于是我找到一个好的库

[errepi/ude: A C# port of Mozilla Universal Charset Detector.](https://github.com/errepi/ude)

但是他不太好用，于是用了大神写的 [NuGet Gallery | SimpleHelpers.FileEncoding (C# - Source file) 1.4.0](https://www.nuget.org/packages/SimpleHelpers.FileEncoding/)

实际测试这个方法，对GBK的支持不太好，有一些是GBK的文件会识别为其他格式，所以使用判断默认编码是GBK，如果是，就使用上面提供的方法。

UWP 检测编码可以使用这个库

![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F201752215058.jpg)

[如何检测或判断一个文件或字节流（无BOM）是什么编码类型 - 路过秋天 - 博客园](https://www.cnblogs.com/cyq1162/p/9183424.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  