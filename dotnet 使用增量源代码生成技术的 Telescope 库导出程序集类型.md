# dotnet 使用增量源代码生成技术的 Telescope 库导出程序集类型

本文将告诉大家在 dotnet 里面使用免费完全开源的基于增量源代码生成技术的 Telescope 库，进行收集导出项目程序集里面指定类型。可以实现性能极高的指定类型收集，方便多模块对接入自己的业务框架

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

此 Telescope 库是基于最友好的 MIT 协议开源的，免费开源可商用：[https://github.com/dotnet-campus/SourceFusion](https://github.com/dotnet-campus/SourceFusion)

在日常开发过程中，也许会有这样的需求：将项目程序集里面的某种特征的类型们收集起来，用于实现自己的业务需求。比如说自己写了某些工作器，这些工作器类型都是继承 IWorker 接口的，此时业务上期望有某个逻辑可以将其收集导出，方便对接到自己业务上的框架

或者是自己写了某些过程过滤器类型，这些过滤器类型都继承 IFilter 接口，期望能够从项目里面导出收集，方便接入 IoC 容器或者是自动注入到过滤框架里面

此时可选的实现方法是通过反射，找到程序集里面满足条件的类型，对齐进行处理。然而反射的性能是不高的，再加上需要扫描一次程序集，性能就更低了。同时扫描程序集可能导致在启动过程中存在性能问题，比如扫描程序集导致更多依赖程序集被立刻加载，从而降低启动性能

本文将和大家介绍的是我所在的 dotnet 职业技术学苑（dotnet campus）组织开源的 Telescope 库。此 Telescope 库原本就是一个预编译库，在源代码生成技术 SourceGenerator 推出之前早已有此功能。有一个小道消息是 dotnet 的源代码生成技术有部分可能也受到到此库的启发（我脸皮是不是有点厚）哈

在 dotnet 推出了 IIncrementalGenerator 增量 Source Generator 源代码生成技术之后，我也对 Telescope 库进行稍微的更改，推出了基于增量源代码生成技术的版本，下面来看看此库的使用方法和功能

按照 dotnet 惯例，先安装 NuGet 库。可以右击项目管理 NuGet 包安装 [dotnetCampus.Telescope.SourceGeneratorAnalyzers](https://www.nuget.org/packages/dotnetCampus.Telescope.SourceGeneratorAnalyzers) 库，也可以编辑 csproj 项目文件添加以下代码安装

```xml
<PackageReference Include="dotnetCampus.Telescope.SourceGeneratorAnalyzers" Version="0.10.7-alpha17">
  <PrivateAssets>all</PrivateAssets>
  <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
</PackageReference>
```

这里需要提到的是 Telescope 的基于增量源代码生成技术的版本是完全的开发者工具侧的库，完全是修改源代码而不需要引入额外的库。换句话说就是使用 Telescope 库可以在发布自己的项目的时候，可以不会有额外的 DLL 引入。这对于许多商用项目都是非常棒的，不会让自己的输出被污染，不会让自己的安装包里面包含了其他组织制作的库

当然了，需要再次提醒一下，这个 Telescope 是基于免费的 MIT 协议的，完全可以商用的，无任何纠纷问题，可以放心使用

完成了基础库的安装之后，接下来就来编写演示的代码了。假定项目程序集里面有如下的一些类型，比如名为 Base 的基础类型，以及名为 FooAttribute 的特性。接下来的任务就是找到程序集里面所有继承 Base 基础类型且标记了 FooAttribute 特性的类型

大家可以假想一下，在自己的项目里面，那些需要反射扫描整个项目程序集才能实现的代码，看看能否套用到这里。如果可以的话，那推荐来试试这个 Telescope 库，看能否给你的项目提升一些性能

```csharp
class Base
{
}

class FooAttribute : Attribute
{
}
```

为了方便演示，这里再创建两个类型，用来继承 Base 基础类型且标记了 FooAttribute 特性

```csharp
[FooAttribute]
class F1 : Base
{
}

[FooAttribute]
class F2 : Base
{
}
```

现在咱的任务是收集项目程序集定义的继承 Base 基础类型且标记了 FooAttribute 特性的类型，如以上的 F1 和 F2 类型

请看一下使用 Telescope 的收集方式的代码

```csharp
internal partial class Program
{
    static void Main(string[] args)
    {
        foreach (var (type, attribute, creator) in ExportFooEnumerable())
        {
        }
    }

    [dotnetCampus.Telescope.TelescopeExportAttribute()]
    private static partial IEnumerable<(Type type, FooAttribute attribute, Func<Base> creator)> ExportFooEnumerable();
}
```

可以看到用法非常简单，只需要一个分部方法，在方法上标记了 TelescopeExportAttribute 特性即可，没有其他多余的侵入代码

可以看到这里的导出代码是通过 `partial` 的方式实现源代码生成对接的，只需要编写一个 partial 类型，在这个 partial 类型里面包含一个 partial 的方法，要求这个方法有满足条件的导出返回值，再给方法标记特性，即可自动生成导出类型的代码

如以上的代码即可在 Main 里面的遍历找到了 F1 和 F2 两个类型

更具体的用法要求是标记了 TelescopeExportAttribute 特性的方法的返回值有一定的要求。如要求使用的是 `IEnumerable` 等类型，且里面使用 ValueTuple 方式。这个 ValueTuple 的形式大概固定，格式如下

```csharp
(Type type, FooAttribute attribute, Func<Base> creator)
```

首个参数将会返回收集的类型的 Type 值，比如收集到 F1 那将会是 `typeof(F1)` 的类型。第二个参数表示要求类型继续标记的特性，如此即可让代码可以有更好的控制。第三个参数是 `Func<T>` 这里的 T 是表示要求收集的类型必须继承的基类型，可以是类型或接口

导出类型的方法名没有要求，方法的修饰也没有要求，也就是可以是 private 也可以是 public 的等等，可以是静态的也可以是非静态的

通过以上的方式即可在增量源代码生成里面生成出自动收集类型的代码，可以规避使用反射带来的性能损耗，同时也能更好的支持 AOT 打包

除此之外还有许多高级的功能，比如说收集的类型不限于当前项目程序集，也能收集到当前项目的所有依赖项。如果想要收集到依赖程序集里面的类型，可以在 TelescopeExportAttribute 里面加上对 IncludeReferences 属性的设置即可，如以下代码

```csharp
internal partial class Program
{
    [dotnetCampus.Telescope.TelescopeExportAttribute(IncludeReferences = true)]
    private static partial IEnumerable<(Type type, FooAttribute attribute, Func<Base> creator)> ExportFooEnumerable();
}
```

加上了 IncludeReferences 将会自动收集到满足要求的所有类型，包括当前项目引用的程序集。但必须说明的是加上了 IncludeReferences 属性设置为 true 将会在 Telescope 里收集引用的程序集类型，可能导致开发过程中的卡顿，但也只会影响开发，不会影响到程序运行

更多关于我博客请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
