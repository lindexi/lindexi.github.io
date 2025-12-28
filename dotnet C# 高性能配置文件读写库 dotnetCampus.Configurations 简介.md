# dotnet C# 高性能配置文件读写库 dotnetCampus.Configurations 简介

在应用程序运行的时，需要根据不同的配置执行不同的内容。有很多根据配置而初始化的功能往往是在应用程序启动的时候需要执行。对于很多类型的应用程序，特别是客户端的应用程序，启动的性能特别重要。也因此，在启动过程中需要依赖配置文件的不同配置而启动不同的功能时，就对配置文件的读写和解析性能提出了很高的要求

本文来和大家简单介绍我团队开源的 dotnetCampus.Configurations 高性能配置文件读写库。这个库不仅包含了配置文件的读取解析，还包括了自定义配置文件格式，也就是 COIN 硬币格式的配置文件。提供了多线程和多进程的读写安全的功能和毫秒级的配置文件读取解析性能，以及最低支持到 .NET Framework 4.5 框架

<!--more-->
<!-- CreateTime:2021/7/18 19:43:35 -->

<!-- 发布 -->
<div id="toc"></div>

## 背景

我有很多个客户端 .NET 应用程序，我需要在客户端启动的过程中，读取一些配置文件，包括机器级配置和用户级配置。原本一开始我的应用程序都是采用先启动通用逻辑，将通用界面显示出来，接着慢慢去读取配置文件，根据配置文件展开不同的功能

后面产品变心了，加了一些有趣的功能。如换肤等功能，此时就需要在第一个界面显示出来之前就需要读取配置了。我写了另一篇博客 [C# 配置文件存储 各种序列化算法性能比较](https://blog.lindexi.com/post/C-%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E5%AD%98%E5%82%A8-%E5%90%84%E7%A7%8D%E5%BA%8F%E5%88%97%E5%8C%96%E7%AE%97%E6%B3%95%E6%80%A7%E8%83%BD%E6%AF%94%E8%BE%83.html) 告诉大家各个配置文件的读取性能和序列化解析性能

但是现在通用的 XML 或 JSON 或 INI 等格式的性能，尽管看起来足够快了，但放在启动过程这个业务里面，依然显得性能不够。在启动的流程，每一个毫秒都是非常重要的。于是我所在的 CBB 公共组件团队就对配置文件的读取和解析有了性能上的要求，在基准测试机器人，能够在 10 毫秒内完全读取完成一份基准的配置文件。然而对于通用如上几个格式的文件来说，几乎没有一个能在小于 90 毫秒内完成。这就使得我需要去寻找一个更快的配置文件读写方式

在后续的产品迭代中，有几个产品的应用是允许用户多开的，开启多个进程的时候，也需要进行读写相同的一个配置文件。此时就出现另一个问题，如何保证配置的读写是进程级安全的

综合考虑了之后，在太子的带领下，开发和开源了 dotnetCampus.Configurations 硬币格式的高性能配置文件读写库

为什么叫硬币 COIN 呢，原因是取自 `COIN = Configuration\n` 即“配置+换行符”，因默认使用“\n”作为换行符而得名

## 开源

这是基于最友好的 MIT 协议的在 GitHub 完全开源的仓库，请看 [https://github.com/dotnet-campus/dotnetCampus.Configurations](https://github.com/dotnet-campus/dotnetCampus.Configurations)

此配置文件库完全百分百使用 C# 编写，支持如下 .NET 框架

- netstandard2.0
- net45
- netcoreapp3.0

等等 .NET 5 和 .NET 6 呢？在 .NET 5 或更高版本将会自动使用 .NET Core 3.0 的库，放心，这是完全 IL 级兼容的。为什么要有 .NET Standard 2.0 的？ 因为还要给 Xamarin 做兼容哦。对于 .NET Framework 系列的，最低要求是 .NET Framework 4.5 版本，对于更高的 .NET Framework 版本，也将会自动引用 .NET Framework 4.5 版本，放心，这也是完全 IL 级兼容的

本库已在超过 500 万台设备上稳定运行超过一年时间，还请放心使用

## 使用方法

介绍了那么多，是时候来看看此配置文件库的使用方法

按照惯例，在使用 .NET 库只需要两步，第一是通过 NuGet 安装，第二是开始使用。本文的硬币格式的高性能配置库也是通过 NuGet 分发的，包含了两个分支版本，分别是传统的 DLL 版本的 NuGet 和源代码两个版本。为了方便起见，咱先来介绍传统的 DLL 版本的使用方法

右击项目管理 NuGet 程序包，在浏览里面搜寻 dotnetCampus.Configurations 进行安装

在命令行使用如下代码即可给项目安装上硬币格式的高性能配置文件读写库

```
dotnet add package dotnetCampus.Configurations
```

除了使用命令行安装之外，对于 SDK 风格的新 csproj 项目格式的项目，可以编辑 csproj 文件，在 csproj 文件上添加如下代码进行安装

```xml
<PackageReference Include="dotnetCampus.Configurations" Version="1.6.8" />
```

使用硬币格式的高性能配置文件读写库时，需要传入配置文件所在的路径，如以下代码

```csharp
// 使用一个文件路径创建默认配置的实例。文件可以存在也可以不存在，甚至其所在的文件夹也可以不需要提前存在。
// 这里的配置文件后缀名 coin 是 Configuration\n，即 “配置+换行符” 的简称。你也可以使用其他扩展名，因为它实际上只是 UTF-8 编码的纯文本而已。
var configs = DefaultConfiguration.FromFile(@"C:\Users\lvyi\Desktop\walterlv.coin");
```

在获取到 configs 变量之后，即可对此变量进行读写，如下面代码

获取值：

```csharp
// 获取配置 Foo 的字符串值。
// 这里的 value 一定不会为 null，如果文件不存在或者没有对应的配置项，那么为空字符串。
string value0 = configs["Foo"];

// 获取字符串值的时候，如果文件不存在或者没有对应的配置项，那么会使用默认值（空传递运算符 ?? 可以用来指定默认值）。
string value1 = configs["Foo"] ?? "anonymous";
```

设置值：

```csharp
// 设置配置 Foo 的字符串值。
configs["Foo"] = "lvyi";

// 可以设置为 null，但你下次再次获取值的时候却依然保证不会返回 null 字符串。
configs["Foo"] = null;

// 可以设置为空字符串，效果与设置为 null 是等同的。
configs["Foo"] = "";
```

## 在大型项目中使用

实际应用中，应该将 configs 缓存起来，而不是每次使用的时候，都通过 DefaultConfiguration.FromFile 去创建新的对象

初始化：

```csharp
// 这里是大型项目配置初始化处的代码。
// 此类型中包含底层的配置读写方法，而且所有读写全部是异步的，防止影响启动性能。
var configFileName = @"C:\Users\lvyi\Desktop\walterlv.coin";
var config = ConfigurationFactory.FromFile(configFileName);

// 如果你需要对整个应用程序公开配置，那么可以公开 CreateAppConfigurator 方法返回的新实例。
// 这个实例的所有配置读写全部是同步方法，这是为了方便其他模块使用。
// 以下是 Container 即是容器，放入到容器中相当于全局单例
Container.Set<IAppConfigurator>(config.CreateAppConfigurator());
```

在业务模块中定义类型安全的配置类：

```csharp
internal class StateConfiguration : Configuration
{
    /// <summary>
    /// 获取或设置整型。
    /// </summary>
    internal int? Count
    {
        get => GetInt32();
        set => SetValue(value);
    }

    /// <summary>
    /// 获取或设置带默认值的整型。
    /// </summary>
    internal int Length
    {
        get => GetInt32() ?? 2;
        set => SetValue(Equals(value, 2) ? null : value);
    }

    /// <summary>
    /// 获取或设置布尔值。
    /// </summary>
    internal bool? State
    {
        get => GetBoolean();
        set => SetValue(value);
    }

    /// <summary>
    /// 获取或设置字符串。
    /// </summary>
    internal string Value
    {
        get => GetString();
        set => SetValue(value);
    }

    /// <summary>
    /// 获取或设置带默认值的字符串。
    /// </summary>
    internal string Host
    {
        get => GetString() ?? "https://localhost:17134";
        set => SetValue(Equals(value, "https://localhost:17134") ? null : value);
    }

    /// <summary>
    /// 获取或设置非基元值类型。
    /// </summary>
    internal Rect? Screen
    {
        get => this.GetValue<Rect>();
        set => this.SetValue<Rect>(value);
    }
}
```

在业务模块中使用：

```csharp
private readonly IAppConfiguration _config = Container.Get<IAppConfigurator>(); // 从 Container 容器获取，相当于从单例获取对象

// 读取配置。
private void Restore()
{
    var config = _config.Of<StateConfiguration>();
    var bounds = config.Screen;
    if (bounds != null)
    {
        // 恢复窗口位置和尺寸。
    }
}

// 写入配置。
public void Update()
{
    var config = _config.Of<StateConfiguration>();
    config.Screen = new Rect(0, 0, 3840, 2160);
}
```

## 兼容 Microsoft.Extensions.Configuration 配置

默认在 ASP.NET Core 里面将使用 Microsoft.Extensions.Configuration 配置，硬币格式的高性能配置文件读写库于此也是兼容的。但需要额外再安装一个兼容层的 NuGet 库，通过右击项目管理 NuGet 程序包，在浏览里面搜寻 dotnetCampus.Configurations.MicrosoftExtensionsConfiguration 进行安装

安装完成之后，可以通过扩展方法 ToAppConfigurator 从 IConfigurationBuilder 或者 IConfiguration 创建 IAppConfigurator 对象，通过 IAppConfigurator 即可重新接入到硬币格式的高性能配置文件读写库

```csharp
public void Foo(IConfigurationBuilder builder)
{
     IAppConfigurator appConfigurator = builder.ToAppConfigurator();
     // 完成接入
}

public void Foo(IConfiguration configuration)
{
     IAppConfigurator appConfigurator = configuration.ToAppConfigurator();
     // 完成接入
}
```

如在拿到 appConfigurator 变量之后，即可和上文一样的代码访问配置

```csharp
private void Foo(IConfiguration configuration)
{
    IAppConfigurator appConfigurator = configuration.ToAppConfigurator();
    var config = appConfigurator.Of<StateConfiguration>();
    // 读取配置。
    var bounds = config.Screen;
    if (bounds != null)
    {
        // 恢复窗口位置和尺寸。
    }
    // 写入配置。
    config.Screen = new Rect(0, 0, 3840, 2160);
}
```

## 配置文件格式

配置文件读写库的性能，除了代码层面的影响，更重要的是配置文件格式的影响。为了做到尽可能的高性能，于是重新设置了一套配置文件格式，这就是 COIN 硬币配置文件格式

配置格式如下

配置文件以行为单位，将行首是 `>` 字符的行作为注释，在 `>` 后面的内容将会被忽略。在第一个非 `>` 字符开头的行作为 `Key` 值，在此行以下直到文件结束或下一个 `>` 字符开始的行之间内容作为 `Value` 值

```
> 配置文件
> 版本 1.0
State.BuildLogFile
xxxxx
> 注释内容
Foo
这是第一行
这是第二行
>
> 配置文件结束
```

此配置文件格式不支持树型结构，而是 Key-Value 方式。作为配置文件是足够的，但是作为存储文件格式却是不适合的，这就是和 XML 和 JSON 最大的差别

## 特性

1. 高性能读写
    - 在初始化阶段使用全异步处理，避免阻塞主流程。
    - 使用特别为高性能读写而设计的配置文件格式。
    - 多线程和多进程安全高性能读写
1. 无异常设计
    - 所有配置项的读写均为“无异常设计”，你完全不需要在业务代码中处理任何异常。
    - 为防止业务代码中出现意料之外的 `NullReferenceException`，所有配置项的返回值均不为实际意义的 `null`。
        - 值类型会返回其对应的 `Nullable<T>` 类型，这是一个结构体，虽然有 `null` 值，但不会产生空引用。
        - 引用类型仅提供字符串，返回 `Nullable<ConfigurationString>` 类型，这也是一个结构体，你可以判断 `null`，但实际上不可能为 `null`。
1. 全应用程序统一的 API
    - 在大型应用中开放 API 时记得使用 `CreateAppConfigurator()` 来开放，这会让整个应用程序使用统一的一套配置读写 API，且完全的 IO 无感知。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
