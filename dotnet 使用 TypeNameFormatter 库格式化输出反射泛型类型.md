# dotnet 使用 TypeNameFormatter 库格式化输出反射泛型类型

默认的反射输出带泛型的类型，都会使用反引号的字符串。使用 TypeNameFormatter 库可以输出贴近代码的输出

<!--more-->
<!-- CreateTime:2021/3/25 20:08:24 -->

<!-- 发布 -->

默认的类型的输出是和代码写的方法不相同，如获取 `List<int>` 类型的输出

```csharp
Console.WriteLine(typeof(List<int>));
// 大概输出是 List`1[System.Int32]
```

而我期望输出的是 `List<int>` 的内容，使用 TypeNameFormatter 库的代码如下

```csharp
    class Program
    {
        static void Main(string[] args)
        {
            var fType = typeof(List<F>);
            Console.WriteLine(fType.GetFormattedName());
            // 输出 List<F>
        }
    }

    class F
    {
        public string A { get; set; }
        public string B { get; set; }
        public string C { get; set; }
    }
```

使用 NuGet 搜 TypeNameFormatter 就可以找到这个库，需要加上命名空间

```csharp
using TypeNameFormatter;
```

编辑 csproj 添加下面代码可以快速安装

```xml
  <ItemGroup>
    <PackageReference Include="TypeNameFormatter.Sources" Version="1.1.1">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
  </ItemGroup>
```

这是一个源代码包，按照之后，可以在 obj 文件夹找到代码。或者进入 GetFormattedName 方法定义，可以看到代码

这个库在 GitHub 开源，请看 [https://github.com/stakx/TypeNameFormatter](https://github.com/stakx/TypeNameFormatter)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
