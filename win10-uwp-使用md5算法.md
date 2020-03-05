# win10 UWP 使用 MD5算法

在我们的应用需求很常见的，我们需要使用md5算法。

uwp的 md5 和 WPF 的使用差不多。


<!--more-->
<!-- CreateTime:2019/7/29 12:02:42 -->


<div id="toc"></div>

在 WPF ，我们使用

```csharp
        private string GetMD5(string str)
        {
            System.Security.Cryptography.MD5CryptoServiceProvider md5 = new System.Security.Cryptography.MD5CryptoServiceProvider();
            byte[] temp;
            StringBuilder strb = new StringBuilder();
            temp = md5.ComputeHash(Encoding.Unicode.GetBytes(str));
            md5.Clear();
            for (int i = 0; i < temp.Length; i++)
            { 
                strb.Append(temp[i].ToString("X").PadLeft(2 , '0'));
            }
            return strb.ToString().ToLower();            
        }
```

然而在 UWP ，没有`System.Security.Cryptography.MD5CryptoServiceProvider`，新的加密类放在`Windows.Security.Cryptography.Core.CryptographicHash`

UWP 的 md5使用很简单

首先添加在类的最前，让我们打的时候减少一些。

```csharp
using Windows.Security.Cryptography;
using Windows.Security.Cryptography.Core;
using Windows.Storage.Streams;

```

然后把输入的字符串转为 md5 需要的二进制，注意编码。

```csharp
            Windows.Security.Cryptography.Core.HashAlgorithmProvider objAlgProv = Windows.Security.Cryptography.Core.HashAlgorithmProvider.OpenAlgorithm(Windows.Security.Cryptography.Core.HashAlgorithmNames.Md5);
            Windows.Security.Cryptography.Core.CryptographicHash md5 = objAlgProv.CreateHash();
            Windows.Storage.Streams.IBuffer buffMsg1 = Windows.Security.Cryptography.CryptographicBuffer.ConvertStringToBinary(str , Windows.Security.Cryptography.BinaryStringEncoding.Utf16BE);
            md5.Append(buffMsg1);
            Windows.Storage.Streams.IBuffer buffHash1 = md5.GetValueAndReset();

```

`buffHash1`就是转换后的二进制，我们可以把它转为 base64 或 Hex

网上很多都是 Hex ，基本看到 md5 就是二进制转 Hex， Hex 就是16进制。

我们先说下如何转为 Base64

```csharp
Windows.Security.Cryptography.CryptographicBuffer.EncodeToBase64String(buffHash1);

```

那么如何转为 Hex ？

```csharp
CryptographicBuffer.EncodeToHexString(buffHash1);

```

下面写出代码，测试通过，在站长工具转换结果一样

```csharp
        public static string Md5(string str)
        {
            HashAlgorithmProvider algorithm = HashAlgorithmProvider.OpenAlgorithm(HashAlgorithmNames.Md5);
            CryptographicHash md5 = algorithm.CreateHash();
            Windows.Storage.Streams.IBuffer buffer = CryptographicBuffer.ConvertStringToBinary(str, BinaryStringEncoding.Utf16BE);
            md5.Append(buffer);
            return CryptographicBuffer.EncodeToHexString(md5.GetValueAndReset());
        }

```

<script src="https://gist.github.com/lindexi/0ecf1d8de7a222cda5f058e74de335c1.js"></script>


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 