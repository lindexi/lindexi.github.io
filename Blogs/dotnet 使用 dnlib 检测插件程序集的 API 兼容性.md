---
title: dotnet 使用 dnlib 检测插件程序集的 API 兼容性
description: 本文将和大家介绍在开发 dotnet 的插件时，如何通过 dnlib 库检测当前的插件是否由于主应用程序的版本差异导致存在 API 兼容性问题

<!--more-->

tags: dotnet
category: 
---

<!-- CreateTime:2024/2/19 9:31:09 -->

<!-- 发布 -->
<!-- 博客 -->

众所周知，在开发插件的过程中，插件与主程序之间的兼容性问题将持续是一个令开发者烦恼的事情。举个例子，我开发的插件是面向 1.0 版本的主程序开发了，我需要用到 A 类型的 B 方法。结果在我插件发布一段时间之后，我的主程序更新到 2.0 版本了，此版本的主程序更改了 A 类型的 B 方法，比如删除了 B 方法，或者修改了 B 方法的函数参数。那么此时我的插件将会与主程序存在 API 不兼容问题，强行运行将会导致运行过程中抛出找不到成员的异常

本文介绍的 dnlib 库，可以用来辅助检测，当前的插件是否和主程序存在不兼容的问题。可以预先知道是否存在兼容问题，从而可以更好的给出用户交互

具体使用方法如下，按照 dotnet 的惯例，先安装 dnlib 库。可以通过如下方式编辑 csproj 项目文件，添加如下代码用来快速安装

```xml
  <ItemGroup>
    <PackageReference Include="dnlib" Version="4.4.0" />
  </ItemGroup>
```

接下来编写一个名为 CompatibilityChecker 的静态类型，将在此类型实现通过 dnlib 提供的功能进行兼容性检测

在 CompatibilityChecker 添加一个名为 CheckCompatibility 的方法，此方法将可以用来检测输入程序集是否存在与主程序的兼容性问题。方法定义如下

```csharp
using dnlib.DotNet;

static class CompatibilityChecker
{
    /// <summary>
    /// 检查插件API兼容性
    /// </summary>
    /// <param name="assemblyFilePath"></param>
    /// <param name="searchPathList"></param>
    /// <returns>
    /// result: true表示API兼容 false表示不兼容
    /// missingMembers: 缺失的API
    /// exception: 检测过程中的异常
    /// </returns>
    public static (bool result, List<MemberRef> missingMembers, Exception? exception) CheckCompatibility(string assemblyFilePath, List<string> searchPathList)
    {
        try
        {
            var missingMembers = CompatibilityChecker.GetMembersRef(assemblyFilePath, searchPathList).GetMissingMembers();
            return (!missingMembers.Any(), missingMembers, null);
        }
        catch (Exception e)
        {
            return (false, new List<MemberRef>(0), e);
        }
    }
}
```

以上代码的 GetMembersRef 则需要将程序集进行分析加载，此分析加载过程中并非将程序集加入到当前程序域内，仅仅只是做二进制分析而已

```csharp
    private static IEnumerable<MemberRef> GetMembersRef(string filePath, List<string> searchPathList)
    {
        var context = new ModuleContext(new AssemblyResolverWithSearchPathList(searchPathList));
        var module = ModuleDefMD.Load(filePath, context);
        return GetModuleMembersRef(module);
    }
```

以上的 AssemblyResolverWithSearchPathList 类型为自定义类型，作用就是根据输入的程序集依赖寻找路径列表，执行程序集依赖寻找策略。这个类型为本文所需要的核心实现方法，其核心原理就是通过 dnlib 的分析，读取程序集依赖寻找路径，查找是否存在某些依赖成员无法找到，从而了解是否存在兼容性问题

```csharp
class AssemblyResolverWithSearchPathList : AssemblyResolver
{
    public AssemblyResolverWithSearchPathList(List<string> searchPathList)
    {
        SearchPathList = searchPathList;
    }

    private List<string> SearchPathList { get; }

    protected override IEnumerable<string> GetModuleSearchPaths(ModuleDef module) => base.GetModuleSearchPaths(module).Concat(SearchPathList);
}
```

以上的 GetModuleMembersRef 方法为获取当前 Module 的成员引用，其实现方法如下

```csharp
    private static IEnumerable<MemberRef> GetModuleMembersRef(ModuleDefMD module)
    {
        return module.GetMemberRefs()
            .Select(x => (member: x, assembly: x.DeclaringType.DefinitionAssembly))
            .Where(x => x.assembly != module.Assembly)
            .Where(x => x.assembly is not null) // 如果存在动态程序集，那这里可能拿到空值
            .Where(x => !IgnoreAssemblies.Contains(x.assembly.Name.ToString())) // 这是可选的
            .Select(x => x.member);
    }
```

以上代码里面将过滤出依赖的成员，同时通过 IgnoreAssemblies 加入一些可供忽略的程序集。这些程序集是我实际开发过程中，发现 dnlib 支持较弱的，代码如下

```csharp
    private static readonly HashSet<string> IgnoreAssemblies = new()
    {
        "Microsoft.CSharp",
        "mscorlib",
        "PresentationCore",
        "PresentationFramework",
        "System",
        "System.Collections",
        "System.Core",
        "System.Diagnostics.Debug",
        "System.Drawing",
        "System.Globalization",
        "System.IO",
        "System.IO.Compression",
        "System.Linq",
        "System.Linq.Expressions", // 尝试解决 dynamic 找不到 CallSite 的锅
        "System.Net.Http",
        "System.Reflection",
        "System.Reflection.Extensions",
        "System.Resources.ResourceManager",
        "System.Runtime",
        "System.Runtime.Extensions",
        "System.Runtime.InteropServices",
        "System.Text.Encoding",
        "System.Threading",
        "System.Threading.Tasks",
        "System.ValueTuple",
        "System.Windows.Forms",
        "System.Xaml",
        "WindowsBase",
        // 以下这个库会提示找不到 get_PageSize 和 Render 方法
        "PdfiumViewer",
        // 以下的几个库提示找不到方法，细节我还不知道
        "OpenAI-DotNet",
        "Azure.AI.OpenAI",
        "Azure.Core",
        "Microsoft.SemanticKernel.Core",
        "Microsoft.SemanticKernel",
        "Microsoft.SemanticKernel.Abstractions",
        "Microsoft.SemanticKernel.Planning.ActionPlanner",
        "Microsoft.SemanticKernel.Planning.SequentialPlanner",
        "Microsoft.SemanticKernel.Skills.Core",
        "Microsoft.SemanticKernel.Connectors.AI.OpenAI",
    };
```

如果没有忽略这几个程序集，可能插件程序集在寻找依赖是否缺失的过程中，将会寻找失败或者是提示以上程序集里面必定存在某些缺失的成员

最后的 GetMissingMembers 方法则是通过判断其引用成员是否 Resolve 失败，返回失败的列表，代码如下

```csharp
    private static List<MemberRef> GetMissingMembers(this IEnumerable<MemberRef> members) => members.Where(x => x.Resolve() == null).ToList();
```

如此即可完成 CompatibilityChecker 类型的实现，下面来看看其使用方法

首先是获取需要检测的插件程序集所在的文件路径，作为 `filePath` 参数传入，这个属于大家自己的业务逻辑，还请自行解决。接下来构建 依赖寻找文件夹路径列表，一般来说插件程序集所在的文件夹里面可能包含插件本身所需依赖，于是先将插件程序集所在文件夹加入到依赖寻找文件夹路径列表里，代码如下

```csharp
    var searchPathList = new List<string>();
    var directoryName = Path.GetDirectoryName(filePath);
    if (directoryName != null)
    {
        searchPathList.Add(directoryName);
    }
    else
    {
        // 见鬼了，这个文件不在文件夹里？
    }
```

接下来将主应用程序所在的文件夹也加入到 依赖寻找文件夹路径列表 里面

最后需要将 dotnet 系列依赖加入，比如我的 dotnet 依赖是打到主应用程序里面的，参考 [记将一个大型客户端应用项目迁移到 dotnet 6 的经验和决策](https://blog.lindexi.com/post/%E8%AE%B0%E5%B0%86%E4%B8%80%E4%B8%AA%E5%A4%A7%E5%9E%8B%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%BA%94%E7%94%A8%E9%A1%B9%E7%9B%AE%E8%BF%81%E7%A7%BB%E5%88%B0-dotnet-6-%E7%9A%84%E7%BB%8F%E9%AA%8C%E5%92%8C%E5%86%B3%E7%AD%96.html )

我需要使用如下代码将应用程序所使用的定制版本的 dotnet 加入到依赖寻找列表，如以下代码

```csharp
    var dotnetRuntimeFolderRoot = Path.Combine(mainApplicationPath, @"..\runtime\shared\Microsoft.NETCore.App\");
    if (Directory.Exists(dotnetRuntimeFolderRoot))
    {
        var dotnetRuntimeFolder = Directory
            .GetDirectories(dotnetRuntimeFolderRoot, "*", SearchOption.TopDirectoryOnly).FirstOrDefault();
        if (dotnetRuntimeFolder != null)
        {
            searchPathList.Add(dotnetRuntimeFolder);
        }
    }
```

对于 WPF 和 WinForms 项目，我还需要将 Microsoft.WindowsDesktop.App 也加入到依赖寻找列表，如以下代码

```csharp
    var desktopRuntimeFolderRoot = Path.Combine(mainApplicationPath, @"..\runtime\shared\Microsoft.WindowsDesktop.App\");
    if (Directory.Exists(desktopRuntimeFolderRoot))
    {
        var desktopRuntimeFolder = Directory.GetDirectories(desktopRuntimeFolderRoot, "*", SearchOption.TopDirectoryOnly).FirstOrDefault();
        if (desktopRuntimeFolder != null)
        {
            searchPathList.Add(desktopRuntimeFolder);
        }
    }
```

完成依赖寻找列表之后，即可调用 CheckCompatibility 方法，如以下代码

```csharp
    var (result, missingMembers, exception) = CompatibilityChecker.CheckCompatibility(filePath, searchPathList);
```

通过判断 result 即可知道当前的插件程序集是否和主应用程序之间存在兼容问题，且通过 missingMembers 可以了解存在哪些 API 不兼容

通过此方法即可判断插件是否与主应用程序存在兼容性问题，从而更好进行用户界面交互
