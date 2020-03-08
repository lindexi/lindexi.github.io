# C# 大端小端转换

关于大端和小端，是一个有趣的问题。本文告诉大家如何在C#转换大端和小端。


<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


这里有一个有趣的故事，请看[详解大端模式和小端模式 - CSDN博客](https://blog.csdn.net/ce123_zhouwei/article/details/6971544 )

默认的 C# 使用的是小端，如果收到的消息是大端，那么就会出现解析错误。

例如收到的数据是 byte 数组，现在知道数据是大端数据，需要把大端转小端，首先需要把数据复制出来。

## 复制数组

假设收到的数据是 data ，里面的前两个 byte 是不需要的，格式是

![](http://image.acmx.xyz/lindexi%2F2018528102650406.jpg)

也就是需要复制出第2个到第5个byte出来，转换这个数据反序。

复制数组的方式有很多个，例如 Array.Copy 和 Buffer.BlockCopy 两个函数使用方式差不多

下面我使用 Array.Copy 做例子

首先定义一个数组用来反序

```csharp
var revertByteList = new byte[4];
```

然后复制数据

```csharp
Array.Copy(data, 2, revertByteList, 0, 4);
```

对数据反序，这样就转换大端

```csharp
revertByteList = revertByteList.Reverse().ToArray();
```

## 数组转整数

从数组转整数的方式很简单，使用下面代码就可以转换

```csharp
var n = BitConverter.ToInt32(revertByteList, 0);
```

小端转大端就是先把 int 转 byte ，然后按照每 4 个 byte 反序就可以

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
