# 使用 SatelliteResourceLanguages 减少 WPF 发布的多语言文件夹数量

不知是否有伙伴也遇到这样的问题，WPF 发布的时候，生成的多语言文件夹数量太多了。这些多语言文件夹里面，绝大部分内容都是些用来抛异常用的字符串，或者是一些不常用的字符串。虽然单个 DLL 文件不大，但是数量太多了，如此也会多占用一些磁盘空间

<!--more-->
<!-- CreateTime:2025/06/17 07:06:20 -->

<!-- 发布 -->
<!-- 博客 -->

在 dotnet 发布机制里面，可以通过 SatelliteResourceLanguages 来指定需要发布的多语言的范围。这样就可以减少发布的多语言文件夹数量

其配置示例如下

```xml
    <SatelliteResourceLanguages>zh-Hant;zh-Hans;en-US</SatelliteResourceLanguages>
```

将其放在 csproj 项目文件的 PropertyGroup 里面即可。如以下我的 csproj 项目文件代码

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net9.0-windows</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <UseWPF>true</UseWPF>
  </PropertyGroup>

  <PropertyGroup>
    <SatelliteResourceLanguages>zh-Hant;zh-Hans;en-US</SatelliteResourceLanguages>
  </PropertyGroup>
</Project>
```

如果不知道具体怎么配置，可以到本文末尾拉取我的全部项目代码，查看完全的项目组织内容

完成以上配置之后，重新发布项目，就可以看到生成的多语言文件夹数量减少了。只剩下在 SatelliteResourceLanguages 里面指定的多语言文件夹了。其他的多语言文件夹都没有生成了

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/8c46e8f81be31493016a3d35fccde479c8bf2656/WPFDemo/WhaicarairhalnerKajuluho) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/8c46e8f81be31493016a3d35fccde479c8bf2656/WPFDemo/WhaicarairhalnerKajuluho) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 8c46e8f81be31493016a3d35fccde479c8bf2656
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 8c46e8f81be31493016a3d35fccde479c8bf2656
```

获取代码之后，进入 WPFDemo/WhaicarairhalnerKajuluho 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )