本文记录在 OpenXML SDK 2.15 版本下，为 PPTX 文件添加 CoreFilePropertiesPart 的方法，通过本文的方法可以正确且简单的添加 core.xml 文件到 PPTX 文件里

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

当前的 OpenXML SDK 存在已知问题，那就是默认情况下生成不包含 core.xml 文件，请参阅 <https://github.com/dotnet/Open-XML-SDK/issues/1093>

在 OpenXML SDK 里面提供了 AddCoreFilePropertiesPart 方法可以用来添加 CoreFilePropertiesPart 的内容。但是对其调用有要求，比如说在 PresentationDocument 的 PackageProperties 属性赋值之前进行调用，如以下代码，则会出现 `System.Xml.XmlException:“Root element is missing.”` 异常

```csharp
        public void CreatePackage(string filePath)
        {
            using PresentationDocument package = PresentationDocument.Create(filePath, PresentationDocumentType.Presentation);

            var document = package;

            if (document.CoreFilePropertiesPart is null)
            {
                document.AddCoreFilePropertiesPart();
            }

            SetPackageProperties(package);
        }

        private void SetPackageProperties(OpenXmlPackage document)
        {
            document.PackageProperties.Creator = "dexi lin";
            document.PackageProperties.Title = "PowerPoint 演示文稿";
            document.PackageProperties.Revision = "1";
            document.PackageProperties.Created = System.Xml.XmlConvert.ToDateTime("2024-01-24T09:19:23Z", System.Xml.XmlDateTimeSerializationMode.RoundtripKind);
            document.PackageProperties.Modified = System.Xml.XmlConvert.ToDateTime("2024-01-24T09:19:34Z", System.Xml.XmlDateTimeSerializationMode.RoundtripKind);
            document.PackageProperties.LastModifiedBy = "dexi lin";
        }
```

以上代码进入到设置 `PackageProperties.Creator` 属性时，将会收到 `System.Xml.XmlException:“Root element is missing.”` 异常。这是因为预期的 CoreFilePropertiesPart 已经存在，但是里面没有任何内容

这时候网上的[许多方法](https://stackoverflow.com/questions/70319867/avoid-google-spreadsheet-to-convert-an-xlsx-file-created-by-open-xml-sdk-to-xlsm/70371638#70371638)都是推荐采用如下或类似的代码写入 CoreFilePropertiesPart 的内容

```csharp
            if (document.CoreFilePropertiesPart is null)
            {
                var coreFilePropertiesPart = document.AddCoreFilePropertiesPart();

                using (XmlTextWriter writer = new XmlTextWriter(coreFilePropertiesPart.GetStream(FileMode.Create), System.Text.Encoding.UTF8))
                {
                    writer.WriteRaw("<?xml version=\"1.0\" encoding=\"UTF-8\"?>\r\n<coreProperties xmlns:dc=\"http://purl.org/dc/elements/1.1/\" xmlns:dcterms=\"http://purl.org/dc/terms/\" xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\" xmlns=\"http://schemas.openxmlformats.org/package/2006/metadata/core-properties\"></coreProperties>");
                    writer.Flush();
                }
            }

            SetPackageProperties(package);
```

尽管以上代码能够正常工作，但代码不好看

如果你不小心抄错了 EXCEL 的代码，那你将会遇到 `System.Xml.XmlException:“Unrecognized root element in Core Properties part. Line 2, position 2.”` 错误哈

下面我推荐给大家一个简单的代码方法，那就是在 SetPackageProperties 完成之后再调用 AddCoreFilePropertiesPart 方法，也就是将 SetPackageProperties 方法先调用，代码如下

```csharp
        public void CreatePackage(string filePath)
        {
            using PresentationDocument package = PresentationDocument.Create(filePath, PresentationDocumentType.Presentation);

            var document = package;

            SetPackageProperties(package);

            if (document.CoreFilePropertiesPart is null)
            {
                document.AddCoreFilePropertiesPart();
            }
        }

        private void SetPackageProperties(OpenXmlPackage document)
        {
            document.PackageProperties.Creator = "dexi lin";
            document.PackageProperties.Title = "PowerPoint 演示文稿";
            document.PackageProperties.Revision = "1";
            document.PackageProperties.Created = System.Xml.XmlConvert.ToDateTime("2024-01-24T09:19:23Z", System.Xml.XmlDateTimeSerializationMode.RoundtripKind);
            document.PackageProperties.Modified = System.Xml.XmlConvert.ToDateTime("2024-01-24T09:19:34Z", System.Xml.XmlDateTimeSerializationMode.RoundtripKind);
            document.PackageProperties.LastModifiedBy = "dexi lin";
        }
```

看起来十分简单，只是将 SetPackageProperties 和 AddCoreFilePropertiesPart 的调用顺序调换即可

如果 SetPackageProperties 等是生成的代码，不想修改顺序，可以在 AddCoreFilePropertiesPart 方法调用之前，随意对 PackageProperties 的属性进行赋值，如下面代码

```csharp
        public void CreatePackage(string filePath)
        {
            using PresentationDocument package = PresentationDocument.Create(filePath, PresentationDocumentType.Presentation);

            var document = package;

            if (document.CoreFilePropertiesPart is null)
            {
                document.PackageProperties.Creator = "xxxxx"; // 随意
                document.AddCoreFilePropertiesPart();
            }

            CreateParts(package);
        }
```

通过以上的方法即可成功创建 core.xml 文件到 PPTX 文件里面。如果你使用本文的方法没有创建成功，那我推荐你使用下面的方法拉取本文的代码，跑一下代码试试

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/f4a8f9c5265f3e52f3b9f93bb6570c9e73dc41c4/WefejurkawFekejiyi) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/f4a8f9c5265f3e52f3b9f93bb6570c9e73dc41c4/WefejurkawFekejiyi) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin f4a8f9c5265f3e52f3b9f93bb6570c9e73dc41c4
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin f4a8f9c5265f3e52f3b9f93bb6570c9e73dc41c4
```

获取代码之后，进入 WefejurkawFekejiyi 文件夹

更多关于 CoreFilePropertiesPart 请参阅 ECMA 376 文档的 15.2.12.1 章内容

更多关于 OpenXML 相关知识，请参阅 [Office 使用 OpenXML SDK 解析文档博客目录](https://blog.lindexi.com/post/Office-%E4%BD%BF%E7%94%A8-OpenXML-SDK-%E8%A7%A3%E6%9E%90%E6%96%87%E6%A1%A3%E5%8D%9A%E5%AE%A2%E7%9B%AE%E5%BD%95.html )
