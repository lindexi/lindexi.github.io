# dotnet SemanticKernel 入门 注入日志

使用 SemanticKernel 框架在对接 AI 时，由于使用到了大量的魔法，需要有日志的帮助才好更方便定位问题，本文将告诉大家如何在 SemanticKernel 注入日志

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

本文属于 SemanticKernel 入门系列博客，更多博客内容请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

在 KernelBuilder 创建器里面可以通过 WithLogger 注入 ILogger 类型的日志对象。咱既可以自己定义一个类型继承 ILogger 类型，也可以使用官方的日志库

本文将使用官方的日志库作为例子，告诉大家如何在 SemanticKernel 注入日志

按照 dotnet 的习俗，使用官方的日志库的第一步就是通过 NuGet 安装库，可以编辑 csproj 项目文件如下代码用来快速安装 Microsoft.Extensions.Logging.Console 库

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net7.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Extensions.Logging.Console" Version="7.0.0" />
    <PackageReference Include="Microsoft.SemanticKernel" Version="0.20.230821.4-preview" />
  </ItemGroup>

</Project>
```

这里必须说明的是 `Microsoft.Extensions.Logging.Console` 是一个通用的日志库，而不是一个专门给 ASP.NET Core 专用的库，上回我就遇到一位新手开发者误解了这个问题。官方的日志库是不单设计给到 ASP.NET Core 使用的，也可以在控制台或 WPF 应用或 WinForms 应用上使用的。另外值得一提的是以上我提到的三个框架也是非常方便的进行相互之间引用的，也就是说可以在一个项目里面同时使用上这三个框架

按照官方的日志库的通用做法，先是创建出 LoggerFactory 日志工厂，接着在工厂里面调用 AddConsole 加上控制台日志，最后调用 KernelBuilder 的注入，如以下代码

```csharp
var loggerFactory = LoggerFactory.Create(builder =>
{
    builder.AddConsole();
    builder.SetMinimumLevel(LogLevel.Debug);
});
var logger = loggerFactory.CreateLogger("SemanticKernel");

IKernel kernel = new KernelBuilder().WithLogger(logger).Build();
```

通过以上代码即可完成将日志模块注入到 SemanticKernel 里

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/84c1e073be77bee177607596b5e03cabb0c0a719/SemanticKernelSamples/Example03_Variables) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/84c1e073be77bee177607596b5e03cabb0c0a719/SemanticKernelSamples/Example03_Variables) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 84c1e073be77bee177607596b5e03cabb0c0a719
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 84c1e073be77bee177607596b5e03cabb0c0a719
```

获取代码之后，进入 SemanticKernelSamples\Example03_Variables 文件夹