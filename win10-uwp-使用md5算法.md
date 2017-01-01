# win10 UWP 使用MD5算法

很常见，我们需要使用md5算法，uwp的md5和WPF的使用差不多


<!--more-->

在WPF，我们使用

```csharp
        private string get_MD5(string str)
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

然而在UWP，没有`System.Security.Cryptography.MD5CryptoServiceProvider`，放在`Windows.Security.Cryptography.Core.CryptographicHash`

UWP的md5使用很简单

首先

```csharp
using Windows.Security.Cryptography;
using Windows.Security.Cryptography.Core;
using Windows.Storage.Streams;

```

然后把输入的字符串转为md5二进制

```csharp
            Windows.Security.Cryptography.Core.HashAlgorithmProvider objAlgProv = Windows.Security.Cryptography.Core.HashAlgorithmProvider.OpenAlgorithm(Windows.Security.Cryptography.Core.HashAlgorithmNames.Md5);
            Windows.Security.Cryptography.Core.CryptographicHash md5 = objAlgProv.CreateHash();
            Windows.Storage.Streams.IBuffer buffMsg1 = Windows.Security.Cryptography.CryptographicBuffer.ConvertStringToBinary(str , Windows.Security.Cryptography.BinaryStringEncoding.Utf16BE);
            md5.Append(buffMsg1);
            Windows.Storage.Streams.IBuffer buffHash1 = md5.GetValueAndReset();

```

`buffHash1`就是转换后的，我们可以把它转为base64或Hex

网上很多都是Hex

转为Base64

```csharp
Windows.Security.Cryptography.CryptographicBuffer.EncodeToBase64String(buffHash1);

```

转为Hex

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



