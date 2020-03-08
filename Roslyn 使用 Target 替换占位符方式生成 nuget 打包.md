# Roslyn 使用 Target 替换占位符方式生成 nuget 打包

本文告诉大家如何编写在编译过程修改打包文件

<!--more-->
<!-- CreateTime:2019/7/29 10:01:18 -->

<!-- 标签：Roslyn,MSBuild,编译器,nuget,打包 -->

在项目文件的相同文件夹可以放一个 nuspec 用来告诉 VisualStudio 如何打包

现在尝试创建一个项目 NearjerbetearDeeyitoo ，在这个项目用来告诉大家如何使用替换占位符的方法

在开始做之前需要告诉大家为什么需要使用这个方法

因为写的 nuspec 文件是可以保持不动，在多个项目使用相同的一个 nuspec 文件，但是对不同的项目使用定制的方式，让项目自己输入在编译才能知道的变量，这样可以保持不修改 nusec 文件。

先来创建一个 nuspec 文件，把这个文件随意一个文件名`ReresouJesou.nuspec`，如果在 VisualStudio 使用某个 nuspec 文件打包，就需要在项目文件添加下面代码

```csharp
    <NuspecFile>ReresouJesou.nuspec</NuspecFile>

```

为了在之后的打包过程可以添加一些变量，就需要修改项目文件

```csharp
  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
    <PackageId>NearjerbetearDeeyitoo</PackageId>
    <NuspecFile>ReresouJesou.nuspec</NuspecFile>
    <IsTool>True</IsTool>
  </PropertyGroup>
```

需要稍微解释一下上面的代码，这里的 `PackageId` 实际上是我随意给的，大家可以替换`PackageId`为自己喜欢的字符串。在`NuspecFile`就需要指定`nuspec`文件所在的路径，这里用的是相对的路径。最后设置`IsTool`只是用来告诉安装 Nuget 的程序，这是一个工具 nuget 包没有引用。

现在修改一下 `ReresouJesou.nuspec` 文件，添加下面代码

```csharp
<?xml version="1.0" encoding="utf-8"?>
<package xmlns="http://schemas.microsoft.com/packaging/2012/06/nuspec.xsd">
    <metadata>
        <id>$id$</id>
        <version>$version$</version>
        <authors>lindexi</authors>
        <owners>lindexi</owners>
        <developmentDependency>true</developmentDependency>
        <projectUrl>https://lindexi.github.io/lindexi</projectUrl>
        <description>这个文件告诉大家如何在编译修改占位字符</description>
    </metadata>
</package>
```

可以从上面代码看到和普通的 nuget 文件的不相同，第一个是`id`使用的是`$id$` ，这里的id就是使用占位符，可以在项目文件使用 target 的方式替换占位符。

上面代码有 id 和版本都使用占位符，下面就来写 target 来替换两个占位符为项目需要的字符。

```csharp
  <Target Name="GenerateNuspecProperties" BeforeTargets="GenerateNuspec">
    <PropertyGroup>
      <NuspecProperties>
        $(NuspecProperties);
        id=$(PackageId);
        version=1.0;
        dll=$(TargetPath);
      </NuspecProperties>
    </PropertyGroup>
  </Target>
```

写一个 Target 需要给这个 Target 一个命名，还需要告诉 VisualStudio 在什么的时候才使用这个 Target 这里是在创建 nuget 文件的时候才使用。

这里通过定义 nuget 属性的方式用来替换。

替换的语法是 `占位符 = 字符串;` 的方法，因为这里的字符串可以使用 `$(变量)` 的方式，所以就可以用到刚才在上面定义的字符串。

在属性的`$(NuspecProperties);`就是在有其他的 target 也使用了 `NuspecProperties` 不会被这个 target 覆盖。从上面的代码可以看到我多设置了一个`dll`的字符串，在nuget文件是不存在这个`dll`字符串，但是也没有问题。

但是可以多设置 nuget 文件不使用的字符串，不可以少设置 nuget 文件存在的字符串，不然就可能出现下面的代码

```csharp
错误		值不能为 null 或空字符串。
	NuGet.Build.Tasks.Pack\build\NuGet.Build.Tasks.Pack.targets	

```

如何写 target 请看 [如何编写基于 Microsoft.NET.Sdk 的跨平台的 MSBuild Target（附各种自带的 Task） - walterlv](https://walterlv.github.io/post/write-msbuild-target.html )

更多关于 Roslyn 请看 [手把手教你写 Roslyn 修改编译](https://lindexi.oschina.io/lindexi/post/roslyn.html ) 

参见：[专栏：Roslyn 入门 - CSDN博客](https://blog.csdn.net/column/details/23159.html )

[NuGet pack and restore as MSBuild targets](https://docs.microsoft.com/en-us/nuget/reference/msbuild-targets )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  

