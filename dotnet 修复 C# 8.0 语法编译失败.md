# dotnet 修复 C# 8.0 语法编译失败

在使用 using 等新语法时，在 VisualStudio 2019 会自动判断框架版本，如在 net 45 就不会自动使用最新版本的语法，需要修改项目文件

<!--more-->
<!-- CreateTime:2020/2/1 16:59:41 -->

<!-- 发布 -->

在使用 C# 8.0 之前，请在[官网](https://visualstudio.microsoft.com/) 下载最新的 VisualStudio 2019 版本

如果在编译时提示

```
“Using 声明”在 C# 7.3 中不可用。请使用 8.0 或更高的语言版本
```

或

```
error CS8370: 功能“可为 null 的引用类型”在 C# 7.3 中不可用。请使用 8.0 或更高的语言版本。
```

解决方法是在 csproj 项目文件里面添加下面代码

```

  <PropertyGroup>
    <LangVersion>preview</LangVersion>
  </PropertyGroup>


```

如果不知道写在哪，请看 csproj 文件

```
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net45</TargetFramework>
    <LangVersion>preview</LangVersion>
  </PropertyGroup>

</Project>

```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
