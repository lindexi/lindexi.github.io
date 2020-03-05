# NuGet 命令行上传找不到 snupkg 文件

在 NuGet 提供符号 NuGet 库的支持，在默认上传将会同时上传符号库。在 NuGet 上传文件将会默认将 snupkg 符号文件上传

<!--more-->
<!-- CreateTime:2019/12/8 15:07:18 -->

<!-- 发布 -->
<!-- 标签：NuGet -->

让 NuGet 发布默认不上传符号文件的方法是添加参数 NoSymbols 请看代码

```csharp
 nuget push .\bin\release\*.nupkg -Source https://api.nuget.org/v3/index.json -SkipDuplicate -NoSymbols 
```

在 nuget 发布可以给某个文件路径，将这个路径所有文件上传，在上传文件时，将会同步上传符号文件。如果符号文件不存在，建议输出提示

```
File does not exist (.\bin\release\*.snupkg)
```

通过在命令行添加参数不上传外，还可以在创建 NuGet 库创建符号文件，这样就不会提示找不到

在 sdk style 格式的项目文件，添加下面代码，添加之后打包就会创建 snupkg 文件

```xml
<PropertyGroup>
    <IncludeSymbols>true</IncludeSymbols>	
    <SymbolPackageFormat>snupkg</SymbolPackageFormat>	
</PropertyGroup>
```

这里的 PropertyGroup 元素可以添加到 Project 元素下

另一个方法是在命令行打包添加参数

```csharp
dotnet pack -p:IncludeSymbols=true -p:SymbolPackageFormat=snupkg
```

如果使用 msbuild 打包，可以使用下面代码

```csharp
msbuild /t:pack /p:IncludeSymbols=true /p:SymbolPackageFormat=snupkg
```

如果使用 nuget 打包，如对应的 `xx.nuspec` 可以使用下面代码

```csharp
nuget pack MyPackage.nuspec -Symbols -SymbolPackageFormat snupkg
```

[NuGet 符号服务器](https://blog.lindexi.com/post/NuGet-%E7%AC%A6%E5%8F%B7%E6%9C%8D%E5%8A%A1%E5%99%A8.html )

[How to publish NuGet symbol packages using the new symbol package format '.snupkg'](https://docs.microsoft.com/en-us/nuget/create-packages/symbol-packages-snupkg )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
