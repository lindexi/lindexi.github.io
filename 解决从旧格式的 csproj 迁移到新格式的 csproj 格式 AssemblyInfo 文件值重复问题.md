# 解决从旧格式的 csproj 迁移到新格式的 csproj 格式 AssemblyInfo 文件值重复问题

现在很多小伙伴开始使用了 dotnet core 项目，但是如果是从以前的 dotnet framework 的项目修改为 dotnet core 项目格式，会发现编译的时候出现了 AssemblyInfo 里面的很多值重复

<!--more-->
<!-- CreateTime:2019/11/29 8:58:48 -->


<!-- 标签：VisualStudio -->

如果直接修改格式，没有删除 AssemblyInfo 文件，很多时候会发现编译的时候出现下面提示

```csharp
    Error CS0579: “System.Reflection.AssemblyCompanyAttribute”特性重复 
    Error CS0579: “System.Reflection.AssemblyConfigurationAttribute”特性重复 
    Error CS0579: “System.Reflection.AssemblyFileVersionAttribute”特性重复 
    Error CS0579: “System.Reflection.AssemblyProductAttribute”特性重复 
    Error CS0579: “System.Reflection.AssemblyTitleAttribute”特性重复 
    Error CS0579: “System.Reflection.AssemblyVersionAttribute”特性重复 
```

修复编译时候提示项目文件特性重复有两个方法可以修复

## 删除重复的特性

打开 AssemblyInfo 文件，几乎可以删除里面的所有代码，除了 ComVisible 和 ThemeInfo 和其他自己添加的代码，其他都可以删除

```csharp
using System.Runtime.InteropServices;
using System.Windows;


// 将 ComVisible 设置为 false 会使此程序集中的类型
//对 COM 组件不可见。如果需要从 COM 访问此程序集中的类型
//请将此类型的 ComVisible 特性设置为 true。
[assembly: ComVisible(false)]

//若要开始生成可本地化的应用程序，请设置
//.csproj 文件中的 <UICulture>CultureYouAreCodingWith</UICulture>
//例如，如果您在源文件中使用的是美国英语，
//使用的是美国英语，请将 <UICulture> 设置为 en-US。  然后取消
//对以下 NeutralResourceLanguage 特性的注释。  更新
//以下行中的“en-US”以匹配项目文件中的 UICulture 设置。

//[assembly: NeutralResourcesLanguage("en-US", UltimateResourceFallbackLocation.Satellite)]


[assembly: ThemeInfo(
    ResourceDictionaryLocation.None, //主题特定资源词典所处位置
                                     //(未在页面中找到资源时使用，
                                     //或应用程序资源字典中找到时使用)
    ResourceDictionaryLocation.SourceAssembly //常规资源词典所处位置
                                              //(未在页面中找到资源时使用，
                                              //、应用程序或任何主题专用资源字典中找到时使用)
)]

```

## 不自动创建 AssemblyInfo 特性

在新的 dotnet core 格式，默认会自动创建 AssemblyInfo 特性，编译不通过的原因是存在 AssemblyInfo 文件和使用 dotnet core 项目格式创建的 AssemblyInfo 特性除了删除 AssemblyInfo 文件还可以让 dotnet core 项目格式不要创建

通过在 csproj 添加下面代码可以不创建

```csharp
<GenerateAssemblyInfo>false</GenerateAssemblyInfo>
```

上面这个代码需要放在 PropertyGroup 里，请看下面

```csharp
    <PropertyGroup>
        <OutputType>WinExe</OutputType>
        <TargetFramework>net472</TargetFramework>
        <UseWPF>true</UseWPF>
        <GenerateAssemblyInfo>false</GenerateAssemblyInfo>
    </PropertyGroup>
```

这里有一个好用的迁移工具 [CsprojToVs2017 Tooling for converting pre 2017 project to the new Visual Studio 2017 format.](https://github.com/hvanbakel/CsprojToVs2017 ) 可以将之前的 csporj 格式修改为新的 csproj 格式

[从以前的项目格式迁移到 VS2017 新项目格式](https://blog.lindexi.com/post/%E4%BB%8E%E4%BB%A5%E5%89%8D%E7%9A%84%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F%E8%BF%81%E7%A7%BB%E5%88%B0-VS2017-%E6%96%B0%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
