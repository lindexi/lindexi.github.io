# win10 UWP 序列化

将对象的状态信息转换为可以存储或传输的形式的过程。在序列化期间，对象将其当前状态写入到临时或持久性存储区。以后，可以通过从存储区中读取或反序列化对象的状态，重新创建该对象。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:50 -->

<div id="toc"></div>

.NET Framework 提供了两个序列化技术：

 - 二进制序列化保持类型保真，这对于多次调用应用程序时保持对象状态非常有用。例如，通过将对象序列化到剪贴板，可在不同的应用程序之间共享对象。您可以将对象序列化到流、磁盘、内存和网络等。远程处理使用序列化，“按值”在计算机或应用程序域之间传递对象。


 - XML 序列化只序列化公共属性和字段，并且不保持类型保真。当您希望提供或使用数据而不限制使用该数据的应用程序时，这一点非常有用。

## Binary 

首先需要定义一个类，这个类作为保存的类，需要使用特性 Serializable

![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F2017420192248.jpg)

然后使用 `binaryFormatter` 他可以写入流

创建一个文件夹，返回和People类，就可以把他保存在本地


```csharp
             BinaryFormatter binaryFormatter = new BinaryFormatter();
            FileStream stream = new FileStream("./file", FileMode.Create);
            binaryFormatter.Serialize(stream, people);
```

这就是序列化，如果需要从保存的文件拿出来，可以参见下面代码：


```csharp
             using (FileStream stream = new FileStream("./file", FileMode.Open))
            {
                people = (People) binaryFormatter.Deserialize(stream);
                Console.WriteLine(people.Name);
            }
```

可以看到代码都是使用命令行不是使用通用程序

## XML序列化

```csharp
               XmlSerializer xmlSerializer = new XmlSerializer(typeof(People));

            using (FileStream stream = new FileStream("./file", FileMode.Create))
            {
                xmlSerializer.Serialize(stream, people);
            }

            using (FileStream stream = new FileStream("./file", FileMode.Open))
            {
                people = (People)xmlSerializer.Deserialize(stream);
                Console.WriteLine(people);
            }
```


如果有一些属性需要不显示，也就是不放在文件，可以参见：[https://msdn.microsoft.com/zh-cn/library/83y7df3e(v=vs.90).aspx](https://msdn.microsoft.com/zh-cn/library/83y7df3e(v=vs.90).aspx)

XmlSerializer 创建 C# 文件并将其编译为 .dll 文件，以执行此序列化。
为了提高性能，XML 序列化基础结构动态生成程序集，以便对指定类型进行序列化和反序列化。该基础结构将找到并重新使用这些程序集。仅当使用以下构造函数时，才会发生此行为：

XmlSerializer.XmlSerializer(Type)

XmlSerializer.XmlSerializer(Type, String)

如果使用任何其他构造函数，则将生成同一个程序集的多个版本，这些版本始终不予卸载

Yaml序列化

首先搜索 YamlDotNet ，安装

![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F201742019318.jpg)

这里使用参见 http://www.cnblogs.com/RicCC/archive/2010/03/01/serialization-data-format.html



参见：

[win10 uwp json](http://lindexi.oschina.io/lindexi//post/win10-uwp-json/)

[win10 uwp 读写XML](http://lindexi.oschina.io/lindexi//post/win10-uwp-%E8%AF%BB%E5%86%99XML/)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  