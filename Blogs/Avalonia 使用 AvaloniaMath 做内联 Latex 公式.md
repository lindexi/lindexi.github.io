---
title: Avalonia 使用 AvaloniaMath 做内联 Latex 公式
description: 本文将告诉大家如何使用 AvaloniaMath 库在 TextBlock 里面，通过 InlineUIContainer 实现内联 Latex 公式效果
tags: Avalonia
category: 
---

<!-- 发布 -->
<!-- 博客 -->

按照 .NET 惯例，先通过 NuGet 安装库，安装之后的 csproj 项目文件代码大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ApplicationManifest>app.manifest</ApplicationManifest>
    <AvaloniaUseCompiledBindingsByDefault>true</AvaloniaUseCompiledBindingsByDefault>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Avalonia" Version="11.3.12" />
    <PackageReference Include="Avalonia.Desktop" Version="11.3.12" />
    <PackageReference Include="Avalonia.Themes.Fluent" Version="11.3.12" />
    <PackageReference Include="Avalonia.Fonts.Inter" Version="11.3.12" />
    <!--Condition below is needed to remove Avalonia.Diagnostics package from build output in Release configuration.-->
    <PackageReference Include="Avalonia.Diagnostics" Version="11.3.12">
      <IncludeAssets Condition="'$(Configuration)' != 'Debug'">None</IncludeAssets>
      <PrivateAssets Condition="'$(Configuration)' != 'Debug'">All</PrivateAssets>
    </PackageReference>
    <PackageReference Include="AvaloniaMath" Version="2.1.0" />
  </ItemGroup>
</Project>
```

本文用到的 AvaloniaMath 库是一个开源项目，最初是做 WPF 的 Latex 公式支持，后面为了支持 Avalonia 框架就改名为 xaml-math 项目，项目地址是： <https://github.com/ForNeVeR/xaml-math>

完成基础库安装之后，即可直接在 XAML 里面使用，其演示代码如下

```xml
<Window ... xmlns:controls="clr-namespace:AvaloniaMath.Controls;assembly=AvaloniaMath">
        <TextBlock>
            <Run>123</Run>
            <InlineUIContainer BaselineAlignment="Bottom">
                <controls:FormulaBlock Formula="\frac{1}{2}"/>
            </InlineUIContainer>
        </TextBlock>
</Window>
```

运行起来的效果如下

<!-- ![](image/Avalonia 使用 AvaloniaMath 做内联 Latex 公式/Avalonia 使用 AvaloniaMath 做内联 Latex 公式0.png) -->

![](https://img2024.cnblogs.com/blog/1080237/202604/1080237-20260413085551818-932179181.png)

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/55a2a3f647d2cce4e1c5813471f18085dbae42a8/AvaloniaIDemo/QercirawhalroNaicawcalqel) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/55a2a3f647d2cce4e1c5813471f18085dbae42a8/AvaloniaIDemo/QercirawhalroNaicawcalqel) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 55a2a3f647d2cce4e1c5813471f18085dbae42a8
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 55a2a3f647d2cce4e1c5813471f18085dbae42a8
```

获取代码之后，进入 AvaloniaIDemo/QercirawhalroNaicawcalqel 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
