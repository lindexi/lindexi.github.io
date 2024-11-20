
# dotnet 9 WPF 项目禁用 IncludePackageReferencesDuringMarkupCompilation 导致源代码包 XAML 构建失败

本文记录在 dotnet 6 时通过禁用 IncludePackageReferencesDuringMarkupCompilation 解决源代码冲突问题时，在 dotnet 9 将因此导致 XAML 构建生成的 g.cs 文件包含的 XAML 只记录相对文件路径，从而导致构建不通过

<!--more-->


<!-- CreateTime:2024/11/20 07:17:26 -->

<!-- 发布 -->
<!-- 博客 -->

在 [WPF 修复 dotnet 6 与源代码包冲突](https://blog.lindexi.com/post/WPF-%E4%BF%AE%E5%A4%8D-dotnet-6-%E4%B8%8E%E6%BA%90%E4%BB%A3%E7%A0%81%E5%8C%85%E5%86%B2%E7%AA%81.html ) 这篇博客里面和大家介绍通过禁用 IncludePackageReferencesDuringMarkupCompilation 解决源代码冲突问题

以下是在 dotnet 6 里的构建失败信息

```
C:\Program Files\dotnet\sdk\6.0.101\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.Sdk.DefaultItems.Shared.targets(190,5): error NETSDK1022: 包含了重复的“Compile”项。.NET SDK 默认包含你项目目录中的“Compile”项。可从项目文件中删除这些项；如果希望将其显式包含在项目文件中，可将“EnableDefaultCompileItems”属性设置为“false”。有关详细信息，请参阅 https://aka.ms/sdkimplicititems。重复项为: 
```

通过禁用 IncludePackageReferencesDuringMarkupCompilation 即可解决，代码如下

```xml
    <IncludePackageReferencesDuringMarkupCompilation>False</IncludePackageReferencesDuringMarkupCompilation>
```

更改之后的 csproj 代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net6.0-windows</TargetFramework>
    <UseWPF>true</UseWPF>
    <IncludePackageReferencesDuringMarkupCompilation>False</IncludePackageReferencesDuringMarkupCompilation>
  </PropertyGroup>

</Project>
```

依然是上述代码不变，升级到 dotnet 9 的 SDK 之后，将会让代码构建不通过。假定源代码包里面包含了 MyControl.xaml 文件，那么构建过程提示错误内容大概如下

```
MyControl.g.cs(62,18): error CS1504: 无法打开源文件“MyControl.xaml”-- 无法找到文件
```

进入到 obj 文件夹下，去看 MyControl.g.cs 文件，可以看到开始的 `#pragma checksum` 记录的就是错误的相对路径

```csharp
#pragma checksum "MyControl.xaml" "{ff1916ec-aa5e-4d10-97f7-6f4963933460}" "196544C162DD55A903399A4024C5999A1B6017EB"
```

预期的情况下，此时的记录应该是一个正确的相对路径或绝对路径，如下面代码所示才是正确的

```csharp
#pragma checksum "..\..\..\..\..\..\..\..\..\..\..\Users\lindexi\.nuget\packages\lindexi.package.wpf.source\1.0.0\src\View\MyControl.xaml" "{ff1916ec-aa5e-4d10-97f7-6f4963933460}" "196544C162DD55A903399A4024C5999A1B6017EB"
```

解决方法只有是开启 IncludePackageReferencesDuringMarkupCompilation 功能，即只需注释掉 `<IncludePackageReferencesDuringMarkupCompilation>False</IncludePackageReferencesDuringMarkupCompilation>` 即可

当然了，注释掉 IncludePackageReferencesDuringMarkupCompilation 的禁用，则又会出现源代码兼容问题。不过现在是 dotnet 9 了，是时候更新源代码包啦

```xml
<IncludePackageReferencesDuringMarkupCompilation Condition="$([MSBuild]::VersionGreaterThanOrEquals($(NETCoreSdkVersion), 9.0))">True</IncludePackageReferencesDuringMarkupCompilation>
```

以上代码为常写的判断代码，但在使用以上代码之前，还请确保源代码包已经更新

禁用 IncludePackageReferencesDuringMarkupCompilation 导致构建时 `#pragma checksum` 记录错误的路径，这个问题不单只是在引用源代码包的时候能够出现，也能够在跨项目引用 xaml 时复现

禁用 IncludePackageReferencesDuringMarkupCompilation 功能，将会导致在跨项目引用 xaml 文件，将 xaml 文件作为链接方式添加时，构建过程中出现 error CS1504 错误

禁用 IncludePackageReferencesDuringMarkupCompilation 功能之后，从 xaml 生成的 g.cs 文件里的 `#pragma checksum` 将记录错误的相对文件路径，进而导致构建失败。最简单的复现方式如下：

1. 创建两个 wpf 项目，其中一个为 WPF 库项目，一个为 WPF 应用项目
2. 在 WPF 库项目里面添加名为 MyUserControl 的用户控件
3. 在 WPF 应用项目里面通过以下代码引用 MyUserControl.xaml 和 MyUserControl.xaml.cs 文件

```xml
  <ItemGroup>
    <Compile Include="..\HalllidanairjelDawearlairnal\MyUserControl.xaml.cs" Link="MyUserControl.xaml.cs" />

    <Page Include="..\HalllidanairjelDawearlairnal\MyUserControl.xaml" Link="MyUserControl.xaml">
      <Generator>MSBuild:Compile</Generator>
    </Page>
  </ItemGroup>
```

如果此时同样在 WPF 应用项目里设置禁用 IncludePackageReferencesDuringMarkupCompilation 功能，则构建时将会提示 `obj\Debug\net9.0-windows\MyUserControl.g.cs(59,21,59,41): error CS1504: 无法打开源文件“MyUserControl.xaml”-- 无法找到文件` 错误。以下是设置禁用 IncludePackageReferencesDuringMarkupCompilation 功能的 WPF 应用项目的 csproj 项目文件的代码

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net9.0-windows</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <UseWPF>true</UseWPF>

    <IncludePackageReferencesDuringMarkupCompilation>False</IncludePackageReferencesDuringMarkupCompilation>

  </PropertyGroup>

  <ItemGroup>
    <Compile Include="..\HalllidanairjelDawearlairnal\MyUserControl.xaml.cs" Link="MyUserControl.xaml.cs" />

    <Page Include="..\HalllidanairjelDawearlairnal\MyUserControl.xaml" Link="MyUserControl.xaml">
      <Generator>MSBuild:Compile</Generator>
    </Page>
  </ItemGroup>

</Project>
```

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/ee493914d4ac77e06bf6ac789ad84a0e8452be76/WPFDemo/TestIncludePackageReferencesDuringMarkupCompilation) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/ee493914d4ac77e06bf6ac789ad84a0e8452be76/WPFDemo/TestIncludePackageReferencesDuringMarkupCompilation) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin ee493914d4ac77e06bf6ac789ad84a0e8452be76
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin ee493914d4ac77e06bf6ac789ad84a0e8452be76
```

获取代码之后，进入 WPFDemo/TestIncludePackageReferencesDuringMarkupCompilation 文件夹，即可获取到源代码

大家可以尝试拉取项目代码跑跑看。 此问题已经和 WPF 官方报告，详细请看 <https://github.com/dotnet/wpf/issues/10093>

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。