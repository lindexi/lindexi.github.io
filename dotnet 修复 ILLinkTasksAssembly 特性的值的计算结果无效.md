# dotnet 修复 ILLinkTasksAssembly 特性的值的计算结果无效

在加上 IL Link 的项目里面，在升级到 .NET 6 预览版，有一些项目将会构建不通过，或者有些 C++ CLI 项目加载失败。提示 元素 UsingTask 中“AssemblyFile”特性的值“$(ILLinkTasksAssembly)”的计算结果“”无效。原因就是 .NET 6 预览版里面，或者自己的设备上 ILLinkTasksAssembly 属性定义失效

<!--more-->
<!-- CreateTime:2021/4/6 8:34:27 -->

<!-- 发布 -->

解决方法是先创建一个空白项目，找找自己本地的 ILLinkTasksAssembly 定义是否存在，默认在 SDK 里面是存在 Microsoft.NET.ILLink.Tasks 的。如果发现自己的设备上不存在 Microsoft.NET.ILLink.Tasks 这个文件夹，那么请将 dotnet sdk 卸载重新安装，或者安装更新版本的 sdk 然后查看自己的环境变量，是否有设置特定版本的 dotnet sdk 如果有设置，就删除此项值或者修改为更新版本

创建空白项目，找找自己本地的 ILLinkTasksAssembly 定义的做法是在空项目的 csproj 里面添加如下代码，用来输出 IL Link 的路径

```xml
  <Target Name="Foo" BeforeTargets="BeforeBuild">
    <Warning Text="$(ILLinkTasksAssembly)"/>
  </Target>
```

此时的空白项目的 csproj 内容大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net5.0</TargetFramework>
  </PropertyGroup>

  <Target Name="Foo" BeforeTargets="BeforeBuild">
    <Warning Text="$(ILLinkTasksAssembly)"/>
  </Target>
</Project>
```

此时构建这个空白项目，可以看到输出的警告，警告里面就是 IL Link 的路径。如 `C:\Program Files\dotnet\sdk\6.0.100-preview.2.21155.3\Sdks\Microsoft.NET.ILLink.Tasks\Sdk\..\tools\net472\ILLink.Tasks.dll` 的路径

在自己构建失败的项目，或者加载失败的 C++\CLI 项目的项目文件里面，在 PropertyGroup 里面添加如下代码

```xml
<ILLinkTasksAssembly>C:\Program Files\dotnet\sdk\6.0.100-preview.2.21155.3\Sdks\Microsoft.NET.ILLink.Tasks\Sdk\..\tools\net472\ILLink.Tasks.dll</ILLinkTasksAssembly>
```

在 PropertyGroup 添加如上代码的效果如下

```xml
  <PropertyGroup>
    <ILLinkTasksAssembly>C:\Program Files\dotnet\sdk\6.0.100-preview.2.21155.3\Sdks\Microsoft.NET.ILLink.Tasks\Sdk\..\tools\net472\ILLink.Tasks.dll</ILLinkTasksAssembly>
  </PropertyGroup>
```

请将上面的路径替换为你自己本地的路径，大概此时就能修改构建或项目加载。本文的方法能修复的是在构建和加载项目提示如下内容

```
 error  : 元素 <UsingTask> 中“AssemblyFile”特性的值“$(ILLinkTasksAssembly)”的计算结果“”无效。  C:\Program Files\dotnet\sdk\6.0.100-preview.2.21155.3\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.ILLink.targets
```

在 Microsoft.NET.ILLink.targets 文件可以看到如下定义

```xml
 <UsingTask TaskName="ILLink" AssemblyFile="$(ILLinkTasksAssembly)" />
```

上面提示内容就是 `$(ILLinkTasksAssembly)` 没有定义，拿到空值。因此自己在项目里面加上定义是可以解决此问题

本文的空项目代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/ffe013bf/JoruneecijerDaryiqikuhakuye ) 欢迎小伙伴访问

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
