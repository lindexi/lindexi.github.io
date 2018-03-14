
# win10 uwp 读写XML

UWP 对 读写 XML做了一些修改，但和之前 WPF 的方法没有大的区别。
我们先来说下什么是 XML ，
XML 其实是 树结构，可以表达复杂的结构，所以在定制要求高的、或其他方面如json 做不到的结构，那么一般就使用XML，如果XML的数据结构都做不到，那么基本上也难找到其他的结构。
XML 的优点是读写很简单，也支持定制。缺点是复杂，当然这也是他的优点。在网络传输数据，如果使用XML，相对的传输大小会比 Json 多两倍。所以是不是要用到这么高级的结构，还是看需要。
wr 很喜欢用 XML，可以看到我们的项目，`*.csproj` 和页面 xaml 都是XML，当然Html也是，Xml 其实还可以用作本地数据库，所以 XML 还是很重要。
本文就提供简单的方法来读写 XML 。提供方法有两个，放在前面的方法是比较垃圾的方法，放在后面的才是我希望大家使用的。
如果遇到了 C# 或 UWP 读取 xml 返回的 Node 是空，那么请检查命名空间，关于命名空间内容，请继续看博客。

<!--more-->



<div id="toc"></div>

## xml 语法

xml 一开始一般就是 文档声明

文档声明就是

		
```csharp
<?xml version="1.0" encoding="编码方式" standalone="yes|no"?>

```
XML声明放在XML文档的第一行

XML声明由以下几个部分组成：

 - version 文档符合XML1.0规范

 - encoding 文档字符编码，比如"gb2312"

 - standalone 文档定义是否独立使用

 - standalone="yes" 可选

 - standalone="no"   默认

对于XML标签中出现的所有空格和换行，XML解析程序都会当作标签内容进行处理。

属性值用双引号（"）或单引号（'）分隔

如果希望知道更多，请看：[http://www.cnblogs.com/yaoyinglong/p/xml.html](http://www.cnblogs.com/yaoyinglong/p/xml.html)

下面就是一个 xml 的读写方法。

## XmlDocument 

在 UWP 如果需要 读取解析xml 
我们可以使用 XmlDocument 。

一开始需要创建 XmlDocument ，创建 XmlDocument 有三个方法，首先是从 StorageFile 创建。
		
```csharp
            XmlDocument.LoadFromFileAsync(file);   读取xml


```

注意要等待。



第二方法：从Uri创建，`XmlDocument.LoadFromUriAsync(uri);   `

第三方法：先创建一个 XmlDocument 然后使用 Load
		
```csharp
            
            XmlDocument doc = new XmlDocument();
              
            doc.LoadXml(str);

```

注意str是字符串。

读取xml之后需要解析。

如果想在 xml 中获取某个标签，假如我们获取的是 Page.xaml 的 TextBlock ，那么我们可以遍历一次 doc.FirstChild

		
```csharp
            var grid=doc.FirstChild.ChildNodes;
            for (var i = 0; i < grid.Count; i++)
            {
                var temp = grid[i];
                if (temp.NodeName == "TextBlock")
                {

                }
            }

```

大概是一个垃圾办法，我在下面写一个简单的方法，一般放在最前写的就是最垃圾的方法。

获取了标签，我们还想获取属性，我们可以使用 IXmlNode 的 Attributes 。Attributes 就是所有的属性，假如我们想得到 TextBlock 的 Name ，那么可以使用

		
```csharp
                    foreach (var attribute in temp.Attributes)
                    {
                        if (attribute.NodeName == "Name")
                        {
                            name = "TextBlock" + attribute.InnerText;
                        }
                    }

```

如果你想用 Linq 去查而不使用循环，那么我希望你看到下面的 [Linq读写 XML](# Linq 读写 XML)再写代码，我下面有一个简单的方法。

如何去写入或创建节点，请看：http://www.cnblogs.com/zery/p/3362480.html 

需要注意的是，如果属性有明明空间，那么刚才的方法是比较难用的。



## Linq 读写 XML

这个是我推荐的方法。

首先来说下如何从文件创建 xml ，我们需要使用
		
```csharp
            XDocument doc = XDocument.Load(new StreamReader(
                await file.OpenStreamForReadAsync()));

```

如果需要从字符串创建，那么使用
		
```csharp
XDocument.Load(new StringReader(str));

```

使用的时候，需要`using System.Xml.Linq;`

我们还是来读一个 xaml ，假如我们想拿出所有的 TextBlock ，那么我们有简单的方法。

		
```csharp
            var page = doc.Root;
            var name = page.Name.NamespaceName;

            var textBlockList = page.Descendants(XName.Get("TextBlock", name));

```
注意，我们的 Descendants 参数是 XName，需要使用命名空间，一开始我就不知道需要命名空间，总是没找到 TextBlock 。希望大家在网上看到的博客写的是 string 记住我们的 Descendants 参数是 XName 。

因为我们 xaml 的 TextBlock 是使用命名空间，和简单的 xml 不同，当然，xml 也是可以使用命名空间。其实不可以去责怪大神们没有写 Descendants 的参数是 XName ，因为我们基本遇到的 XML 都不会用到 命名空间。

那么我们就可以简单从 xaml 拿出所有的 TextBlock ，不管他放在多少个 Grid 里。

![](http://7xqpl8.com1.z0.glb.clouddn.com/e53972be-081c-4087-9ea1-bff50ae213b8201714201632.jpg)

看到上面的图片，放了5层的 TextBlock 也可以拿到。

然后我们如何拿到属性，在知道属性的名称情况，可以使用`temp.Attribute("属性名")?.Value`来获得，Attribute 的参数是 XName，那么 为何我们还直接用 string，原因是除了开始用冒号分开的属性，如`x:Name`，其他的都可以直接使用 string。

那么如果是`x:Name`的属性，我们需要使用 x 的命名空间 `http://schemas.microsoft.com/winfx/2006/xaml`

		
```csharp
                string textname = textBlock.Attribute(XName.Get("Name", "http://schemas.microsoft.com/winfx/2006/xaml"))?.Value;


```

获取完属性，我们需要知道如何添加属性。

我们可以使用 SetAttributeValue 来添加删除属性。

假如我们添加 `x:Uid` ，value 是 name

		
```csharp
                textBlock.SetAttributeValue(XName.Get("Uid", "http://schemas.microsoft.com/winfx/2006/xaml"), name);


```

如果我们要删除 Text ，那么使用 `textBlock.SetAttributeValue("Text",null);`

value 是 null，就删除属性。


说完如何添加属性，那么如何添加 node

我们需要用到 XElement

假如我们要添加一个

		
```xml
<node name="lindexi"/>

```

那么我们可以使用
		
```csharp
                            var node=new XElement("node");
                            node.SetAttributeValue("name","lindexi");
                            doc.Root.Add(node);

```

写完保存`doc.Save(await file.OpenStreamForWriteAsync());`

XDocument 和 WPF 的CUID都一样，如果需要删除或其他的方法，请去找WPF的方法。

我使用 XDocument 把 `*.csproj ` 的所有文件拿出来，代码：[https://gist.github.com/lindexi/813e4b7111c16ac7b8a5149f44226e30](https://gist.github.com/lindexi/813e4b7111c16ac7b8a5149f44226e30)

<script src="https://gist.github.com/lindexi/813e4b7111c16ac7b8a5149f44226e30.js"></script>

最近看 xml 是因为我在写一个多语言自动拿出来的工具。名字还没想好，功能大概是我们在写一个Xaml ，因为之前没有想做多语言，于是我们把所有的 TextBlock 都写了 Text ，没有写 Uid，在想做多语言时，我们需要拿出所有的 Text 的文字，给每个 TextBlock 一个 Uid，写在 resw 。如果我有 100000 个 TextBlock ，那么对每个 TextBlock 的操作是拿出 Text，在资源写上 uid 和粘贴 Text，返回 TextBlock 写 Uid，大概5个操作。那么我们就需要做 500000 次。

我这个软件可以帮助大家，自动拿出 TextBlock 的Text 放在资源文件。

还没做出来，所以就不说啦。

下面是我看到的xml相关博客：

<!-- 可以看到，我们使用 StreamReader ，于是我们在里面使用  -->

[http://www.cnblogs.com/portalsky/archive/2008/09/11/1289461.html](http://www.cnblogs.com/portalsky/archive/2008/09/11/1289461.html )

[http://blog.csdn.net/cdjcong/article/details/8473539](http://blog.csdn.net/cdjcong/article/details/8473539 )

[http://blog.csdn.net/ht_zhaoliubin/article/details/38900275](http://blog.csdn.net/ht_zhaoliubin/article/details/38900275 )

[http://www.cnblogs.com/zery/p/3362480.html](http://www.cnblogs.com/zery/p/3362480.html )


关于命名空间：[https://msdn.microsoft.com/en-us/library/aa468565.aspx?f=255&MSPPError=-2147217396](https://msdn.microsoft.com/en-us/library/aa468565.aspx?f=255&MSPPError=-2147217396 )

## WPF 读XML

可以 XmlDocument 读 xml ，如果遇到命名空间问题

建议：XmlNamespaceManager 

假设一个属性存在命名控件，必须使用 XmlNamespaceManager ，如果没有，SelectSingleNode 返回空。

那么可以使用  XmlNamespaceManager ，但是需要知道 xml 的内容，因为需要拿到空间。

新建一个 XmlNamespaceManager ：


```csharp
    new XmlNamespaceManager(document.NameTable)
            {

            };
```

但是需要设置空间，`XmlNamespaceManager.AddNamespace("随意名称", NamespaceURI);`

如果看不懂上面写的，请看例子


假如要读取项目xml，也就是C#项目文件


```xml
    <?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="12.0" DefaultTargets="Build" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <Import Project="$(MSBuildExtensionsPath)\$(MSBuildToolsVersion)\Microsoft.Common.props" Condition="Exists('$(MSBuildExtensionsPath)\$(MSBuildToolsVersion)\Microsoft.Common.props')" />
</Project>
```


可以使用


```csharp
            XmlDocument document = new XmlDocument();
            document.Load("1.xml");
            var temp = new XmlNamespaceManager(document.NameTable)
            {

            };
            temp.AddNamespace("xm", document.DocumentElement.NamespaceURI);
            XmlNode root = document.SelectSingleNode("xm:Project",temp);
            XmlNode t = root.SelectSingleNode("xm:Import",temp);
```

## WPF 读写 xaml

实际上 wpf 读写和 UWP 相同，所以就不在这里多说了。

那么如何写出下面的代码

```csharp
<?xml version="1.0" encoding="utf-16"?>
<_XPXML Note="">
  <_InvTrans IC="010006" />
</_XPXML>
```

可以使用这个方法

```csharp
           XDocument doc = new XDocument();
            XElement node = new XElement("_XPXML");
            node.SetAttributeValue("Note", "");
            var invTrans = new XElement("_InvTrans");
            node.Add(invTrans);
            invTrans.SetAttributeValue("IC", "010006");

            doc.Add(node);

            StringBuilder str = new StringBuilder();
            TextWriter stream = new StringWriter(str);
            doc.Save(stream);
```




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。