# C# json 转 xml 字符串

本文告诉大家如何将 json 转 xml 或将 xml 转 json 字符串

<!--more-->
<!-- CreateTime:2019/3/1 9:20:24 -->


首先需要安装 Newtonsoft.Json 库，打开 VisualStudio 2019 新建一个 dotnet core 项目，然后右击编译 csproj 输入下面的代码

```csharp
  <ItemGroup>
    <PackageReference Include="Newtonsoft.Json" Version="12.0.1" />
  </ItemGroup>
```

尝试创建一个类用来转换为 xml 请看代码

```csharp
    public class Foo
    {
        public string Name { get; set; }

        public string Blog { get; set; }
    }
```

将类转换为 xml 的代码

```csharp
            var foo = new Foo()
            {
                Name = "lindexi",
                Blog = "https://blog.csdn.net/lindexi_gd",
            };

            var xmlSerializer = new XmlSerializer(typeof(Foo));
            var str = new StringBuilder();

            xmlSerializer.Serialize(new StringWriter(str), foo);

            var xml = str.ToString();
            Console.WriteLine(xml);
```

现在运行就可以看到下面代码

```
<?xml version="1.0" encoding="utf-16"?>
<Foo xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <Name>lindexi</Name>
  <Blog>https://blog.csdn.net/lindexi_gd</Blog>
</Foo>
```

这里的 encoding 是 utf-16 因为 StringWriter 使用的是 Unicode 如果需要修改为 utf-8 需要修改代码，但是本文就不在这里说

## xml 转 json 字符串

从 xml 转 json 需要将 xml 字符串创建 XmlDocument 才可以

```csharp
            XmlDocument doc = new XmlDocument();
            doc.LoadXml(xml);
```

通过下面代码就可以将 XmlDocument 转 json 字符串

```csharp
            string text = JsonConvert.SerializeXmlNode(doc);
```

运行代码可以看到转换的代码

```csharp
{"?xml":{"@version":"1.0","@encoding":"utf-16"},"Foo":{"@xmlns:xsd":"http://www.w3.org/2001/XMLSchema","@xmlns:xsi":"http://www.w3.org/2001/XMLSchema-instance","Name":"lindexi","Blog":"https://blog.csdn.net/lindexi_gd"}}
```

## json 转 xml 字符串

在上面已经转换出 json 可以通过下面代码将 json 转 xml 字符串

```csharp
            doc = (XmlDocument) JsonConvert.DeserializeXmlNode(text);

```

如果需要将 doc 做字符串输出，可以使用 `doc.InnerXml` 转字符串

```csharp
            doc = (XmlDocument) JsonConvert.DeserializeXmlNode(text);
            Console.WriteLine("json转xml");
            Console.WriteLine(doc.InnerXml);
```

运行软件可以看到下面代码

```
<?xml version="1.0" encoding="utf-16"?><Foo xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"><Name>lindexi</Name><Blog>https://blog.csdn.net/lindexi_gd</Blog></Foo>
```

下面是全部的代码

```csharp
   class Program
    {
        static void Main(string[] args)
        {
            var foo = new Foo()
            {
                Name = "lindexi",
                Blog = "https://blog.csdn.net/lindexi_gd",
            };

            var xmlSerializer = new XmlSerializer(typeof(Foo));
            var str = new StringBuilder();

            xmlSerializer.Serialize(new StringWriter(str), foo);

            var xml = str.ToString();
            Console.WriteLine(xml);

            XmlDocument doc = new XmlDocument();
            doc.LoadXml(xml);

            string text = JsonConvert.SerializeXmlNode(doc);
            Console.WriteLine("转换json");
            Console.WriteLine(text);

            doc = (XmlDocument) JsonConvert.DeserializeXmlNode(text);
            Console.WriteLine("json转xml");
            Console.WriteLine(doc.InnerXml);

            Console.Read();
        }
    }

    public class Foo
    {
        public string Name { get; set; }

        public string Blog { get; set; }
    }
```

运行可以看到下面方法

```csharp

<?xml version="1.0" encoding="utf-16"?>
<Foo xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <Name>lindexi</Name>
  <Blog>https://blog.csdn.net/lindexi_gd</Blog>
</Foo>
转换json
{"?xml":{"@version":"1.0","@encoding":"utf-16"},"Foo":{"@xmlns:xsd":"http://www.w3.org/2001/XMLSchema","@xmlns:xsi":"http://www.w3.org/2001/XMLSchema-instance","Name":"lindexi","Blog":"https://blog.csdn.net/lindexi_gd"}}
json转xml
<?xml version="1.0" encoding="utf-16"?><Foo xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"><Name>lindexi</Name><Blog>https://blog.csdn.net/lindexi_gd</Blog></Foo>
```

[Converting between JSON and XML](https://www.newtonsoft.com/json/help/html/ConvertingJSONandXML.htm )

代码 https://gitee.com/lindexi/lindexi_gd/tree/dev/LapouRairpaltearwou


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

