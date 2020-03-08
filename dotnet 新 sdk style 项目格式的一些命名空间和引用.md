# dotnet 新 sdk style 项目格式的一些命名空间和引用

在使用 sdk style 的 csproj 项目格式，会发现右击引用找不到程序集，此时有一些命名空间没有找到。本文收集一些命名空间所在的引用

<!--more-->
<!-- CreateTime:2020/1/3 15:12:56 -->

<!-- cdsn -->

## System.Net.Http

引用方法

```xml
  <ItemGroup Condition="$(TargetFramework)=='net45'">
    <Reference Include="System.Net.Http"></Reference>
  </ItemGroup>
```

可以修复在新 sdk style 的 csproj 项目格式找不到 `System.Net.Http` 命名空间

包含类有

- System.Net.Http.HttpClient
- System.Net.Http.HttpMethod

## System.Web

引用方法

```xml
  <ItemGroup Condition="$(TargetFramework)=='net45'">
    <Reference Include="System.Web"></Reference>
  </ItemGroup>
```

包含以下命名空间

- System.Web.HttpUtility


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
