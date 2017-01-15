# C＃ 判断文件编码

我们的项目中会包含有很多文件，但是可能我们没有注意到的，我们的文件的编码不一定是utf-8，所以可能在别人电脑运行时出现乱码。最近在做一个项目，这个项目可以把我们的文件夹里的所有文本，判断他们是什么编码，如果不是用户规定的编码，那么就告诉用户，是否要把它规范为设置的编码。

<!--more-->
<!-- csdn -->

我们常用的编码有 UTF-8 和 GBK ，所以这就是我们的重点关注编码，可惜现在没有一个好的办法区别 UTF-8 和 GBK 。

如果是带 BOM 的文件，带 BOM 就是带签名，我们可以看到在 VisualStudio 的 文件-高级保存 有 UTF-8 带签名和 UTF-8 编码。

![](http://7xqpl8.com1.z0.glb.clouddn.com/fce442cf-cf99-421e-8fd2-768b101f2bcd201711094251.jpg)

![](http://7xqpl8.com1.z0.glb.clouddn.com/fce442cf-cf99-421e-8fd2-768b101f2bcd201711094313.jpg)

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

那么对于没有带签名的文件，我们如何判断，其实我找了现在很多大神的博客，他们都认为这个是没有一个可行的方法，精确判断，所以我们只能通过一个近似的方法来判断。

找了很久，发现了一个很好的算法，对于文件长度不是3的倍数，和包含有中文、ASCII字符的 GBK 编码文件，几乎不会与UTF8混淆。

我们统计属于GBK的byte个数和属于UTF8的byte个数，比较两个个数，如果countGBK 大于 countUtf8 那么编码就是 GBK，否则是 UTF8。如果相同，gg，所以我们需要一个置信度变量。

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

那么如何统计文件中属于 GBK的byte个数

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

