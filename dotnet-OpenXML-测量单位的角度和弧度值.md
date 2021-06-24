
# dotnet OpenXML 测量单位的角度和弧度值

在 OpenXML 中表示的角度和咱日常使用的角度不相同，而在 .NET 里面的 Math 函数里面使用的是弧度表示，此时就需要有一些转换。本文来告诉大家一些概念，让大家明白角度和弧度的差别

<!--more-->


<!-- 发布 -->

其实在英文语境里面，可以采用 Angle 和 Radians 和 Degree 分开，不过在中文里面，咱使用角度代表一切，同时也用角度代表 0-360 度的角度值。而弧度特别指的是 0-2 π 范围的弧度的值

英文里面的 Radians 表示的是中文的弧度，也就是 0-2 π 范围的弧度的值

英文里面的 Degree 表示的中文特指 0-360° 的角度值

在 OpenXML SDK 里面，采用的基础单位是 60000 倍的 Degree 角度值，也就是在获取到 OpenXML 的 Int32Value 时，获取数值，除以 60000 就拿到了角度值

将角度 Degree 转换为弧度，可以采用如下公式

```csharp
Radians = Degree / 180 * Math.PI;
```

在 .NET 里面的 Math 系列函数，如 Sin 等函数，传入的参数要求使用的是 Radians 弧度表示

因此在获取到 OpenXML SDK 的角度值的时候，需要进行两步转换才能在 .NET 的 Math 进行转换，第一步是除以 60000 就拿到了角度值，第二步是将角度转换为弧度值

更多请看 [Office Open XML 的测量单位](https://blog.lindexi.com/post/Office-Open-XML-%E7%9A%84%E6%B5%8B%E9%87%8F%E5%8D%95%E4%BD%8D.html )

关于 OpenXML 的单位，我写了一个库用来做转换，请看 [dotnetCampus.OpenXMLUnitConverter](https://github.com/dotnet-campus/dotnetCampus.OfficeDocumentZipper)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。