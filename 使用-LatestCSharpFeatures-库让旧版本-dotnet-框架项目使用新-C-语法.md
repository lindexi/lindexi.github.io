
# 使用 LatestCSharpFeatures 库让旧版本 dotnet 框架项目使用新 C# 语法

本文将和大家介绍 dotnet campus 开源组织的 LatestCSharpFeatures 库，通过 LatestCSharpFeatures 库可以让使用旧版本的 dotnet 框架的项目可以应用上更多最新版本的 C# 语法

<!--more-->


<!-- CreateTime:2024/12/14 07:06:05 -->

<!-- 发布 -->
<!-- 博客 -->

## 背景

众所周知，在咱 dotnet 体系里面，框架和语言版本是分离的。即使 C# 的版本支持情况只和编译器有关，而与运行时版本无直接关系

换句话说就是即使现在还在使用老旧的 .NET Framework 4.5 的 dotnet 版本，也依然可以使用 C# 13 甚至更高版本的语法

但为什么有时候会给一些新手以错觉，认为 C# 语言版本是跟随 dotnet 版本走的？这个错觉是因为有两个主要问题导致的

首要问题是 dotnet 是分 SDK 和运行时这两个东西，编译器是放在 SDK 里面，即至少 SDK 版本需要更加新的版本才能支持更加新的 C# 语法。但 SDK 是跑在开发者的电脑上的，版本新或高是没有任何问题的，新的 SDK 高版本是完全可以构建出旧的 dotnet 版本的项目的。运行时是跑在用户设备上的，如上文举例的 .NET Framework 4.5 的 dotnet 版本，一般而言指的是配置在 csproj 项目里面，指定运行时版本，运行时版本可以旧一些。用新的 SDK 高版本构建旧的运行时的应用是完全没有问题的，整个 dotnet 体系的兼容性就是做的这么好。有些新手开发者分不清 SDK 版本和运行时版本，错将用于构建的 SDK 版本号当成运行时版本号，从而有了认为 C# 语言版本是跟随 dotnet 版本走的错误知识

次之的原因是高版本的 C# 语言可能用到了一些类型，这些类型是只有在新版本的 dotnet 框架里才有定义的。这就导致了默认情况下，由于有些语法对应的类型在低版本 dotnet 框架找不到，从而导致开发者无法在低版本的 dotnet 使用高版本 C# 语法

本文和大家介绍的 dotnet campus 开源组织的 LatestCSharpFeatures 库正是用来解决上文次之的原因提到的问题，通过 LatestCSharpFeatures 库补全缺失的 C# 高版本语法所需的类型，从而让低版本的 dotnet 项目可以放心使用高版本 C# 语法

仓库开源地址： <https://github.com/dotnet-campus/dotnetCampus.LatestCSharpFeatures>

## 用法

按照 dotnet 的惯例，第一步是安装 NuGet 包： https://www.nuget.org/packages/dotnetCampus.LatestCSharpFeatures

对于这个库来说，就没有第二步了，安装完成之后，即可在项目里面放心使用高版本 C# 语法了

这个 LatestCSharpFeatures 库的版本号有一个特殊约定，那就是第一位主版本号就是对应的 C# 语法版本号了哈

## 原理

既然用法如此简单，那自然就有伙伴好奇这个库的原理了。其实在 LatestCSharpFeatures 库里面是没有什么科技的，真正的科技都在 C# dotnet 这边

按照 C# 和 dotnet 的设计，语言是独立于框架的存在，也就是 C# 是独立于 dotnet 的存在。在构建过程中，高版本 C# 语法所需的一些类型，只需要这些类型能够被找到即可。类型能够被找到只需满足命名空间和类型名符合约定即可，至于这个类型具体放在哪里，这不是编译器所关心的。这就意味着高版本 C# 语法所需的辅助类型，只要在项目里面有能找到定义就好了，至于具体是放在 dotnet 运行时里还是在项目里面定义的，那都随意。通过如此设计，即可让 C# 和 dotnet 进一步分离，尽管要实现某些 C# 甜甜语法，需要用到一些辅助类型，但是没有规定这些类型一定要在 dotnet 基础库里面找到

额外的，如上文所述规定，只是要求满足命名空间和类型名符合约定，完全没有要求访问权限是什么，也就是在项目里面放 internal 还是 public 的辅助类型，都是可以的

基于 C# 和 dotnet 的如此巧妙的设计，自然就可以很方便写出一个源代码生成器项目，让源代码生成器生成高版本 C# 所需的辅助类型，再配合默认的条件编译符的方式，只在低版本的 dotnet 项目开放代码内容。如此即可让低版本的 dotnet 项目使用高版本的 C# 语法

举个例子，在 C# 9 引入的 init 语法，就要求存在 `System.Runtime.CompilerServices.IsExternalInit` 辅助类。这个 IsExternalInit 辅助类型是在 dotnet 5 引入的。如果要在 .NET Core 3.1 的项目上使用 init 语法，可以在自己的项目上添加以下代码

```csharp
using System.ComponentModel;

namespace System.Runtime.CompilerServices
{
    /// <summary>
    /// Reserved to be used by the compiler for tracking metadata.
    /// This class should not be used by developers in source code.
    /// This dummy class is required to compile records when targeting .NET Standard
    /// </summary>
    [EditorBrowsable(EditorBrowsableState.Never)]
    internal static class IsExternalInit
    {
    }
}
```

在 LatestCSharpFeatures 库里面，就使用源代码生成器技术，将以上代码塞入到项目里面

可能此时大家有疑惑，那如果我的项目是 dotnet 6 框架的呢？这里塞入的 IsExternalInit 类型会不会和 dotnet 基础库提供的冲突了？答案是如果就只有以上代码，那肯定是冲突的。为了解决冲突，就加上条件编译符，加上之后的代码如下

```csharp
#if NET5_0_OR_GREATER
#else
using System.ComponentModel;

namespace System.Runtime.CompilerServices
{
    /// <summary>
    /// Reserved to be used by the compiler for tracking metadata.
    /// This class should not be used by developers in source code.
    /// This dummy class is required to compile records when targeting .NET Standard
    /// </summary>
    [EditorBrowsable(EditorBrowsableState.Never)]
    internal static class IsExternalInit
    {
    }
}
#endif
```

先判断当前的框架版本是多少，如果是 .NET 5 或更高版本，则啥都不用干。否则加塞进入 IsExternalInit 类型

通过如此方式即可实现让低于 dotnet 5 版本的项目使用到 C# 9 的 init 语法

喜欢探寻研究技术的伙伴也许会有更多疑惑，为什么我知道可以使用 `NET5_0_OR_GREATER` 条件编译符来判断当前的框架版本？源代码生成器是如何实现的？我在以下附加了一些链接，希望能帮助大家

- [dotnet 新项目格式与对应框架预定义的宏](https://blog.lindexi.com/post/dotnet-%E6%96%B0%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F%E4%B8%8E%E5%AF%B9%E5%BA%94%E6%A1%86%E6%9E%B6%E9%A2%84%E5%AE%9A%E4%B9%89%E7%9A%84%E5%AE%8F.html)
- [尝试 IIncrementalGenerator 进行增量 Source Generator 生成代码](https://blog.lindexi.com/post/%E5%B0%9D%E8%AF%95-IIncrementalGenerator-%E8%BF%9B%E8%A1%8C%E5%A2%9E%E9%87%8F-Source-Generator-%E7%94%9F%E6%88%90%E4%BB%A3%E7%A0%81.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。