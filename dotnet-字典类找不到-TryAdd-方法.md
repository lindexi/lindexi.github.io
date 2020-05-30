
# dotnet 字典类找不到 TryAdd 方法

我在给 dotnet 的 runtime 仓库提PR时，小伙伴告诉我可以使用 TryAdd 方法减少判断，但是我修改这个代码发现 100 个自动化测试都失败了，都告诉我没有找到这个方法

<!--more-->


<!-- 发布 -->

在这个更改 [https://github.com/dotnet/runtime/pull/37041](https://github.com/dotnet/runtime/pull/37041) 有小伙伴告诉我可以使用 TryAdd 方法减少判断，我添加之后发现差不多 100 个自动化测试都失败，提示下面代码


```
'Dictionary<string, string>' does not contain a definition for 'TryAdd' and no accessible extension method 'TryAdd' accepting a first argument of type
```

原因是 Microsoft.Extensions.Configuration.CommandLine 这个库使用了 dotnet standard 2.0 版本

```
  <PropertyGroup>
    <TargetFrameworks>netstandard2.0;</TargetFrameworks>
    <EnableDefaultItems>true</EnableDefaultItems>
  </PropertyGroup>
```

而 TryAdd 方法是在 .NET Standard 2.1 才添加的，也就是 2.0 是没有这个方法

[Dictionary<TKey,TValue>.TryAdd(TKey, TValue) Method (System.Collections.Generic)](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2.tryadd )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。