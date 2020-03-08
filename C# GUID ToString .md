# C# GUID ToString 

最近在看到小伙伴直接使用 Guid.ToString ，我告诉他需要使用 Guid.ToString("N") ，为什么需要使用 N ，因为默认的是 D 会出现连字符。

<!--more-->
<!-- CreateTime:2018/10/19 9:04:44 -->


Guid 是 Globally Unique Identifier 全局唯一标识符，是一种由算法生成的唯一标识是微软的UUID标准的实现。

Guid.ToString 里面可以添加下面几个参数，“N”，“D”，“B”，“P”，“X”

如果直接使用 `Guid.ToString()` 那么就是使用 “D”，这个值大概就是在数字中添加连字符

```csharp
00000000-0000-0000-0000-000000000000
536b4dd7-f3dd-4664-bd69-bc0859d710ab
```

如果使用 “N” 那么就是只有32位数字，数字是 16 进制，字符串有 a-f

```csharp
00000000000000000000000000000000
2329fcac4fd640f1bc221e254b14d621
```

所以我就建议使用 N ，剩下的 B 和 P 只是在使用括号包字符串

```csharp
            System.Console.WriteLine(Guid.NewGuid().ToString("B"));
            {e34dead4-212d-442a-8f4c-e00107baec24}
```

```csharp
System.Console.WriteLine(Guid.NewGuid().ToString("P"));
(ac10d607-2b39-448f-99b5-0a3205cc9ac1)
```

从代码可以看到 B 使用`{` ，P 使用`(`，但是最特殊的是 x ，他会存在 4 个数字，最后一个数字是 8 个数字组合的

```csharp
   Console.WriteLine(Guid.NewGuid().ToString("X"));
  {0xd3f51d9d,0x31b3,0x45f6,{0x9b,0x7c,0x89,0x1d,0xa5,0x6a,0xa3,0x43}}
```

## GUID 转 int 

一个 GUID 需要 16 个 byte 也就是 4 个 int ，可以使用下面的方法转换

```csharp
      public static int[] Guid2Int(Guid value)
        {
            byte[] b = value.ToByteArray();
            int bint = BitConverter.ToInt32(b, 0);
            var bint1 = BitConverter.ToInt32(b, 4);
            var bint2 = BitConverter.ToInt32(b, 8);
            var bint3 = BitConverter.ToInt32(b, 12);
            return new[] {bint, bint1, bint2, bint3};
        }

        public static Guid Int2Guid(int value, int value1, int value2, int value3)
        {
            byte[] bytes = new byte[16];
            BitConverter.GetBytes(value).CopyTo(bytes, 0);
            BitConverter.GetBytes(value1).CopyTo(bytes, 4);
            BitConverter.GetBytes(value2).CopyTo(bytes, 8);
            BitConverter.GetBytes(value3).CopyTo(bytes, 12);
            return new Guid(bytes);
        }
```

参见：[全局唯一标识符 - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/%E5%85%A8%E5%B1%80%E5%94%AF%E4%B8%80%E6%A0%87%E8%AF%86%E7%AC%A6 )

![](http://image.acmx.xyz/65fb6078-c169-4ce3-cdd9-e35752d07be0%2Fyande.re%2520443795%2520sample%2520bikini%2520goto_jun%2520kaneshiro_sora%2520momijidani_nozomi%2520noguchi_takayuki%2520swimsuits%2520tenshi_no_three_piece%2521201841104040.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
