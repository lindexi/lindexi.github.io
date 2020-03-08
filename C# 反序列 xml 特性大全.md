# C# 序列类为 xml 可以使用的特性大全

本文告诉大家如何使用序列类，以及序列时可以用到的特性，特性的作用和一些容易被问的问题

<!--more-->
<!-- CreateTime:2019/11/29 8:59:02 -->

<div id="toc"></div>

最近我在把项目文件修改为 VisualStudio 2017 的格式，请看[从以前的项目格式迁移到 VS2017 新项目格式](https://blog.lindexi.com/post/%E4%BB%8E%E4%BB%A5%E5%89%8D%E7%9A%84%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F%E8%BF%81%E7%A7%BB%E5%88%B0-VS2017-%E6%96%B0%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F.html )，这时虽然可以自动打包，但是我还是需要生成 Nuspec 文件，所以本文就是记录我在从 csproj 文件创建 nuspec 文件遇到的转换

实际就是做[将 .NET Core 项目打一个最简单的 NuGet 源码包，安装此包就像直接把源码放进项目一样 - walterlv](https://walterlv.github.io/post/the-simplest-way-to-pack-a-source-code-nuget-package.html )，把项目作为源代码打包

## 保存序列类

例如有类 NuspecMetadata ，需要把这个类转换为 xml 字符串，可以使用下面的代码

```csharp
    public class NuspecMetadata
    {
        public string Id { get; set; }
    }
```

先创建 StringBuilder 使用 XmlWriter 写入，使用 XmlSerializer 序列

```csharp
            var nuspecMetadata = new NuspecMetadata()
            {
                Id = "lindexi.MVVM.Framework"
            };
            var str = new StringBuilder();

            using (var xmlWriter = XmlWriter.Create(str))
            {
                var xmlSerializer = new XmlSerializer(typeof(NuspecMetadata));
                xmlSerializer.Serialize(xmlWriter, nuspecMetadata);
            }
```

这时使用 `str.ToString()` 可以看到下面代码

```xml
<?xml version="1.0" encoding="utf-16"?><NuspecMetadata xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema"><Id>lindexi.MVVM.Framework</Id></NuspecMetadata>
```

这就是序列类的方法，现在看起来和 nuspec 文件还不一样，所以下面告诉大家如何修改

## 设置属性别名

可以看到 nuspec 文件的属性都是使用小写，如

```xml
  <metadata>
     <!-- The unique identifier for the package. This is the package name that is shown
     when packages are listed using the Package Manager Console. These are also used when
     installing a package using the Install-Package command within the Package Manager
     Console. Package IDs may not contain any spaces or characters that are invalid in
     an URL. In general, they follow the same rules as .NET namespaces do. So Foo.Bar
     is a valid ID, Foo! and Foo Bar are not. -->
    <id>lindexi.MVVM.Framework</id>
  </metadata>
```

如果创建 `metadata` 类，那么属性 id 需要使用大写

```csharp
    public class NuspecMetadata
    {
        public string Id { get; set; }
    }
```

这时如果序列NuspecMetadata就会发现创建的 id 是大写的`Id`，这不是需要的

```xml
<?xml version="1.0" encoding="utf-16"?><NuspecMetadata xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
    <Id>lindexi.MVVM.Framework</Id>
</NuspecMetadata>
```

在 id 属性添加 `XmlElement` 可以告诉序列的元素叫什么，而不是直接从属性名作为元素

```csharp
    public class NuspecMetadata
    {
        [XmlElement("id")]
        public string Id { get; set; }
    }
```

因为添加`[XmlElement("id")]` 现在 xml 知道这个属性叫 `id` 所以这时运行上面的转换代码，可以看到下面的代码

```xml
<?xml version="1.0" encoding="utf-16"?><NuspecMetadata xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema"><id>lindexi.MVVM.Framework</id></NuspecMetadata>
```

现在所有的代码

```csharp
        static void Main(string[] args)
        {
            var nuspecMetadata = new NuspecMetadata()
            {
                Id = "lindexi.MVVM.Framework"
            };
            var str = new StringBuilder();

            using (var xmlWriter = XmlWriter.Create(str))
            {
                var xmlSerializer = new XmlSerializer(typeof(NuspecMetadata));
                xmlSerializer.Serialize(xmlWriter, nuspecMetadata);
            }

            var rawceeyopereSuwhisa = str.ToString();
            Console.WriteLine(rawceeyopereSuwhisa);
        }

    public class NuspecMetadata
    {
        [XmlElement("id")]
        public string Id { get; set; }
    }
```

## 设置属性作为 XmlAttribute

在 nuspec 文件存在一些属性是需要做特性，如

```csharp
<dependency id="lindexi.wpf.Framework" version="[1.1.2,)"></dependency>
```

那么先定义 dependency 类

```csharp
    public class NuspecDependency
    {
        public string Id { get; set; }

        public string Version { get; set; }
    }
```

这时使用下面代码序列 NuspecDependency 可以看到 id 和版本都作为元素而不是特性，这和上面代码的不相同

```csharp
    public class NuspecDependency
    {
        public string Id { get; set; }

        public string Version { get; set; }
    }

    // 其他代码

             var nuspecDependency = new NuspecDependency()
            {
                Id = "lindexi.wpf.Framework",
                Version = "[1.1.2,)"
            };

            var str = new StringBuilder();

            using (var xmlWriter = XmlWriter.Create(str))
            {
                var xmlSerializer = new XmlSerializer(typeof(NuspecDependency));
                xmlSerializer.Serialize(xmlWriter, nuspecDependency);
            }
```

这时运行代码，可以看到 str 的值是下面代码

```xml

<?xml version="1.0" encoding="utf-16"?><NuspecDependency xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema"><Id>lindexi.wpf.Framework</Id><Version>[1.1.2,)</Version></NuspecDependency>
```

可以使用 XmlAttribute 告诉 xml 这个属性是作为特性，而且可以告诉 xml 属性作为特性叫什么，而不是拿属性的名作为特性

修改上面的代码为下面代码

```csharp
    public class NuspecDependency
    {
        [XmlAttribute("id")]
        public string Id { get; set; }

        [XmlAttribute(attributeName: "version")]
        public string Version { get; set; }
    }
```

添加特性 XmlAttribute 就可以告诉 xml 这个属性作为特性，现在运行上面代码，可以看到 str 的值和需要的一样

```xml
<?xml version="1.0" encoding="utf-16"?><NuspecDependency xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" id="lindexi.wpf.Framework" version="[1.1.2,)" />
```

## 设置类别名

从上面代码可以看到 NuspecDependency 的类和需要的 dependency 不相同

```
      <dependency id="lindexi.wpf.Framework" version="[1.1.2,)"></dependency>

```

可以使用 XmlType 告诉 xml 这个类序列叫什么而不是直接使用类

```csharp
    [XmlType("dependency")]
    public class NuspecDependency
    {
        [XmlAttribute("id")]
        public string Id { get; set; }

        [XmlAttribute(attributeName: "version")]
        public string Version { get; set; }
    }
```

这个代码主要是添加`[XmlType("dependency")]`告诉 xml 把 NuspecDependency 在序列使用`dependency` 尝试运行上面代码，现在的 str 的值就把 NuspecDependency 修改

```csharp
<?xml version="1.0" encoding="utf-16"?><dependency xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" id="lindexi.wpf.Framework" version="[1.1.2,)" />
```

## 设置列表元素别名

但是 dependency 的使用是这样使用

```xml
    <dependencies>
      <dependency id="lindexi.wpf.Framework" version="[1.1.2,)"></dependency>
    </dependencies>
```

所以在 NuspecMetadata 类添加下面代码

```csharp
        public List<NuspecDependency> Dependencies { set; get; } = new List<NuspecDependency>();

```

所有代码

```csharp

    public class NuspecMetadata
    {
        [XmlElement("id")]
        public string Id { get; set; }

        public List<NuspecDependency> Dependencies { set; get; } = new List<NuspecDependency>();
    }

            var nuspecMetadata = new NuspecMetadata()
            {
                Id = "lindexi.MVVM.Framework",
                Dependencies =
                {
                    new NuspecDependency()
                    {
                        Id = "lindexi.wpf.Framework",
                        Version = "[1.1.2,)"
                    }
                }
            };
            var str = new StringBuilder();

            using (var xmlWriter = XmlWriter.Create(str))
            {
                var xmlSerializer = new XmlSerializer(typeof(NuspecMetadata));
                xmlSerializer.Serialize(xmlWriter, nuspecMetadata);
            }

            var rawceeyopereSuwhisa = str.ToString();
```

这时尝试运行，请看 str 的值

```xml
<?xml version="1.0" encoding="utf-16"?><NuspecMetadata xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema"><id>lindexi.MVVM.Framework</id><Dependencies><dependency id="lindexi.wpf.Framework" version="[1.1.2,)" /></Dependencies></NuspecMetadata>
```

可以看到 Dependencies 的输出还是有些不相同

这是代码的输出

```xml
<Dependencies><dependency id="lindexi.wpf.Framework" version="[1.1.2,)" /></Dependencies>
```

这是需要的文件

```xml
    <dependencies>
      <dependency id="lindexi.wpf.Framework" version="[1.1.2,)"></dependency>
    </dependencies>
```

对比一下可以发现属性的名不对

在 xml 对于列表或数组的序列是需要做特殊处理，请看代码

```csharp
        [XmlArray(elementName: "dependencies")]
        [XmlArrayItem(elementName: "dependency")]
        public List<NuspecDependency> Dependencies { set; get; } = new List<NuspecDependency>();
```

这时运行代码可以看到 str 的值是符合

```csharp
<?xml version="1.0" encoding="utf-16"?><NuspecMetadata xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema"><id>lindexi.MVVM.Framework</id><dependencies><dependency id="lindexi.wpf.Framework" version="[1.1.2,)" /></dependencies></NuspecMetadata>
```

添加的代码是`[XmlArray(elementName: "dependencies")]`告诉这是一个列表，使用`[XmlArrayItem(elementName: "dependency")]`告诉每一列叫什么

因为已经设置了 NuspecDependency 的名，所以设置 XmlArrayItem 没看出效果，尝试把 XmlArrayItem 修改为

```csharp
        [XmlArrayItem(elementName: "doubi")]

```

这时运行可以看到把 dependency 修改为 doubi ，请看代码

```xml
<?xml version="1.0" encoding="utf-16"?><metadata xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema"><id>lindexi.MVVM.Framework</id><dependencies><doubi id="lindexi.wpf.Framework" version="[1.1.2,)" /></dependencies></metadata>
```

这是原来的代码

```csharp
<dependencies><dependency id="lindexi.wpf.Framework" version="[1.1.2,)" /></dependencies>
```

修改后的代码

```csharp
<dependencies><doubi id="lindexi.wpf.Framework" version="[1.1.2,)" /></dependencies>
```

所有代码

```csharp
   [XmlType(typeName: "metadata")]
    public class NuspecMetadata
    {
        [XmlElement("id")]
        public string Id { get; set; }

        [XmlArray(elementName: "dependencies")]
        public List<NuspecDependency> Dependencies { set; get; } = new List<NuspecDependency>();
    }

    [XmlType("dependency")]
    public class NuspecDependency
    {
        [XmlAttribute("id")]
        public string Id { get; set; }

        [XmlAttribute(attributeName: "version")]
        public string Version { get; set; }
    }
```

## 去掉命名空间

默认保存的 xml 的字符串，可以看到如下面的命名空间

```csharp
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema"
```

那么如何去掉`xmlns:xsi`命名空间

最简单的方法是创建 `XmlSerializerNamespaces` 添加空白的命名空间

```csharp
            XmlSerializerNamespaces ns = new XmlSerializerNamespaces();
            ns.Add("", "");
```

在序列类时传入

```csharp
                xmlSerializer.Serialize(xmlWriter, nuspecMetadata, ns);
```

所有代码

```csharp
         var nuspecMetadata = new NuspecMetadata()
            {
                Id = "lindexi.MVVM.Framework",
                Dependencies =
                {
                    new NuspecDependency()
                    {
                        Id = "lindexi.wpf.Framework",
                        Version = "[1.1.2,)"
                    }
                }
            };

            var ns = new XmlSerializerNamespaces();
            ns.Add("", "");

            var str = new StringBuilder();

            using (var xmlWriter = XmlWriter.Create(str))
            {
                var xmlSerializer = new XmlSerializer(typeof(NuspecMetadata));
                xmlSerializer.Serialize(xmlWriter, nuspecMetadata, ns);
            }

            var rawceeyopereSuwhisa = str.ToString();
            Console.WriteLine(rawceeyopereSuwhisa);

    [XmlType(typeName: "metadata")]
    public class NuspecMetadata
    {
        [XmlElement("id")]
        public string Id { get; set; }

        [XmlArray(elementName: "dependencies")]
        public List<NuspecDependency> Dependencies { set; get; } = new List<NuspecDependency>();
    }

    [XmlType("dependency")]
    public class NuspecDependency
    {
        [XmlAttribute("id")]
        public string Id { get; set; }

        [XmlAttribute(attributeName: "version")]
        public string Version { get; set; }
    }
```

尝试运行上面代码

```xml
<?xml version="1.0" encoding="utf-16"?><metadata><id>lindexi.MVVM.Framework</id><dependencies><dependency id="lindexi.wpf.Framework" version="[1.1.2,)" /></dependencies></metadata>
```

## 找不到文件异常

在保存文件的构造函数 XmlSerializer 如果在 dotnet framework 4.5 以上，那么会出现异常

System.IO.FileNotFoundException

```csharp
System.IO.FileNotFoundException occurred
  Message="Could not load file or assembly '[Containing Assembly of MyType].XmlSerializers, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null' or one of its dependencies. The system cannot find the file specified."
  Source="mscorlib"
  FileName="[Containing Assembly of MyType].XmlSerializers, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null"
  FusionLog=""
  StackTrace:
       at System.Reflection.Assembly._nLoad(AssemblyName fileName, String codeBase, Evidence assemblySecurity, Assembly locationHint, StackCrawlMark& stackMark, Boolean throwOnFileNotFound, Boolean forIntrospection)
       at System.Reflection.Assembly.nLoad(AssemblyName fileName, String codeBase, Evidence assemblySecurity, Assembly locationHint, StackCrawlMark& stackMark, Boolean throwOnFileNotFound, Boolean forIntrospection)
```

这是因为垃圾微软会先找程序集的 XmlSerializers ，也就是[xx程序集].XmlSerializers.dll 从这个程序集可能包含如何序列类的代码，这样可以提高性能。如果这个dll 没有生成，那么就会出现这个异常。默认是没有生成这个类。这里出现了异常，没关系，垃圾微软会在构造函数拿到这个异常，在运行时生成序列的代码。

所以只需要不管这个异常就可以

## XmlIgnore 

这个特性表示类的某个属性需要在序列忽略，也就是不使用这个属性

在 xml 序列忽略某个属性就需要在这个属性设置 `[XmlIgnore]` ，请看代码

```csharp
        [XmlIgnore]
        public string KawbishumaVaslufeeyairrea { get; set; } = "lindexi.github";
```


参见：

[项目文件中的已知 NuGet 属性（使用这些属性，创建 NuGet 包就可以不需要 nuspec 文件啦） - walterlv](https://walterlv.github.io/post/known-nuget-properties-in-csproj.html )

[将 WPF、UWP 以及其他各种类型的旧样式的 csproj 文件迁移成新样式的 csproj 文件 - walterlv](https://walterlv.github.io/post/introduce-new-style-csproj-into-net-framework.html )

[c# - XmlSerializer giving FileNotFoundException at constructor - Stack Overflow](https://stackoverflow.com/questions/1127431/xmlserializer-giving-filenotfoundexception-at-constructor )

![](http://image.acmx.xyz/lindexi%2F201810199941411)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

