# dotnet 新 sdk style 项目格式 文件冲突

在使用 dotnet 的 sdk style 项目格式，会默认在项目上引用文件，此时如果是从旧格式迁移，那么会发现文件冲突。多次引用相同文件

<!--more-->
<!-- CreateTime:2019/12/7 8:55:34 -->

<!-- csdn -->

在 [从以前的项目格式迁移到 VS2017 新项目格式](https://blog.lindexi.com/post/%E4%BB%8E%E4%BB%A5%E5%89%8D%E7%9A%84%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F%E8%BF%81%E7%A7%BB%E5%88%B0-VS2017-%E6%96%B0%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F.html ) 告诉大家如何迁移，但是迁移完成会发现有文件冲突

如 cs 文件重复，默认的格式会引用所有的 `*.cs` 文件，如果此时在项目中因为有一些文件是排除文件，不能直接使用 `*.cs` 需要写引用的是哪个文件，在编译会发现文件被多次引用，此时可以在 PropertyGroup 添加下面代码

```csharp
    <EnableDefaultCompileItems>false</EnableDefaultCompileItems>
```
	
如果提示嵌入的资源冲突了，也就是添加了默认的嵌入资源。默认添加的嵌入资源是按照文件后缀名添加，我的项目对这些后缀名的文件是不需要加入的，迁移项目格式就加入，编译的文件就比原来大。可以添加下面代码，不加入默认的文件

```csharp
    <EnableDefaultEmbeddedResourceItems>false</EnableDefaultEmbeddedResourceItems>
```

对于 WPF 项目，默认的 Page 文件也会被加入，可以通过下面代码解决

```csharp
    <EnableDefaultPageItems>false</EnableDefaultPageItems>
```

大概的项目需要添加下面代码

```csharp
 <PropertyGroup>
    <TargetFramework>net47</TargetFramework>
    <OutputType>Library</OutputType>
    <RootNamespace>lindexi</RootNamespace>
    <AssemblyName>lindexi</AssemblyName>

    <GenerateAssemblyInfo>false</GenerateAssemblyInfo>
    <EnableDefaultCompileItems>false</EnableDefaultCompileItems>
    <EnableDefaultPageItems>false</EnableDefaultPageItems>
    <EnableDefaultEmbeddedResourceItems>false</EnableDefaultEmbeddedResourceItems>
  </PropertyGroup>
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
