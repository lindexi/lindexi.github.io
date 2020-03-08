# SourceYard 制作源代码包

本文带大家走进SourceYard开发之旅


在项目开发中，将一个大的项目拆为多个小项目解耦，减少模块之间的耦合。因为如果将代码放在一起，即使有团队的约束，但只要能写出的代码就会有小伙伴写出，很快就发现各个模块耦合的代码很多。但是对一个项目的拆分会让拆分出来的每一个项目都编译出一个 dll 增加运行文件的启动时间。

在开发中，常常会用到很多工具类，这些小轮子很多的功能基本就只有一个类，如何对这些小轮子进行管理？通过复制代码还是通过 Nuget 管理？

<!--more-->
<!-- CreateTime:2018/12/25 9:43:07 -->


如果使用复制代码的方式，很难知道从哪里复制代码，如果在很多项目都复制了代码，发现原来的代码存在一些虫子，很难修改所有复制代码的项目。通过传统 Nuget 的方式可以方便管理工具的更新，和引入工具，同时会将每个小轮子打包成一个 dll 这样会引入很多 dll 让软件启动的速度和运行的速度降低。关于 dll 数量和启动时间的测试请看 [C# 程序集数量对软件启动性能的影响](https://lindexi.gitee.io/post/C-%E7%A8%8B%E5%BA%8F%E9%9B%86%E6%95%B0%E9%87%8F%E5%AF%B9%E8%BD%AF%E4%BB%B6%E5%90%AF%E5%8A%A8%E6%80%A7%E8%83%BD%E7%9A%84%E5%BD%B1%E5%93%8D.html ) 介绍了程序集数量对软件启动性能的影响，运行的性能是在引用某个 dll 方法的时候需要加载这个 dll 降低速度。

有小伙伴问，为何不将所有的工具放在一个大的项目，这样每次只需要更新大项目的 Nuget 就可以，这样就可以解决引入dll的数量和管理小工具。

虽然将很多的工具放在一个程序集做一个 Nuget 的方式看起来不错，但是只是在很小的项目同时不能维护太久，在我现在的团队有一个库，这个库就是用来放小工具，但是在经过了一段时间，发现基本上所有小伙伴在不知道要将类放在哪个地方的时候，就会放在这个程序集里。同时因为所有工具都在一个程序集里，所有小工具都在相互引用。在我想要修复某个小工具的功能的时候，发现在这个程序集内这个工具已经有 99 引用，其中还有不少地方依赖 bug 编程，这时维护这样一个程序集的成功非常高，同时无法组织小伙伴不断将含义不明确的类放在这个程序集（这里不是在讨论代码审查问题，在我现在的团队是有明确的代码审查，然而没有人能说清这个程序集的功能），所以这就是为什么不建议所有小工具放在一个程序集的原因。另外如果都将代码放在一个程序集，用于分享也是比较难，有小伙伴向我要一些工具，假设我都放在一个程序集里，那么我只能通过拷贝代码的方式给他，因为我不确定工具程序集里面是否有不能对外发布的内容，但如果是 SourceYard 的方法，作为源代码包可以将小伙伴需要的工具发布到 Nuget.org 请他去安装。

当然将工具放在一个工具程序集也不是没有优点，因为所有的工具都在一个程序集里面，小伙伴可以方便找到自己需要的类，而不是通过 Nuget 的方式去寻找安装。同时如果有一个项目多个程序集需要相同的工具，可以同时依赖工具程序集，减少创建出来的 dll 文件里重复代码。

解决上面的两个问题的方法是通过 SourceYard 的方法，使用 SourceYard 不但可以解决项目解耦创建了很多个项目让很多个项目编译出来的 dll 太多让软件启动性能降低，同时解决小工具类太多的问题，还可以解决代码兼容的问题。

## 控制台项目

创建一个 dotnet core 项目进行开发，这里创建一个 dotnet core 项目主要是因为创建出来的项目清真

假设有一个需求是做一个工具，这个工具的功能是用户输入数字，转换为人民币金额大写，听起来这个功能很简单，当然在本文就不会详细告诉大家这个工具的代码。

<!-- ![](image/SourceYard 制作源代码包/SourceYard 制作源代码包0.png) -->

![](http://image.acmx.xyz/lindexi%2F201812911718537)

在我之前的博客[C# 金额转中文大写](https://lindexi.gitee.io/post/C-%E9%87%91%E9%A2%9D%E8%BD%AC%E4%B8%AD%E6%96%87%E5%A4%A7%E5%86%99.html )已经有了代码，可以从[码云复制](https://gitee.com/lindexi/codes/w6bxlue9o14rv5nscjyhf20 ) 复制的代码因为没有命名空间，需要手动添加，于是现在就创建了一个项目，这个项目包含一个类。

<!-- ![](image/SourceYard 制作源代码包/SourceYard 制作源代码包1.png) -->

![](http://image.acmx.xyz/lindexi%2F2018129111242105)

使用 SourceYard 很简单，只需要在 TheLib 项目管理 Nuget 安装 SourceYard 就可以

<!-- ![](image/SourceYard 制作源代码包/SourceYard 制作源代码包2.png) -->

![](http://image.acmx.xyz/lindexi%2F2018129111349653)

现在 SourceYard 还没正式发布，里面还存在一些坑，但是对于这么简单的项目已经可以使用。

如果对 SourceYard 感兴趣，请在 [github](https://github.com/dotnet-campus/SourceYard ) 关注

右击 TheLib 的属性，在打包的页面勾选在版本中生成 Nuget 包，勾选之后重新编译就可以制作出 Nuget 包。但是请不要急，在打包页面还有很多东西需要填写，在[广州 .NET 微软技术俱乐部12月份活动](http://www.10tiao.com/html/391/201811/2654072986/1.html) 的演示中，我使用了[这个黑科技](https://lindexi.gitee.io/post/Roslyn-%E5%B0%86%E8%BF%99%E4%B8%AA%E6%96%87%E4%BB%B6%E6%94%BE%E5%9C%A8%E4%BD%A0%E7%9A%84%E9%A1%B9%E7%9B%AE%E6%96%87%E4%BB%B6%E5%A4%B9-%E6%97%A0%E8%AE%BA%E5%93%AA%E4%B8%AA%E6%8E%A7%E5%88%B6%E5%8F%B0%E9%A1%B9%E7%9B%AE%E9%83%BD%E4%BC%9A%E8%BE%93%E5%87%BA%E6%9E%97%E5%BE%B7%E7%86%99%E6%98%AF%E9%80%97%E6%AF%94.html )瞬间完成了所有属性，小伙伴如果还没学会这个黑科技就需要手动填写内容了，其实只有作者、公司、说明是必要的，其他的可以不写。

<!-- ![](image/SourceYard 制作源代码包/SourceYard 制作源代码包3.png) -->

![](http://image.acmx.xyz/lindexi%2F2018129111533228)

如果需要将 TheLib 修改为 dotnet framework 的项目，只需要右击编辑 csproj 文件就可以，在[理解 C# 项目 csproj 文件格式的本质和编译流程](https://walterlv.gitee.io/post/understand-the-csproj.html )和[广州 .NET 微软技术俱乐部12月份活动](http://www.10tiao.com/html/391/201811/2654072986/1.html) 吕毅都讲了项目的格式，如果只需要修改 dotnet framework 不需要了解那么多，只需要在 TargetFramework 里面修改为 net45 就完成

```
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net45</TargetFramework>
    <GeneratePackageOnBuild>true</GeneratePackageOnBuild>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="dotnetCampus.SourceYard" Version="0.1.7213-alpha">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
    </PackageReference>
  </ItemGroup>

</Project>
```

现在右击项目重新编译就可以打出 Nuget 包，在使用新的项目格式，默认的 dotnet core 项目就是这么简单，具体请看[VisualStudio 使用新项目格式快速打出 Nuget 包](https://lindexi.gitee.io/post/VisualStudio-%E4%BD%BF%E7%94%A8%E6%96%B0%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F%E5%BF%AB%E9%80%9F%E6%89%93%E5%87%BA-Nuget-%E5%8C%85.html )详细写了黑科技

在输出的文件夹可以找到打包的 Nuget 文件

<!-- ![](image/SourceYard 制作源代码包/SourceYard 制作源代码包4.png) -->

![](http://image.acmx.xyz/lindexi%2F2018129112917582)

文件可以通过 [Nuget 管理器](https://www.microsoft.com/store/productId/9WZDNCRDMDM3) 打开，这个文件可以在应用商店找到

<!-- ![](image/SourceYard 制作源代码包/SourceYard 制作源代码包5.png) -->

![](http://image.acmx.xyz/lindexi%2F2018129113026481)

这里两个不同的文件，其他是传统的 Nuget 包，也就是 TheLib.1.0.2.nupkg 里面包含 dll 请打开文件很快就可以看到

<!-- ![](image/SourceYard 制作源代码包/SourceYard 制作源代码包6.png) -->

![](http://image.acmx.xyz/lindexi%2F2018129113243747)

源代码的文件的格式也请小伙伴打开看一下，里面没有 dll 里面是代码，在安装这个文件就会引用代码，代码会编译在引用的项目。多个不同的源代码包会编译为一个程序集。

<!-- ![](image/SourceYard 制作源代码包/SourceYard 制作源代码包7.png) -->

![](http://image.acmx.xyz/lindexi%2F201812911383738)

虽然有 Nuget 文件但是还不知道这个文件能不能使用，创建两个不同的项目用来用这两个文件，因为刚才已经修改项目为 dotnet framework 的，就需要创建一个 dotnet framework 的项目

<!-- ![](image/SourceYard 制作源代码包/SourceYard 制作源代码包8.png) -->

![](http://image.acmx.xyz/lindexi%2F2018129115048210)

右击项目管理 Nuget 引用本地的 Nuget 文件的文件夹，如我这里的 Nuget 文件是在 `D:\lindexi\SourceYard\bin\Debug` 文件夹下，我就需要添加这样的路径，请看图片

<!-- ![](image/SourceYard 制作源代码包/SourceYard 制作源代码包9.png) -->

![](http://image.acmx.xyz/lindexi%2F2018129115544606)

这里的程序包源的名称是可以随意给的，程序员最难的就是命名，好在有[Whitman](https://www.microsoft.com/store/productId/9P8LNZRNJX85 ) 这个工具可以按下 ctrl+, 输入第一个字符为小写的变量，按下 ctrl+shift+, 输入第一个字符为大写的变量。现在这个工具已经从 dotnet framework 升级到 dotnet core 请看 [将基于 .NET Framework 的 WPF 项目迁移到基于 .NET Core 3 - walterlv](https://walterlv.gitee.io/post/migrate-wpf-project-from-dotnet-framework-to-dotnet-core.html ) 关于 WPF 怎么可以在 dotnet core 运行，微软已经将 WPF 的 dotnet core 开源，可以在 [github](https://github.com/dotnet/wpf ) 找到

现在点击本地的源，如刚才命名为 TacaluTawnenai 的源就可以找到刚才的两个文件，如何选择本地的源？请看下图，点击程序包源的下拉就可以找到

<!-- ![](image/SourceYard 制作源代码包/SourceYard 制作源代码包10.png) -->

![](http://image.acmx.xyz/lindexi%2F201812912055633)

刚才创建的 dotnet framework 程序还是比较不清真的，先进行卸载，然后编辑 csproj 文件，可以看到这里的文件内容非常多，这是很不清真的。从刚才的 TheLib 文件里面拷贝 csproj 文件到 AppUsingDll 项目里，记得需要先去掉 SourceYard 的部分

```
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net45</TargetFramework>
    <GeneratePackageOnBuild>true</GeneratePackageOnBuild>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="dotnetCampus.SourceYard" Version="0.1.7213-alpha">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
    </PackageReference>
  </ItemGroup>

</Project>
```

将上面不需要的部分，也就是引用 SourceYard 包的部分和 GeneratePackageOnBuild 去掉，现在剩下的代码很少

```
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net45</TargetFramework>
  </PropertyGroup>
</Project>
```

控制台项目需要做很小的修改，通过右击项目属性，在界面选择控制台

<!-- ![](image/SourceYard 制作源代码包/SourceYard 制作源代码包12.png) -->

![](http://image.acmx.xyz/lindexi%2F201812913059674)

如果此时进行编译会看到编译不通过，因为还需要删除 AssemblyInfo.cs 文件的很多代码，其实可以直接删除这个文件

在这个项目需要安装 TheLib 库，安装的方式和安装其他的 Nuget 没有不同，通过本地的文件夹安装 Nuget 包和通过 Nuget 服务器安装没有不同，如果需要自己搭建 Nuget 服务器也是十分简单，请看[通过ProGet搭建一个内部的Nuget服务器 - 张善友 - 博客园](http://www.cnblogs.com/shanyou/p/5910250.html )我就帮小伙伴在10分钟内搭建 Nuget 服务器

<!-- ![](image/SourceYard 制作源代码包/SourceYard 制作源代码包11.png) -->

![](http://image.acmx.xyz/lindexi%2F201812912055633)

安装之后添加一点代码测试一下能否使用

```csharp
            var money = new Money(12312);
            Console.WriteLine(money.ToCapital());
            Console.ReadLine();
```

这时运行一下，可以看到成功运行了。

再创建一个项目，这个项目尝试使用 dotnet core 的项目

<!-- ![](image/SourceYard 制作源代码包/SourceYard 制作源代码包13.png) -->

![](http://image.acmx.xyz/lindexi%2F2018129143342731)

右击项目管理 Nuget 安装源代码包，然后在主函数添加相同的测试代码

<!-- ![](image/SourceYard 制作源代码包/SourceYard 制作源代码包14.png) -->

![](http://image.acmx.xyz/lindexi%2F2018129143448185)

```csharp
        static void Main(string[] args)
        {
            var money = new Money(12312);
            Console.WriteLine(money.ToCapital());
            Console.ReadLine();
        }
```

现在尝试运行，可以看到和刚才的 dotnet framework 控制台输出相同

但是有一点不相同的是，打开两个项目的输出文件夹，可以看到 dotnet framework 项目引用的是 dll 的方式，输出的文件夹有一个dll和一个exe 在 dotnet core 项目的输出文件夹只有一个 dll 因为默认的 dotnet core 输出的是 dll 源代码就放在相同的 dll 里

<!-- ![](image/SourceYard 制作源代码包/SourceYard 制作源代码包15.png) -->

![](http://image.acmx.xyz/lindexi%2F201812914371424)

<!-- ![](image/SourceYard 制作源代码包/SourceYard 制作源代码包16.png) -->

![](http://image.acmx.xyz/lindexi%2F2018129143741224)

这时就可以看到 SourceYard 的好处，通过 SourceYard 可以将源代码作为 Nuget 包，这样不但减少 dll 的数量，同时做到源代码的兼容。在之前，无论是PCL还是多项目方式的 Nuget 包管理多个不同平台的兼容难度比较大，但是通过 SourceYard 只要源代码可以兼容就可以安装。在本文的控制台的使用的库是 dotnet framework 4.5 但是控制台项目使用的是 dotnet core 2.1 的，这样都可以使用。

## WPF 程序

如果小伙伴觉得控制台还是太简单了，可以尝试使用桌面WPF程序，此时 WinForms 程序也是适合的。

创建一个简单的 WPF 库，注意创建之后需要修改代码，修改项目格式为 VisualStudio 2017 格式

在开始编辑之前，先创建一个用户控件，这里叫 InterestingControl 一个有趣的控件，同时删除 Properties 文件夹的所有文件

<!-- ![](image/SourceYard 制作源代码包/SourceYard 制作源代码包17.png) -->

![](http://image.acmx.xyz/lindexi%2F2018129144418506)

相对来说，做 WPF 的库使用新项目格式要求对新的格式比较熟悉，所以请直接卸载项目，编辑一下项目文件，填入下面的代码

```
<Project Sdk="Microsoft.NET.Sdk" ToolsVersion="15.0">
  <PropertyGroup>
    <LanguageTargets>$(MSBuildToolsPath)\Microsoft.CSharp.targets</LanguageTargets>
    <TargetFrameworks>net45;</TargetFrameworks>
    <OutputType>Library</OutputType>
    <RootNamespace>WpfUI</RootNamespace>
    <AssemblyName>WpfUI</AssemblyName>
    <GeneratePackageOnBuild>true</GeneratePackageOnBuild>
  </PropertyGroup>
  <PropertyGroup>
    <DefineConstants>$(DefineConstants);WPF</DefineConstants>
  </PropertyGroup>
 
  <ItemGroup>
    <Compile Update="**\*.xaml.cs">
      <DependentUpon>%(Filename)</DependentUpon>
    </Compile>
    <Page Include="**\*.xaml">
      <SubType>Designer</SubType>
      <Generator>MSBuild:Compile</Generator>
    </Page>

    <Resource Include="**\*.png" />
    <Resource Include="**\*.jpg" />
    <Resource Include="**\*.cur" />
    <Resource Include="**\*.ps" />
    <None Include="**\*.fx" />
    <None Include="**\*.md" />
    <None Include="**\*.ruleset" />
  </ItemGroup>
 
  <ItemGroup>
    <Compile Remove="obj\**" />
    <EmbeddedResource Remove="obj\**" />
    <None Remove="obj\**" />
    <Page Remove="obj\**" />
    <Resource Remove="obj\**" />
  </ItemGroup>
 
  <ItemGroup>
    <PackageReference Include="dotnetCampus.SourceYard" Version="0.1.7213-alpha">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
    </PackageReference>
  </ItemGroup>
 
  <ItemGroup>
    <Reference Include="System" />
    <Reference Include="System.Data" />
    <Reference Include="System.Xml" />
    <Reference Include="Microsoft.CSharp" />
    <Reference Include="System.Core" />
    <Reference Include="System.Xml.Linq" />
    <Reference Include="System.Data.DataSetExtensions" />
    <Reference Include="System.Net.Http" />
    <Reference Include="System.Xaml" />
    <Reference Include="WindowsBase" />
    <Reference Include="PresentationCore" />
    <Reference Include="PresentationFramework" />
  </ItemGroup>
 
  <ItemGroup>
    <Compile Update="InterestingControl.xaml.cs">
      <SubType>Code</SubType>
    </Compile>
  </ItemGroup>
 
  <ItemGroup>
    <Folder Include="Properties\" />
  </ItemGroup>
 
  <ItemGroup>
    <None Update="InterestingControl.xaml">
      <Generator>MSBuild:Compile</Generator>
    </None>
  </ItemGroup>
</Project>
```

请根据实际的项目更改 AssemblyName 的值

制作源代码包的方式和控制台的相同，只需要在 Nuget 安装 SourceYard 就可以，同样打开属性，和控制台一样

<!-- ![](image/SourceYard 制作源代码包/SourceYard 制作源代码包18.png) -->

![](http://image.acmx.xyz/lindexi%2F201812914469889)

现在右击重新编译，就可以在输出文件夹找到输出的两个 Nuget 包

再创建一个 WPF 程序，通过设置本地的 Nuget 包的文件夹，安装源代码包，然后在界面使用刚才的用户控件，运行就可以发现成功使用了用户控件

打开 WPF 程序的输出文件夹，可以发现这个文件夹里面只有一个 exe 源代码已经放在 exe 里

## 调试

在将项目制作 Nuget 包的时候，就有小伙伴吐槽在开发的时候，如果使用 Nuget 安装，很难进行调试，很难在 dll 里面添加断点，同时在调试的时候修改代码

但是在使用 SourceYard 调试的时候，安装 Nuget 的库和调试本地的引用的代码是完全一样的，就使用上面的控制台调试

例如需要无论用户输出的是什么返回的都是 林德熙是逗比

细心的小伙伴会发现安装 SourceYard 之后，在项目会出现一个文件夹是 SourceProject 里面就有一个文件 TheLib.Source.SourceProject.props 打开这个文件可以看到下面的内容

```
<?xml version="1.0" encoding="utf-8"?>
<Project>

  <!--这个文件由代码创建，不建议删除这个文件-->

  <!--取消下面的注释，并且修改路径为自己对应的 TheLib.Source 文件夹的目录，可以通过修改源代码测试-->

  <!--<PropertyGroup Condition=" !$(DefineConstants.Contains('TheLibSOURCE_REFERENCE')) ">
    <DefineConstants>$(DefineConstants);TheLibSOURCE_REFERENCE</DefineConstants>
  </PropertyGroup>-->
  
  <!--修改路径为自己的源代码文件夹-->
  <!--  <PropertyGroup>
    <TheLibSourceFolder>c:\lindexi\source\ 这是一个示例文件夹，请将这个替换为自己的源代码包文件夹</TheLibSourceFolder>
  </PropertyGroup>  -->


</Project>
```

这个文件里面有一些代码暂时无法使用，需要先做设置，首先设置 TheLibSourceFolder 这里表示源代码包的原始项目的文件夹，如 TheLib 的文件夹是 `C:\lindexi\SourceYard\TheLib` 请小心设置路径

<!-- ![](image/SourceYard 制作源代码包/SourceYard 制作源代码包19.png) -->

![](http://image.acmx.xyz/lindexi%2F2018129145744939)

然后取消除了中文的注释，请看代码

```
<?xml version="1.0" encoding="utf-8"?>
<Project>

  <!--这个文件由代码创建，不建议删除这个文件-->

  <!--取消下面的注释，并且修改路径为自己对应的 TheLib.Source 文件夹的目录，可以通过修改源代码测试-->

  <PropertyGroup Condition=" !$(DefineConstants.Contains('TheLibSOURCE_REFERENCE')) ">
    <DefineConstants>$(DefineConstants);TheLibSOURCE_REFERENCE</DefineConstants>
  </PropertyGroup>

  <!--修改路径为自己的源代码文件夹-->
  <PropertyGroup>
    <TheLibSourceFolder>C:\lindexi\SourceYard\TheLib</TheLibSourceFolder>
  </PropertyGroup>


</Project>
```

此时通过 ctrl+点击的方式可以进入 Money 文件，这时可以进行调试

<!-- ![](image/SourceYard 制作源代码包/SourceYard 制作源代码包20.png) -->

![](http://image.acmx.xyz/lindexi%2F2018129151516104)

此时在这个函数下面返回林德熙是逗比

<!-- ![](image/SourceYard 制作源代码包/SourceYard 制作源代码包21.png) -->

![](http://image.acmx.xyz/lindexi%2F2018129151516104)

按 F5 运行，可以看到输出的是 林德熙是逗比 也就是源代码已经修改

打开 TheLib 的 Money.cs 文件可以发现里面的文件也同时被修改，也就是这个文件被两个项目引用，通过这个方法就和将项目引用的方式那样调试解决 dll 难以断点和修改代码

如果不设置 TheLibSourceFolder 路径，也是可以调试文件，同样也可以在调试的时候修改代码，但是这时的代码是无法上传的，也就是只能在本地的缓存使用，在清空缓存之后，对代码的修改将会找不到

因为 SourceYard 还在开发过程，代码开放在 [github](https://github.com/dotnet-campus/SourceYard ) 欢迎小伙伴贡献
 
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
