# VisualStudio 通过配置 DefaultXamlRuntime 属性 让控制台项目里的 XAML 应用上智能提示

本文记录一个 VisualStudio 黑科技，通过配置 DefaultXamlRuntime 属性，即可让非 WPF 或 WinUI 或 MAUI 等系列类型的项目也可以拥有 XAML 的智能提示，智能提示方式和 WinUI 智能提示行为相同

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

比如说在一个控制台项目里面，我期望从控制台开始，定制自己的 UI 框架，比如说到现在还没有支持 XAML 的 CPF 框架，我期望让 VisualStudio 能够支持 XAML 的智能提示，提高开发者开发效率

一个简单的实现方式就是本文标题里面提到的，通过配置 DefaultXamlRuntime 项目属性，如以下编写在 csproj 里面的代码

```xml
  <PropertyGroup>
    <DefaultXamlRuntime>WinUI</DefaultXamlRuntime>
  </PropertyGroup>
```

如果是框架制作方，也可以放在 NuGet 里面的 `$(PackageId).props` 文件里面，详细关于 NuGet 等定制构建过程，请参阅 我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

我以一个控制台项目作为例子，和大家演示一下使用效果

先创建一个名为 BellikarjeHakurheekall 的控制台项目，接着编辑 csproj 文件，修改为以下代码

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>

    <DefaultXamlRuntime>WinUI</DefaultXamlRuntime>
  </PropertyGroup>

</Project>
```

以上代码里面的核心就是添加了 `<DefaultXamlRuntime>WinUI</DefaultXamlRuntime>` 这行代码，其他的都是默认的控制台项目的代码

再创建两个**空**文件，分别是 BlankPage1.xaml 和 BlankPage1.xaml.cs 文件。在 csproj 里面将 BlankPage1.xaml 设置为 Page 方式，如以下代码

```xml
  <ItemGroup>
    <Page Include="BlankPage1.xaml">
      <Generator>MSBuild:Compile</Generator>
    </Page>
  </ItemGroup>
```

修改之后整个 csproj 的代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>

    <DefaultXamlRuntime>WinUI</DefaultXamlRuntime>
  </PropertyGroup>

  <ItemGroup>
    <Page Include="BlankPage1.xaml">
      <Generator>MSBuild:Compile</Generator>
    </Page>
  </ItemGroup>
</Project>
```

对于框架的制作方，可以放心的使用 `<Page Include="**\*.xaml">` 的方式引用，这样就不用每添加一个 XAML 都引用一次

接着就可以开始在 BlankPage1.xaml.cs 创建一个空类了，代码如下

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;

namespace BellikarjeHakurheekall;

public sealed partial class BlankPage1 : Page
{
    public BlankPage1()
    {
    }
}
```

当然了，此时的代码还是构建不通过的，毕竟找不到名为 Page 的类型。为了让 BlankPage1 类型能够符合语法，咱将添加名为 Page 的类型，且放入一个用于测试的 Background 属性，代码如下

```csharp
namespace BellikarjeHakurheekall;

public class Page
{
    public object Background { set; get; }
}
```

只有一个 Page 也太空了，咱再添加一个名为 Grid 的类型，如以下代码

```csharp
namespace BellikarjeHakurheekall;

public class Grid
{
    public object Background { set; get; }
}
```

完成此之后，即可编写 BlankPage1.xaml 的代码，代码如下

```xml
<Page
    x:Class="BellikarjeHakurheekall.BlankPage1"
    xmlns="clr-namespace:BellikarjeHakurheekall"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:BellikarjeHakurheekall"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d"
    Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">

    <Grid>

    </Grid>
</Page>
```

可以看到在编写 BlankPage1.xaml 的过程中，是存在智能提示的。以上代码的一个细节是按照 XML 的标准规范，设置了 `xmlns="clr-namespace:BellikarjeHakurheekall"` 作为默认的命名空间，于是所有放在此命名空间下的类型都可以不用手动再标注命名空间了

以上代码的 `mc:Ignorable="d"` 和 `xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"` 和 `xmlns:d="http://schemas.microsoft.com/expression/blend/2008"` 这几个属于通用的命名空间，推荐复用，可以用来作为设计时使用

完成以上代码之后，可以试试编译项目。项目编译的时候是不会对 XAML 做任何的事情的，只会当成一个不认识的文件，被忽略掉。这是因为咱没有加入任何的 XAML 编译器和处理逻辑，仅仅只是开启 VisualStudio 对 XAML 的智能提示

更进一步，有时候 VisualStudio 不好好工作，没有将 xaml 和 xaml.cs 文件折叠到一起，此时可以通过 DependentUpon 的配置，手动将其进行合并代码，如以下代码例子，详细请看 [VisualStudio 合并代码文件](https://blog.lindexi.com/post/VisualStudio-%E5%90%88%E5%B9%B6%E4%BB%A3%E7%A0%81%E6%96%87%E4%BB%B6.html )

```xml
    <Compile Include="BlankPage1.xaml.cs">
      <DependentUpon>BlankPage1.xaml</DependentUpon>
      <SubType>Code</SubType>
    </Compile>
```

如此即可在实现自己的 UI 框架的时候，进行 XAML 的支持，且让开发者在开发过程中也有比较好的 XAML 智能提示功能

代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/a68446eac510fa6a80757abafde4c2bffd963cda/BellikarjeHakurheekall) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/a68446eac510fa6a80757abafde4c2bffd963cda/BellikarjeHakurheekall) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin a68446eac510fa6a80757abafde4c2bffd963cda
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin a68446eac510fa6a80757abafde4c2bffd963cda
```

获取代码之后，进入 BellikarjeHakurheekall 文件夹，即可获取到源代码