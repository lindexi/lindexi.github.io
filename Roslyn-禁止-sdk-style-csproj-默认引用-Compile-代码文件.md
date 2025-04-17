
# Roslyn 禁止 sdk style csproj 默认引用 Compile 代码文件

默认在 SDK Style 的 csproj 文件将会引用所有的 .cs 文件到 Compile 项，如果是 WPF 项目还会添加 xaml 的引用。如果想要自己手动设置，让一些项不默认引用，需要添加属性 EnableDefaultCompileItems 告诉 msbuild 不要默认引用

<!--more-->


<!-- CreateTime:6/18/2020 7:50:18 PM -->



<!-- 标签：Roslyn,MSBuild,编译器 -->

禁止 .cs 文件作为 Compile 的默认引用方法

```xml
<PropertyGroup>
    <EnableDefaultCompileItems>false</EnableDefaultCompileItems>
</PropertyGroup>
```

如果没有禁止，将会使用如下引用

```xml
<Compile Include="**\*.cs" />
```

禁止 xaml 文件作为 Page 的默认引用


```xml
<PropertyGroup>
    <EnableDefaultPageItems>false</EnableDefaultPageItems>
</PropertyGroup>
```

禁止创建默认特性

```xml
<PropertyGroup>
    <GenerateAssemblyInfo>false</GenerateAssemblyInfo>
</PropertyGroup>
```

关于默认特性创建请看 [解决从旧格式的 csproj 迁移到新格式的 csproj 格式 AssemblyInfo 文件值重复问题](https://blog.lindexi.com/post/%E8%A7%A3%E5%86%B3%E4%BB%8E%E6%97%A7%E6%A0%BC%E5%BC%8F%E7%9A%84-csproj-%E8%BF%81%E7%A7%BB%E5%88%B0%E6%96%B0%E6%A0%BC%E5%BC%8F%E7%9A%84-csproj-%E6%A0%BC%E5%BC%8F-AssemblyInfo-%E6%96%87%E4%BB%B6%E5%80%BC%E9%87%8D%E5%A4%8D%E9%97%AE%E9%A2%98.html )

禁止图片等作为 None 默认引用

```xml
<PropertyGroup>
    <EnableDefaultNoneItems>false</EnableDefaultNoneItems>
</PropertyGroup>
```

禁止 App.xaml 作为 ApplicationDefinition 默认引用

```xml
<PropertyGroup>
    <EnableDefaultApplicationDefinition>false</EnableDefaultApplicationDefinition>
</PropertyGroup>
```

禁止所有默认引用

```xml
<PropertyGroup>
    <EnableDefaultItems>false</EnableDefaultItems>
</PropertyGroup>
```

[从以前的项目格式迁移到 VS2017 新项目格式](https://blog.lindexi.com/post/%E4%BB%8E%E4%BB%A5%E5%89%8D%E7%9A%84%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F%E8%BF%81%E7%A7%BB%E5%88%B0-VS2017-%E6%96%B0%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F.html)

[MSBuild properties for Microsoft.NET.Sdk - .NET](https://docs.microsoft.com/en-us/dotnet/core/project-sdk/msbuild-props?WT.mc_id=WD-MVP-5003260 )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。