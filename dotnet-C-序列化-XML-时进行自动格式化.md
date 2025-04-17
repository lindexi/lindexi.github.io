
# dotnet C# 序列化 XML 时进行自动格式化

默认的序列化对象为 XML 字符串时，是没有进行格式化的，也就是所有的内容都在相同的一行。本文告诉大家方法，在序列化对象时，转换的 XML 是格式化的。或者说拿到 XML 字符串，对这个 XML 字符串进行格式化

<!--more-->


<!-- CreateTime:2021/3/6 10:56:50 -->

<!-- 发布 -->

在序列化某个对象的时候，序列化出来的 XML 进行格式化，可以在创建的 XmlWriter 传入 XmlWriterSettings 设置 Indent 属性，如下面代码

```csharp
        public static string Serialize<T>(T obj)
        {
            var ns = new XmlSerializerNamespaces();
            ns.Add("", "");

            var xmlSerializer = new XmlSerializer(typeof(T));
            var stringBuilder = new StringBuilder();

            using (var xmlWriter = XmlWriter.Create(new StringWriter(stringBuilder), new XmlWriterSettings
                {
                    Indent = true
                }))
            {
                xmlSerializer.Serialize(xmlWriter, obj, ns);
            }

            return stringBuilder.ToString();
        }
```

上面代码的 XmlSerializerNamespaces 是为了去掉 XML 的命名空间

此时序列化出来的内容将会是格式化的

对于现有的 XML 文件或 XML 字符串，进行格式化，方法是先读取出来，然后再次写入，写入时设置格式化

```csharp
                    var xmlString = File.ReadAllText(xmlFile);
                    XmlDocument document = new XmlDocument();
                    document.LoadXml(xmlString);

                    using var fileStream = new FileStream(xmlFile, FileMode.Create, FileAccess.Write);
                    fileStream.SetLength(0);

                    using var xmlWriter = XmlWriter.Create(fileStream, new XmlWriterSettings()
                    {
                        Indent = true
                    });
                    document.WriteTo(xmlWriter);
```

上面代码的 xmlFile 是一个存放在本地的文件





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。