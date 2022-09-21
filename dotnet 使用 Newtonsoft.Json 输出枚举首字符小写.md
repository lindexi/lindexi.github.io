# dotnet 使用 Newtonsoft.Json 输出枚举首字符小写

本文告诉大家如何使用 Newtonsoft.Json 输出枚举首字符小写

<!--more-->
<!-- CreateTime:2021/11/19 19:05:42 -->

<!-- 发布 -->

实现方法是加上 JsonConverterAttribute 特性，传入 StringEnumConverter 转换器，再加上参数设置首字符小写

如下面代码

```csharp
class F1
{
    [JsonConverter(typeof(StringEnumConverter), true)]
    public Foo Foo { get; set; }
}

enum Foo
{
    Axx,
    AxxBxx,
}
```

在使用 StringEnumConverter 时，可以通过构造传入参数，设置是否使用 camelCase 风格。传入参数时，可以在 JsonConverterAttribute 特性上，加上参数

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
