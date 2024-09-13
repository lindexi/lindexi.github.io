现在在 .NET 系列里面，势头比较猛的 UI 框架中，就包括了 Avalonia 和 UNO 框架。本文将告诉大家如何尝试在一个解决方案里面融合 Avalonia 和 UNO 两个框架，即在一个进程里面跑起来两个框架

<!--more-->


<!-- CreateTime:2024/06/23 07:07:27 -->

<!-- 发布 -->
<!-- 博客 -->

开始之前先聊会背景故事

我比较看好 Avalonia 的现在和 UNO 的未来。但是我不怎么想在 Avalonia 的基础上造基础库和设施。我大概是在 2017 年的时候就参与了 Avalonia 的开发，但是随着更深入的投入发现了 Avalonia 团队的一些问题，那会感觉到 Avalonia 更像是一个玩具、一个实验场，而不是一个可产品化的应用（这里说的是很久之前的，不是说现在的，现在是产品级可用，支持大型项目）。核心原因在于有一些意见上没能和我达成一致。一个框架开发需要比较全面的能力和知识，有一些知识属于特定领域的。但是 Avalonia 团队里面缺乏这部分知识，且很多时候都是拍脑袋按照自己想法进行实现的。这就导致了一些专业的模块实现过于奇怪。其输入法、文本相关、触摸相关部分实现都比较糟心。界面布局方式，以及一些基础实现和我预期相差较大。再加上版本之间的 API 稳定性和行为稳定性，导致了我不想在 Avalonia 的基础上进行投入基础库的制造和基础设施的搭建

好在 2023 的下半年（准确来说是3月，但是刚开始没有什么影响），进了一位 CEO 到 Avalonia 团队，在这个期间给 Avalonia 带来极大的提升，直接从一个玩具级提升到产品级。这个过程中 Avalonia 做了相当多的工作，包括进行了大规模的重构，大量基础设施的建设，优化了非常多的开发调试的能力。整体开发 Avalonia 起来也是非常舒坦，且有了支持大型项目的能力。得益于 Avalonia 非常长的开源时间作为底蕴，从 2013 年开源至今，在 Avalonia 框架里面积累了大量的跨平台经验，特别是在 Linux 的桌面端应用上的经验，进行了非常多的适配。再加上 2023 的[下半年](https://x.com/MikeCodesDotNET/status/1630893746253438976)进了 [Mike James](https://it.linkedin.com/in/micjames) 作为 Avalonia 的 CEO 角色，让 Avalonia 有了非常多的资源投入，以及拉动了非常多相关方的支持，使得 Avalonia 迎来一大波激进的优化。优化方面包括了框架底层到上层 API 的重构，也包括了拉来了 JetBrains 的 ReSharper 和 [Rider](https://www.jetbrains.com/guide/dotnet/links/unleashing-the-power-of-cross-platform-development-with-avalonia-ui-and-jetbrains-rider/) 的官方支持使得开发调试等各种方面的有了非常大的优化

可以这么说，整个 Avalonia 的开发体验，从 2023 的下半年可以和之前作为一个巨大的划分点。让我打分的话，之前是不及格，现在是 90 分

但是 Mike James 在 2023 的 11 月 [跑路](https://www.reddit.com/r/AvaloniaUI/comments/17uxsmk/mike_james_to_step_down_from_ceo_position_at/ )了，这就使得从 2024 的上半年开始，整个 Avalonia 的混乱程度又上来了。好在现在 Mike James 又回到 Avalonia 团队了，期待后续 Avalonia 团队的进步

以下是从 <https://theorg.com/org/avalonia-ui/org-chart/mike-james> 里面拷贝的 Mike James 简介，可以看到他是很厉害的且有经验的

> Mike James has a diverse work experience in the technology industry. Mike is currently serving as the Chief Executive Officer of Avalonia UI since March 2023. Prior to that, they worked at Microsoft from 2016 to 2023, where they held various roles including Senior Developer Advocate, Technical Solutions Professional, and Program Manager II. From 2013 to 2017, Mike worked at Xamarin as a Developer Evangelist/Advocate and a Customer Support Engineer. Mike started their career at Pharos Architectural Controls Ltd in 2010 as a Development Support Technician.
 
这也就是为什么我看好 Avalonia 的现在的原因。当然了 Mike James 是一个原因，客套来说其整个团队也都功不可没。那接下来继续聊一下 UNO 框架

整个 UNO 框架起初是建立在 WinUI 的侧边的，即在现有的 WinUI 或 UWP 应用里面，使用 UNO 框架将其构建出跨平台的版本。这样做的策略是 UNO 框架可以复用 UWP 的基础设施和 API 设计。从一开始上就规避了 Avalonia 里面混乱的 API 设计和基础设施。但是缺点也很明显，就是 WinUI 的 API 设计比 Win32 前辈差太多了，且 UWP 也砍掉了大量的 WPF 能力，导致了 UNO 被 WinUI 所拖累。再加上 UNO 开源时间还短，距今仅有 6 年时间，再再叠加上 UNO 同时在啃食全平台，即移动端 和 WASM 和桌面端，工作量非常庞大，导致了完善程度不如 Avalonia 高

但 UNO 的优势在于有强有力的控制管理，这和以前（特指 2023 之前）的容易受到社区投毒的 Avalonia 有着巨大的不同，其交付能力有所保证。其次是 UNO 的整体开发团队投入和卷的程度看起来比 Avalonia 更大得多。最后是使用了 WinUI 的 API 组织方式进行兜底，以及参考了 WinUI3 的设计，确保了很多专业性模块上的实现正确性。最后一点是和 Avalonia 策略上的差别，在 UNO 上是宁可不实现也尽量不给出知识性错误的实现方式，而 Avalonia 则是别人有我就得有，不管是否水土不服。再加上 WinUI 和 MAUI 团队对 UNO 的帮衬，让 UNO 的整体技术更加全面。这就是我比较看好 UNO 的未来的原因

那如何现在我就需要开发呢？我敢不敢全用 UNO 呢？如果是桌面端的话，不敢，因为现在的 UNO 在桌面端完全不够 Avalonia 打的。选 Avalonia 呢，但是我的基础库和基础设施还是需要造的，一旦选 Avalonia 就意味着我有大量的测试实验需要做，去测试 Avalonia 的各种行为，且可能在下个 Avalonia 版本发布之后，我的这些测试实验和基础库就全都白干了，因为 Avalonia 就进行了不兼容的修改变更了。嗯…在 Avalonia 的 11.0 和 11.1 已经干了这件事了。既然我看好 UNO 的未来，那不如基础库就在 UNO 的基础上造咯。即使我说 UNO 在桌面端完全不够 Avalonia 打的，但是作为基础库所需的基础能力，还是能够提供的

于是我就选择了上层应用使用 Avalonia 做，底层一些基础设施使用 UNO 做。我就有了两个框架在一个应用里面，于是就有了本文的 融合 Avalonia 和 UNO 框架到一个解决方案里面。以下是 Avalonia 对此的评价：

> Avalonia, unlike Uno, isn't a direct reimplementation of any existing framework but a modern, cross-platform toolkit inspired by WPF. Avalonia focuses on rendering the entire UI independently, akin to Flutter, ensuring consistent, high-quality visuals on all platforms. In contrast, Uno aims to replicate the UWP & WinUI API across platforms, utilising native primitives similar to MAUI. This makes Avalonia ideal for creating a uniform, flexible UI design across every platform. We advise developers to try both technologies when considering Avalonia vs Uno Platform and decide which they prefer.

我就听了他 `We advise developers to try both technologies` 半句话，好的，那就尝试两个框架咯

为什么不是合并 Avalonia 和 UNO 框架到一个项目里面？这个想法太可怕了，这两个框架都是进行了大量且深度的黑科技研发的，能够在一个解决方案里面共存能活就好了

以下是我给出的最简的让 Avalonia 和 UNO 框架跑在一个进程上的方法

分别新建 Avalonia 和 UNO 框架两个项目的最简模式，其中 Avalonia 框架命名为 AvaloniaIDemo 项目，将 UNO 框架命名为 UnoDemo 框架。我特别推荐大家先拿空项目进行测试，玩明白了再修改复杂的项目

最简跑起来的方法就是让 UnoDemo 项目引用 AvaloniaIDemo 项目，这时候看起来构建什么的都没有问题。为了测试将 Avalonia 跑起来，修改 UnoDemo 项目的 MainPage.xaml 文件，添加一个按钮，点击这个按钮可以将 Avalonia 框架跑起来，代码如下

```xml
<Page x:Class="UnoDemo.MainPage"
      xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
      xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
      xmlns:local="using:UnoDemo"
      Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
  <StackPanel
        HorizontalAlignment="Center"
        VerticalAlignment="Center">
    <TextBlock AutomationProperties.AutomationId="HelloTextBlock"
          Text="Hello Uno Platform"
          HorizontalAlignment="Center" />
    <Button x:Name="RunAvaloniaButton" HorizontalAlignment="Center" Content="Run Avalonia" Click="RunAvaloniaButton_OnClick"/>
  </StackPanel>
</Page>
```

后台代码实现如下

```csharp
    private void RunAvaloniaButton_OnClick(object sender, RoutedEventArgs e)
    {
        // Create the new Thread to run the Avalonia 
        
        var thread = new Thread(() =>
        {
            AvaloniaIDemo.Program.Main([]);
        })
        {
            IsBackground = true,
            Name = "Avalonia main thread"
        };
        if (OperatingSystem.IsWindows())
        {
            thread.SetApartmentState(ApartmentState.STA);
        }
        thread.Start();
    }
```

这里需要新建一个后台线程给 Avalonia 跑，因为 Avalonia 一跑就会卡住线程，只有在 Avalonia 应用退出时才会退出卡住线程逻辑

额外说明为什么不用 Task 的方式跑，而是选择 Thread 的原因，这是因为 Task 默认走线程池，线程池可不开森你拿一个线程跑长时间的任务，这样会占用线程池的资源。对于此业务情况下，需要长时间运行的，那就是自己开 Thread 更好

以上就是最基础的实现方法了

本文以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/6ef2308fd744d276c4db076ac58efaad7b0d1c25/AvaloniaIDemo/WawjejokeniDejeyaibeweji) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/6ef2308fd744d276c4db076ac58efaad7b0d1c25/AvaloniaIDemo/WawjejokeniDejeyaibeweji) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 6ef2308fd744d276c4db076ac58efaad7b0d1c25
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 6ef2308fd744d276c4db076ac58efaad7b0d1c25
```

获取代码之后，进入 AvaloniaIDemo/WawjejokeniDejeyaibeweji 文件夹，即可获取到源代码

可以试试拉取我的代码，打开 AvaloniaIDemo\WawjejokeniDejeyaibeweji\AvaloniaIDemo.sln 文件，然后切换 UnoDemo 作为启动项目，选择 UnoDemo(Desktop) 而不是 UnoDemo(WinAppSDK Packaged) 跑起来，再试试点击按钮看看是否能够弹出 AvaloniaIDemo 的窗口

此时跑起来的应用是 Avalonia+WPF+Uno 三个框架的，哈哈，为什么这里会加上 WPF 呢？这是因为 UNO 在 Windows 的底层就是 WPF 框架承接。而 Avalonia 是自己对接 Win32 层，没有中间商

可以看到本文的这个方式做的是比较浅的融合，窗口级相互引用而已。更深层次的融合现在可行性不高，欢迎大家自行摸索

以下是我的更多踩坑经验

## 找不到 SDK 项目添加不上来

如果一开始新建的 sln 文件是对 Avalonia 项目的，那么将可以在添加现有 UNO 项目时，发现 VisualStudio 不给添加，提示报错信息如下

找不到指定的 SDK "Uno.Sdk" 项目无法添加

这个原因是在 sln 文件相同的文件夹下找不到包含 Uno.Sdk 定义的 global.json 文件。只需在 sln 文件相同的文件夹下放一个 global.json 文件，里面的内容代码大概如下

```json
{
  // To update the version of Uno please update the version of the Uno.Sdk here. See https://aka.platform.uno/upgrade-uno-packages for more information.
  "msbuild-sdks": 
  {
    "Uno.Sdk": "5.2.161"
  }
}
```

以上的 5.2.161 版本号，还请修改为你创建 UNO 项目时的选用版本号。或者直接将 UNO 项目的 global.json 文件拷贝过去也可以

这是因为在此版本时，新建的 UNO 项目的 csproj 项目文件里使用了 UNO 自己制作的 Uno.Sdk 而不是 Microsoft.NET.Sdk 框架

以下为 UNO 项目的 csproj 项目文件的示例代码

```xml
<Project Sdk="Uno.Sdk">
    ... 忽略其他代码
</Project>
```

以下为其他控制台或 Avalonia 项目的 csproj 项目文件的示例代码

```xml
<Project Sdk="Microsoft.NET.Sdk">
    ... 忽略其他代码
</Project>
```

可以看到 Sdk 属性的不同

## 无法在 Avalonia 项目引用 UNO 项目

为什么在本文例子里面是使用 UNO 项目引用 Avalonia 项目，而不是反过来呢？这是因为在 UNO 的 5.2 版本里面，自创了名为 `netx.xx-desktop` 的框架。从 dotnet 设计上说，自己创建框架也是可行的，毕竟 dotnet 里面就有了 `netx.xx-windows` 等框架，用于区分平台

在 UNO 里，确实使用 `netx.xx-desktop` 框架可以让内部开发更加便利，实现桌面端的跨平台和移动端等的区分

但是这也导致了与其他现有设施对接时候的难点。现在 Avalonia 依然使用的是纯 dotnet 项目，这让 Avalonia 的构建非常简单且稳定。大家都知道，对构建过程更多的定制就一般意味着会有更多诡异的问题，现在的 UNO 就是这样。整体的构建不仅被 WinUI 拖累，还会有自己创建的框架的坑。即使 UNO 团队有专职的测试人员也架不住开发者复杂的开发环境投毒

如果让 Avalonia 项目引用 UNO 项目，将会构建失败，错误信息如下

```
error NU1201: 项目 UnoDemo 与 net8.0 (.NETCoreApp,Version=v8.0) 不兼容
```

其原因就是 UNO 使用的是 `net8.0-desktop` 框架，而 Avalonia 项目是 net8.0 框架的。从 dotnet 的 SDK 设计约束上就是 `net8.0-desktop` 框架范围比 net8.0 框架更大，不能让更小范围的框架引用更大的范围，这就是失败的原因

## 发布 Linux 平台失败

发布 linux 平台时，需要先在 Avalonia 项目里面进行一次发布，发布参数需要和 UNO 项目的相同。如在 AvaloniaIDemo 里面，选用 Release 加 linux-x64 的独立发布方式进行发布，再在 UNO 项目也选用 Release 加 linux-x64 的独立发布方式进行发布，如此才能发布成功

否则如果只是立刻在 UNO 项目里发布，则可能遇到 `未能找到元数据文件` 错误，失败信息大概如下

```
未能找到元数据文件“C:\lindexi\Code\AvaloniaIDemo\WawjejokeniDejeyaibeweji\AvaloniaIDemo\obj\Release\net8.0\linux-x64\ref\AvaloniaIDemo.dll”
```

这个原因大概是 Avalonia 也对发布做了些科技，导致的不兼容问题。更细节我没有继续研究

经过我的测试，如此方式发布之后，可以在 Ubuntu 和 UOS 两个 Linux 系统上运行，且工作符合预期 

## 让 Avalonia 依赖 net8.0 的 Uno 项目

由于 Uno 不仅可以跑 net8.0-desktop 框架，作为全平台支持的 UI 框架，天生也就支持单跑 net8.0 框架。尝试修改 UNO 的 csproj 项目文件为如下代码

```xml
<Project Sdk="Uno.Sdk">
  <PropertyGroup>
    <TargetFrameworks>
      net8.0-windows10.0.19041;
      net8.0-desktop;
      net8.0;
    </TargetFrameworks>

    ... 忽略其他代码
</Project>
```

此时就满足了给 Avalonia 引用的基础条件了，然而此时却会发现 Avalonia 经常无法创建生成代码，这是因为 Avalonia 所做的黑科技刚好和 Uno 所作的冲突，从而导致 Avalonia 无法成功从 axaml 生成代码

同时也存在了许多类型冲突，进一步导致了项目难以构建。如我的以下变更： <https://github.com/lindexi/lindexi_gd/commit/f7032fc9a952a073d1e4cdb5d81955e38019ac3c> 将会难以进行构建

即使通过仅引用程序集解决了引用问题，也会面临着运行不起来的问题。这是因为 Uno 只有在 desktop 下才拷贝真正的桌面运行时依赖，如 `Uno.UI.Runtime.Skia.Wpf.dll` 和 `Uno.UI.Runtime.Skia.X11.dll` 文件，缺少了这些文件的 Uno 程序集是无法正常运行的

且如果你的 IDE 是 Rider 的话，更会出现问题。在 Rider 里面，只会构建所需的框架，即使只对 UnoDemo 构建 net8.0 框架，而无视 net8.0-desktop 框架。尽管 Rider 这个是为了优化构建速度，但是也带来了更多问题，如现在就无法通过拷贝 net8.0-desktop 框架的内容到输出路径进行替换从而解决运行的问题

解决 Rider 的不构建的方法是采用解决方案的依赖的方式


## 类型冲突

由于 Avalonia 存在大量的类型和 UNO/WinUI 相同，这将会导致如此引用将会很难写代码

比如写一个 Thickness 都可能遇到以下错误信息

```
MainPage.xaml.cs(15,9,15,18): error CS0104: “Thickness”是“Avalonia.Thickness”和“Microsoft.UI.Xaml.Thickness”之间的不明确的引用
```

想要更好的解决是再新建一个作为入口的程序集，这个程序集依然是 UNO 框架的，分别引用 AvaloniaIDemo 和 UnoDemo 项目，只在此入口程序集做启动和实现对接，其他的事情都不要做

为了更好的实现对接，那一般还需要一个纯 dotnet 项目，这个项目是 API 定义项目，用于让互相不引用的 AvaloniaIDemo 和 UnoDemo 通过此 API 定义项目进行抽象对接

如此大家也可以看到通过这个方式开发具备一定的复杂性

接下来我将告诉大家这个方法

## 新入口程序集

看起来再新建一个程序集作为入口程序集也不错，此方式只是搭建稍微有点复杂而已，但能够确保 Avalonia 和 Uno 项目更具独立性

新建一个名为 AppDemo 的控制台项目，再新建一个名为 LibDemo 的控制台项目。断开 UnoDemo 和 AvaloniaIDemo 的联系。让 UnoDemo 和 AvaloniaIDemo 项目都引用 LibDemo 项目。让 AppDemo 同时引用 UnoDemo 和 AvaloniaIDemo 项目

完成引用之后的项目引用关系如下图

<!-- ![](image/dotnet 融合 Avalonia 和 UNO 框架/dotnet 融合 Avalonia 和 UNO 框架0.png) -->
![](https://img2023.cnblogs.com/blog/1080237/202409/1080237-20240913091325019-886855464.png)

为了让 AppDemo 控制台项目能够正确的引用上 UnoDemo 项目，需要修改项目文件，修改之后的代码大概如下

```xml
<Project Sdk="Uno.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0-desktop</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <UnoSingleProject>true</UnoSingleProject>
  </PropertyGroup>

  <ItemGroup>
    <ProjectReference Include="..\AvaloniaIDemo\AvaloniaIDemo.csproj" />
    <ProjectReference Include="..\UnoDemo\UnoDemo\UnoDemo.csproj" />
  </ItemGroup>

</Project>
```

核心就是设置 `<Project Sdk="Uno.Sdk">` 和设置框架版本以及加上 UnoSingleProject 属性

编辑 LibDemo 控制台项目，去掉 OutputType 定义，让其成为基础库项目，修改之后的 csproj 项目文件的代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

</Project>
```

现在 UnoDemo 和 AvaloniaIDemo 项目相互之间不引用，为了能够间接调用到，就需要让两个项目都依赖抽象

在 LibDemo 项目里面提供简单的接口和静态类型用于被注入，代码如下

```csharp
namespace LibDemo;

public interface IAppRunner
{
    void Run();
}
```

以上的 IAppRunner 用于给具体的框架使用，无论是 Avalonia 先启动也会，还是 Uno 先启动也好，只要后被启的框架实现此接口。再注入到下面定义的 Runner 静态方法里面，即可完成被调用的依赖

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace LibDemo;

public static class Runner
{
    public static void SetAppRunner(IAppRunner appRunner)
    {
        _appRunner = appRunner;
    }

    public static void Run() => _appRunner?.Run();

    private static IAppRunner? _appRunner;
}
```

在本文例子里面依然是 Uno 先起来，然后调起 Avalonia 项目，根据上文的设计，需要在 AvaloniaIDemo 项目里面定义具体的实现逻辑，代码如下

```csharp
using LibDemo;

namespace AvaloniaIDemo;

public static class Initializer
{
    public static void InitAssembly()
    {
        Runner.SetAppRunner(new AppRunner());
    }
}

file class AppRunner : IAppRunner
{
    public void Run()
    {
        Program.Main([]);
    }
}
```

以上的 `file class AppRunner` 使用到了 file 关键字，这是表示当前的 AppRunner 类型只有在当前文件可访问。通过此关键字可以更大程度进行控制访问范围

如此即可在 AppDemo 里面，通过调用 InitAssembly 方法完成 AvaloniaIDemo 项目的初始化，代码如下

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace AppDemo;

internal class Program
{
    [STAThread]
    public static void Main(string[] args)
    {
        AvaloniaIDemo.Initializer.InitAssembly();

        ... // 忽略其他代码
    }
}
```

完成 AvaloniaIDemo 的注入之后，在 UnoDemo 即可使用抽象的依赖调用，如以下代码

```csharp
    private void RunAvaloniaButton_OnClick(object sender, RoutedEventArgs e)
    {
        var thread = new Thread(() =>
        {
            Runner.Run();
        })
       
        ... // 忽略其他代码
    }
```

可以看到 UnoDemo 从原本的具体的 `AvaloniaIDemo.Program.Main([]);` 换成了间接的 `Runner.Run();` 调用

修改之后的 UnoDemo 的 MainPage.xaml.cs 的所有代码如下

```csharp
using LibDemo;

namespace UnoDemo;

public sealed partial class MainPage : Page
{
    public MainPage()
    {
        this.InitializeComponent();
    }
    
    private void RunAvaloniaButton_OnClick(object sender, RoutedEventArgs e)
    {
        // Create the new Thread to run the Avalonia 

        var thread = new Thread(() =>
        {
            Runner.Run();
        })
        {
            IsBackground = true,
            Name = "Avalonia main thread"
        };
        if (OperatingSystem.IsWindows())
        {
            thread.SetApartmentState(ApartmentState.STA);
        }
        thread.Start();
    }
}
```

完成基础逻辑之后，即可在 AppDemo 将 Uno 应用启动，代码如下

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace AppDemo;

internal class Program
{
    [STAThread]
    public static void Main(string[] args)
    {
        AvaloniaIDemo.Initializer.InitAssembly();

        UnoDemo.Program.Main(args);
    }
}
```

接下来为了让整体构建更加简单，需要修改让 Avalonia 项目去掉 OutputType 属性，保持输出为基础库方式

```xml
<OutputType>WinExe</OutputType>
```

修改之后的 AvaloniaIDemo 的 csproj 项目文件的代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <BuiltInComInteropSupport>true</BuiltInComInteropSupport>
    <ApplicationManifest>app.manifest</ApplicationManifest>
    <AvaloniaUseCompiledBindingsByDefault>true</AvaloniaUseCompiledBindingsByDefault>
  </PropertyGroup>


  <ItemGroup>
    <PackageReference Include="Avalonia" Version="11.0.6" />
    <PackageReference Include="Avalonia.Desktop" Version="11.0.6" />
    <PackageReference Include="Avalonia.Themes.Fluent" Version="11.0.6" />
    <PackageReference Include="Avalonia.Fonts.Inter" Version="11.0.6" />
    <!--Condition below is needed to remove Avalonia.Diagnostics package from build output in Release configuration.-->
    <PackageReference Condition="'$(Configuration)' == 'Debug'" Include="Avalonia.Diagnostics" Version="11.0.6" />
  </ItemGroup>


  <ItemGroup>
    <ProjectReference Include="..\LibDemo\LibDemo.csproj" />
  </ItemGroup>
</Project>
```

可以看到 AvaloniaIDemo 项目没有对 UnoDemo 项目进行任何的引用

如此即可尝试直接 F5 运行。以及发布之后运行

以上方式我在 Windows 上 F5 直接运行成功，发布到 Ubuntu 和 UOS 上也能运行成功，看起来属于坑比较少的方式

本文以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/6cc41fa8c98001d034a59d20cad83386ae16b5aa/AvaloniaIDemo/WawjejokeniDejeyaibeweji) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/6cc41fa8c98001d034a59d20cad83386ae16b5aa/AvaloniaIDemo/WawjejokeniDejeyaibeweji) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 6cc41fa8c98001d034a59d20cad83386ae16b5aa
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 6cc41fa8c98001d034a59d20cad83386ae16b5aa
```

获取代码之后，进入 AvaloniaIDemo/WawjejokeniDejeyaibeweji 文件夹，即可获取到源代码
