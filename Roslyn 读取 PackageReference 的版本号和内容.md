# Roslyn 读取 PackageReference 的版本号和内容

在写 msbuild 的预编译逻辑，如果想要拿到项目安装的 NuGet 库和版本，可以通过获取 PackageReference 的方法获取

<!--more-->
<!-- 发布 -->

获取方法是放在 Target 里面，如下面代码

```xml
    <Target Name="LabaijalacarDaqarheelu" BeforeTargets="Build">
        <ItemGroup>
            <PackageReferenceVersion Include="Name='%(PackageReference.Identity)' Version='%(PackageReference.Version)' PrivateAssets='%(PackageReference.PrivateAssets)'"></PackageReferenceVersion>
        </ItemGroup>
        <Warning Text="@(PackageReferenceVersion)"/>
    </Target>
```

上面代码使用 PackageReferenceVersion 获取项目所有的 PackageReference 然后使用 Warning 输出

为什么使用 Warning 输出，因为这样调试可以看到，默认的 dotnet 工具输出的 Waring 等级，关于 dotnet 的输出，请看 [How to output the target message in dotnet build command line](https://blog.lindexi.com/post/How-to-output-the-target-message-in-dotnet-build-command-line.html)

另外上面的获取 ItemGroup 的某个 Item 的属性使用的是 [如何在 MSBuild 中正确使用 % 来引用每一个项（Item）中的元数据](https://blog.walterlv.com/post/how-to-reference-msbuild-item-metadata.html)

项目文件的代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <TargetFramework>netcoreapp3.1</TargetFramework>
    </PropertyGroup>
    <ItemGroup>
        <PackageReference Include="NewLife.RocketMQ" Version="1.3.2020.802" />
        <PackageReference Include="Microsoft.Extensions.Logging.Abstractions" Version="3.1.6" PrivateAssets="all"></PackageReference>
    </ItemGroup>
    <Target Name="LabaijalacarDaqarheelu" BeforeTargets="Build">
        <ItemGroup>
            <PackageReferenceVersion Include="Name='%(PackageReference.Identity)' Version='%(PackageReference.Version)' PrivateAssets='%(PackageReference.PrivateAssets)'"></PackageReferenceVersion>
        </ItemGroup>
        <Warning Text="@(PackageReferenceVersion)"/>
    </Target>
</Project>
```

上面代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/63498da1cd6ae26ed3983bc9fc684fbfbed23e6b/DealayhuneaWheluyearfair) 在上面代码所在文件夹执行下面命令，可以看到输出

```
dotnet build
```

输出内容是

```
用于 .NET Core 的 Microsoft (R) 生成引擎版本 16.6.0+5ff7b0c9e
版权所有(C) Microsoft Corporation。保留所有权利。

  正在确定要还原的项目…
  所有项目均是最新的，无法还原。
  ChedarlelaNaykerkeargaini -> C:\lindexi\DealayhuneaWheluyearfair\bin\Debug\netcoreapp3.1\ChedarlelaNaykerkeargaini.dll
C:\lindexi\DealayhuneaWheluyearfair\ChedarlelaNaykerkeargaini.csproj(13,9): warning : Name='NewLife.RocketMQ' Version='1.3.2020.802' PrivateAssets='';Name='Microsoft.Extensions.Logging.Abstractions' Version='3.1.6' PrivateAssets='all'

已成功生成。

C:\lindexi\DealayhuneaWheluyearfair\ChedarlelaNaykerkeargaini.csproj(13,9): warning : Name='NewLife.RocketMQ' Version='1.3.2020.802' PrivateAssets='';Name='Microsoft.Extensions.Logging.Abstractions' Version='3.1.6' PrivateAssets='all'
```



如果将代码的 PackageReferenceVersion 放在 Target 外，如下面代码

```xml
    <ItemGroup>
        <PackageReferenceVersion Include="Name='%(PackageReference.Identity)' Version='%(PackageReference.Version)' PrivateAssets='%(PackageReference.PrivateAssets)'"></PackageReferenceVersion>
    </ItemGroup>
    <Target Name="LabaijalacarDaqarheelu" BeforeTargets="Build">
        <Warning Text="@(PackageReferenceVersion)"/>
    </Target>
```

使用命令行输入下面代码构建

```
dotnet build
```

那么此时的输出如下

```
C:\lindexi\QecoladeBafayearcur\QecoladeBafayearcur.csproj(13,9): warning : Name='%(PackageReference.Identity)' Version='%(PackageReference.Version)' PrivateAssets='%(PackageReference.PrivateAssets)'
```

可以看到没有内容，上面代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/63498da1cd6ae26ed3983bc9fc684fbfbed23e6b/QecoladeBafayearcur) 请小伙伴自己使用命令行试试

更多编译相关请看[手把手教你写 Roslyn 修改编译](https://blog.lindexi.com/post/roslyn.html )

其实在 msbuild 里，如果小伙伴读过吕水大大的 [帮助官方 NuGet 解掉 Bug，制作绝对不会传递依赖的 NuGet 包 - walterlv](https://blog.walterlv.com/post/prevent-nuget-package-been-depended.html ) 那么贺喜，请试试下面的代码

```xml
<Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <TargetFramework>netcoreapp3.1</TargetFramework>
    </PropertyGroup>

    <ItemGroup>
        <PackageReference Include="NewLife.RocketMQ" Version="1.3.2020.802" />
        <PackageReference Include="Microsoft.Extensions.Logging.Abstractions" Version="3.1.6" PrivateAssets="all"></PackageReference>
    </ItemGroup>

    <Target Name="LaldalllayjeaCawerequrwai" BeforeTargets="CollectPackageReferences">
        <ItemGroup>
            <PackageReference Update="F123123" PrivateAssets="123" />
        </ItemGroup>
    </Target>

    <Target Name="LabaijalacarDaqarheelu" BeforeTargets="Build">
        <ItemGroup>
            <PackageReferenceVersion Include="Name='%(PackageReference.Identity)' Version='%(PackageReference.Version)' PrivateAssets='%(PackageReference.PrivateAssets)'"></PackageReferenceVersion>
        </ItemGroup>
        <Warning Text="@(PackageReferenceVersion)"/>
    </Target>

</Project>
```

对比上面的代码，其实添加了 LaldalllayjeaCawerequrwai 的代码，在这里面更新了一个随意的 NuGet 包，将 PrivateAssets 设置为 123 的值

```xml
    <Target Name="LaldalllayjeaCawerequrwai" BeforeTargets="CollectPackageReferences">
        <ItemGroup>
            <PackageReference Update="F123123" PrivateAssets="123" />
        </ItemGroup>
    </Target>
```

此时有什么锅？试试使用命令行构建

```
C:\lindexi\DealayhuneaWheluyearfair\ChedarlelaNaykerkeargaini.csproj(22,9): warning : Name='NewLife.RocketMQ' Version='1.3.2020.802' PrivateAssets='123';Name='Microsoft.Extensions.Logging.Abstractions' Version='3.1.6' PrivateAssets='123'
```

可以看到更新了一个 NuGet 库，但是全部的 NuGet 引用的 PrivateAssets 都被更改了

除了 PrivateAssets 还可以修改 Version 版本的值，也是全局

```xml
    <Target Name="LaldalllayjeaCawerequrwai" BeforeTargets="CollectPackageReferences">
        <ItemGroup>
            <PackageReference Update="Microsoft.Extensions.Logging.Abstractions" Version="3.1.6" PrivateAssets="123" />
        </ItemGroup>
    </Target>
```

此时所有的 NuGet 版本将被修改为 3.1.6 版本

此时将会提示 NewLife.RocketMQ 不包含 3.1.6 版本

[Updated a PackageReference Version will update all of the PackageReference · Issue #12777 · dotnet/sdk](https://github.com/dotnet/sdk/issues/12777 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
