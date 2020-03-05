# C# 从 short 转 byte 方法

本文告诉大家多个方法转换 short 和 byte 有简单的也有快的

<!--more-->
<!-- CreateTime:2019/4/29 12:08:39 -->


## 快速简单的方法

```csharp
static short ToShort(short byte1, short byte2)
{
    return (byte2 << 8) + byte1;
}

static void FromShort(short number, out byte byte1, out byte byte2)
{
    byte2 = (byte) (number >> 8);
    byte1 = (byte) (number & 255);
}
```

## 简单的方法

通过[BitConverter](https://docs.microsoft.com/en-us/dotnet/api/system.bitconverter?wt.mc_id=MVP ) 可以将大量的类转换为 byte 包括 short 的方法

```csharp
short number = 42;
byte[] numberBytes = BitConverter.GetBytes(number);
short converted = BitConverter.ToInt16(numberBytes);
```

但是为了这么简单的 short 两个 byte 创建一个数组，感觉不是很好

https://stackoverflow.com/q/1442583/6116637

![](http://image.acmx.xyz/lindexi%2F201942912529158)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
