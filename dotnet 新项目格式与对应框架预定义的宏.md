# dotnet 新项目格式与对应框架预定义的宏

在 sdk style 的项目格式支持使用多框架开发，此时需要在代码里面通过宏判断，在编译的时候执行不同的代码。本文告诉大家在框架里面对应的预定义的条件编译符有哪些

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


在[让一个 csproj 项目指定多个开发框架 - walterlv](https://blog.walterlv.com/post/configure-projects-to-target-multiple-platforms.html )告诉大家如何在 sdk style 项目格式如何定义多框架开发，在使用多框架开发的时候，有时会使用到特定的平台框架提供的方法，此时就需要使用预定义的宏通过条件判断编译符号的方法，在不同的框架编译不同的代码

例如下面代码根据在不同的平台输出不同的代码

```csharp
    static void Main()
    {
#if NET40
        Console.WriteLine("Target framework: .NET Framework 4.0");
#elif NET45  
        Console.WriteLine("Target framework: .NET Framework 4.5");
#else
        Console.WriteLine("Target framework: .NET Standard 2.0");
#endif
    }
```

以上写法是通过条件编译符，将会在不同条件满足的时候编译出不同的 IL 代码，关于条件编译符请看官方文档[#if 预处理器指令](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-if )

此时需要知道在 sdk style 的项目文件里面给不同的平台添加了哪些条件编译符可以使用

以下是预定义的宏

## .NET Framework

所有 .NET Framework 都定义 `NETFRAMEWORK` 宏，此外对应不同的版本使用不同的宏

- NET20
- NET35
- NET40
- NET45
- NET451
- NET452
- NET46
- NET461
- NET462
- NET47
- NET471
- NET472
- NET48

## .NET Standard

所有 .NET Standard 都定义 `NETSTANDARD` 宏，此外对应不同的版本使用不同的宏

- NETSTANDARD1_0
- NETSTANDARD1_1
- NETSTANDARD1_2
- NETSTANDARD1_3
- NETSTANDARD1_4
- NETSTANDARD1_5
- NETSTANDARD1_6
- NETSTANDARD2_0

## .NET Core

所有 .NET Core 都定义 `NETCOREAPP` 宏，此外对应不同的版本使用不同的宏

- NETCOREAPP1_0
- NETCOREAPP1_1
- NETCOREAPP2_0
- NETCOREAPP2_1
- NETCOREAPP2_2

本文说到的宏是古老的词汇，在官方用语里面叫条件编译符，也叫预处理符号(preprocessor symbols) 在 Roslyn 的语法分析中，就是这样写的

在 csproj 或 Target 中可以采用下面代码判断宏是否包含

```csharp
Condition="$(DefineConstants.Contains(NET30))"
```

[.NET/C# 项目如何优雅地设置条件编译符号？ - walterlv](https://blog.walterlv.com/post/how-to-define-preprocessor-symbols.html )

[在 Roslyn 分析语法树时添加条件编译符号的支持 - walterlv](https://blog.walterlv.com/post/roslyn-syntax-tree-supporting-preprocessor-symbols.html )

[Target frameworks](https://docs.microsoft.com/en-us/dotnet/standard/frameworks )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
