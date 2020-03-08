# C# 在 8.0 对比 string 和 string? 的类型

在 C# 8.0 的时候提供了可空字符串的判断，但是可空字符串和字符串的类型是不是不同的？

<!--more-->
<!-- CreateTime:2019/11/29 8:59:00 -->


打开 VisualStudio 2019 这时就不能再使用 VisualStudio 2017 因为不支持

然后创建一个 dotnet core 项目，打开项目文件添加下面代码

```csharp
<Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>netcoreapp3.0</TargetFramework>
        <LangVersion>8.0</LangVersion>
        <NullableReferenceTypes>true</NullableReferenceTypes>
    </PropertyGroup>

</Project>

```

先详细介绍每一句话的意思

第一句 OutputType 的意思是输出是什么，这里选 Exe 就是输出控制台

通过 TargetFramework 可以设置平台，更多可以设置请看 [从以前的项目格式迁移到 VS2017 新项目格式](https://blog.lindexi.com/post/%E4%BB%8E%E4%BB%A5%E5%89%8D%E7%9A%84%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F%E8%BF%81%E7%A7%BB%E5%88%B0-VS2017-%E6%96%B0%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F.html )

在 LangVersion 设置使用的语言版本，详细请看 [VisualStudio 使用三个方法启动最新 C# 功能](https://lindexi.gitee.io/post/VisualStudio-%E4%BD%BF%E7%94%A8%E4%B8%89%E4%B8%AA%E6%96%B9%E6%B3%95%E5%90%AF%E5%8A%A8%E6%9C%80%E6%96%B0-C-%E5%8A%9F%E8%83%BD.html )

最后通过 NullableReferenceTypes 开启可空类型的判断

现在开始试试之前的其他可空的方法，如下面代码

```csharp
            Console.WriteLine(typeof(int).FullName);
            Console.WriteLine(typeof(int?).FullName);
```

小伙伴都知道输出的 int 和 `int?` 是不同的

```csharp
System.Int32
System.Nullable`1[[System.Int32, System.Private.CoreLib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=7cec85d7bea7798e]]
```

但是如果是 string 和 `string?` 的比较呢？

判断 `typeof(string) == typeof(string?)` 是不是相同的？

```csharp
            System.Console.WriteLine(typeof(string).FullName);
            Console.WriteLine(typeof(string?).FullName);
            Console.WriteLine(typeof(string) == typeof(string?));
```

其实在 C# 8.0 的 `string?` 是糖也就是实际不存在的，对于 `string?` 的类和 string 相同

```csharp
System.String
System.String
True
```

[VisualStudio 2019 尝试使用 C# 8.0 新的方式](https://blog.lindexi.com/post/VisualStudio-2019-%E5%B0%9D%E8%AF%95%E4%BD%BF%E7%94%A8-C-8.0-%E6%96%B0%E7%9A%84%E6%96%B9%E5%BC%8F.html )

![](http://image.acmx.xyz/lindexi%2F2019416101647227)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
