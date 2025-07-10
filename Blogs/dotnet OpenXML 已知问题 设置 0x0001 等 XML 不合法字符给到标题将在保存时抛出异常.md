---
title: dotnet OpenXML 已知问题 设置 0x0001 等 XML 不合法字符给到标题将在保存时抛出异常
description: 本文记录 OpenXML 的已知问题，在 `PackageProperties.Title` 等属性上设置字符串包含如 Unicode 编码为 0x0001 等 XML 不合法字符，将在保存写入文件时，抛出 ArgumentException 异常
tags: dotnet,OpenXML
category: 
---

<!-- CreateTime:2025/02/22 07:21:24 -->

<!-- 发布 -->
<!-- 博客 -->

此问题报告给了官方： <https://github.com/dotnet/Open-XML-SDK/issues/1874>

最简复现代码：

```csharp
using System;

using (var presentationDocument =
       DocumentFormat.OpenXml.Packaging.PresentationDocument.Open("Test.pptx", true))
{
    presentationDocument.PackageProperties.Title = "\u0001";
}
```

以上的 Test.pptx 是任意一份正常的 PPTX 文件

运行以上代码，可以看到如下异常

```
 System.ArgumentException:“'', hexadecimal value 0x01, is an invalid character.”
  	 System.Private.Xml.dll!System.Xml.XmlUtf8RawTextWriter.InvalidXmlChar(int ch, byte* pDst, bool entitize)	
  	 System.Private.Xml.dll!System.Xml.XmlUtf8RawTextWriter.WriteElementTextBlock(char* pSrc, char* pSrcEnd)	
  	 System.Private.Xml.dll!System.Xml.XmlUtf8RawTextWriter.WriteString(string text)	
  	 System.Private.Xml.dll!System.Xml.XmlWellFormedWriter.WriteString(string text)	
  	 System.IO.Packaging.dll!System.IO.Packaging.PartBasedPackageProperties.SerializeDirtyProperties()	
  	 System.IO.Packaging.dll!System.IO.Packaging.PartBasedPackageProperties.Flush()	
  	 System.IO.Packaging.dll!System.IO.Packaging.PartBasedPackageProperties.Close()	
  	 System.IO.Packaging.dll!System.IO.Packaging.Package.System.IDisposable.Dispose()	
  	 System.IO.Packaging.dll!System.IO.Packaging.Package.Close()	
  	 DocumentFormat.OpenXml.Framework.dll!DocumentFormat.OpenXml.Features.StreamPackageFeature.Dispose(bool disposing)	
  	 DocumentFormat.OpenXml.Framework.dll!DocumentFormat.OpenXml.Features.StreamPackageFeature.Dispose()	
  	 DocumentFormat.OpenXml.Framework.dll!DocumentFormat.OpenXml.Packaging.PackageFeatureCollection.DocumentFormat.OpenXml.Features.IContainerDisposableFeature.Dispose()	
  	 DocumentFormat.OpenXml.Framework.dll!DocumentFormat.OpenXml.Packaging.OpenXmlPackage.Dispose(bool disposing)	
  	 DocumentFormat.OpenXml.Framework.dll!DocumentFormat.OpenXml.Packaging.OpenXmlPackage.Dispose()	
 >	 FukemqibairLaylalljerowhem.dll!Program.<Main>$(string[] args)
```

本文的测试代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/6baba8b5407c1d23119d3ac150b1ab5af4cd810c/Pptx/FukemqibairLaylalljerowhem) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/6baba8b5407c1d23119d3ac150b1ab5af4cd810c/Pptx/FukemqibairLaylalljerowhem) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 6baba8b5407c1d23119d3ac150b1ab5af4cd810c
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 6baba8b5407c1d23119d3ac150b1ab5af4cd810c
```

获取代码之后，进入 Pptx/FukemqibairLaylalljerowhem 文件夹，即可获取到源代码

这个异常是在更底层的 dotnet runtime 里面抛出的，详细请看 <https://github.com/dotnet/runtime/blob/c1fe87ad88532f0e80de3739fe7b215e6e1f8b90/src/libraries/System.Private.Xml/src/System/Xml/Core/XmlUtf8RawTextWriter.cs#L932>

可选规避的方法是写一个过滤方法，过滤掉 XML 不合法字符，以下是我给出的过滤代码

```csharp
internal static class XmlSafeTextContentHelper
{
    public static string ToSafeXmlText(string text, char replaceChar = '_')
    {
        // 第一遍是跑一下，因为正常是不会有奇怪的字符的，那就进入快速分支
        // 如果能够找到首个奇怪的字符，则进入慢分支，重新拼装替代字符

        var index = -1;
        for (var i = 0; i < text.Length; i++)
        {
            var c = text[i];
            if (!XmlConvert.IsXmlChar(c))
            {
                index = i;
                break;
            }
        }

        var canFindInvalidChar = index >= 0;
        if (canFindInvalidChar)
        {
            // 慢分支，开始重新拼装字符串
            var stringBuilder = new StringBuilder(text.Length);
            stringBuilder.Append(text, 0, index);
            for (var i = index; i < text.Length; i++)
            {
                var c = text[i];
                if (XmlConvert.IsXmlChar(c))
                {
                    stringBuilder.Append(c);
                }
                else
                {
                    stringBuilder.Append(replaceChar);
                }
            }

            return stringBuilder.ToString();
        }
        else
        {
            return text;
        }
    }
}
```

确保给到 PackageProperties 等属性的字符串都是经过过滤的。另一个过滤方法是通过 XmlConvert.EncodeName 和 XmlConvert.DecodeName 进行过滤。在本文这里不使用 XmlConvert.EncodeName 的原因只是因为如 Title 等内容，经过 XmlConvert.EncodeName 之后可读性会降低，比如一个下划线会被转换为两个下划线等

更多技术博客，请参阅 [Office 使用 OpenXML SDK 解析文档博客目录](https://blog.lindexi.com/post/Office-%E4%BD%BF%E7%94%A8-OpenXML-SDK-%E8%A7%A3%E6%9E%90%E6%96%87%E6%A1%A3%E5%8D%9A%E5%AE%A2%E7%9B%AE%E5%BD%95.html )

<!-- 

Setting `PackageProperties.Title` with 0x0001 Unicode Characters in OpenXML Results in an Error


When attempting to configure the `PackageProperties.Title` attribute of a `PresentationDocument` within OpenXML, inserting a string containing the Unicode character `\u0001` (often known as a soft hyphen) generates an `ArgumentException`. This occurs despite the presence of other valid properties.

```csharp
using System;

using (var presentationDocument =
       DocumentFormat.OpenXml.Packaging.PresentationDocument.Open("Test.pptx", true))
{
    presentationDocument.PackageProperties.Title = "\u0001";
}
```

```
 System.ArgumentException:“'', hexadecimal value 0x01, is an invalid character.”
  	 System.Private.Xml.dll!System.Xml.XmlUtf8RawTextWriter.InvalidXmlChar(int ch, byte* pDst, bool entitize)	
  	 System.Private.Xml.dll!System.Xml.XmlUtf8RawTextWriter.WriteElementTextBlock(char* pSrc, char* pSrcEnd)	
  	 System.Private.Xml.dll!System.Xml.XmlUtf8RawTextWriter.WriteString(string text)	
  	 System.Private.Xml.dll!System.Xml.XmlWellFormedWriter.WriteString(string text)	
  	 System.IO.Packaging.dll!System.IO.Packaging.PartBasedPackageProperties.SerializeDirtyProperties()	
  	 System.IO.Packaging.dll!System.IO.Packaging.PartBasedPackageProperties.Flush()	
  	 System.IO.Packaging.dll!System.IO.Packaging.PartBasedPackageProperties.Close()	
  	 System.IO.Packaging.dll!System.IO.Packaging.Package.System.IDisposable.Dispose()	
  	 System.IO.Packaging.dll!System.IO.Packaging.Package.Close()	
  	 DocumentFormat.OpenXml.Framework.dll!DocumentFormat.OpenXml.Features.StreamPackageFeature.Dispose(bool disposing)	
  	 DocumentFormat.OpenXml.Framework.dll!DocumentFormat.OpenXml.Features.StreamPackageFeature.Dispose()	
  	 DocumentFormat.OpenXml.Framework.dll!DocumentFormat.OpenXml.Packaging.PackageFeatureCollection.DocumentFormat.OpenXml.Features.IContainerDisposableFeature.Dispose()	
  	 DocumentFormat.OpenXml.Framework.dll!DocumentFormat.OpenXml.Packaging.OpenXmlPackage.Dispose(bool disposing)	
  	 DocumentFormat.OpenXml.Framework.dll!DocumentFormat.OpenXml.Packaging.OpenXmlPackage.Dispose()	
 >	 FukemqibairLaylalljerowhem.dll!Program.<Main>$(string[] args)
```

https://github.com/lindexi/lindexi_gd/tree/6baba8b5407c1d23119d3ac150b1ab5af4cd810c/Pptx/FukemqibairLaylalljerowhem
 -->
