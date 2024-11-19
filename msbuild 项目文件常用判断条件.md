# msbuild 项目文件常用判断条件

在写项目文件的时候，需要根据不同的条件定义或执行不同的代码，有一些比较常使用的判断，本文收藏起来，方便大家抄代码

<!--more-->
<!-- CreateTime:2019/11/29 8:36:48 -->

<!-- 标签：Roslyn,MSBuild,编译器 -->


在 msbuild 的项目文件 cspoj 或 xx.target 等文件里面，可以使用 Condition 条件写在很多标签元素作为判断条件

例如以下代码，在 Target 上面添加条件，只有条件满足了才会执行

```xml
  <Target Name="Lindexi" AfterTargets="CoreCompile" Condition="'$(Configuration)|$(TargetFramework)'=='DEBUG|net45'">
    <Message Text="林德熙是逗比"></Message>
  </Target>
```

以上代码的名为 Lindexi 的 Target 将在 DEBUG 模式下，且 TargetFramework 为 net45 才执行

本文是 [手把手教你写 Roslyn 修改编译](https://lindexi.oschina.io/lindexi/post/roslyn.html ) 的系列文章，阅读本文之前，期望大家已经了解了 csproj 或 msbuild 等的基础语法和知识。如对此不了解，推荐先阅读 [MSBuild 如何编写带条件的属性、集合和任务 Condition？ - walterlv](https://blog.walterlv.com/post/how-to-write-msbuild-conditions.html )

下面将告诉大家一些常使用的判断的写法或代码片

## 判断在调试下编译

请看代码

```xml
Condition="'$(Configuration)'=='Debug'"
```

这里 Configuration 的判断是不区分大小写的，默认写的是 `Debug` 字符串。而全大写的 `DEBUG` 则更多是用在条件编译里面

例如这样写即可表示在 Debug 模式下的条件满足下，将在 PropertyGroup 添加 MainProjectPath 属性的赋值

```xml
  <PropertyGroup Condition=" '$(Configuration)' == 'Debug'">
      <MainProjectPath>blog.lindexi.com</MainProjectPath>
  </PropertyGroup>
```

## 判断在发布下编译

请看代码

```xml
Condition="'$(Configuration)'!='Debug'"
```

也就是上面代码反过来判断不是在调试下编译

另一个判断方法请看代码，这是不推荐的判断方法

```xml
Condition="'$(Configuration)'=='Release'"
```

这个不推荐的写法，一般只有调试下和非调试下，用上面的写法可能有逗比写了 `Release-x` 于是就判断不是发布下，此时就没有做发布的优化



## 判断平台

判断在 .NET Framework 4.5 运行

```xml
Condition="'$(TargetFramework)'=='net45'"
```

对应的判断 .NET Standard 使用如下缩写 `netstandard1.0` 等

判断 .NET Core 1.0 使用如下缩写 `netcoreapp1.0` 等

对应关系如下：

- .NET Standard ： `netstandard1.0`
- .NET Core 1.0 ： `netcoreapp1.0`

判断是否 Windows App SDK 可使用 GetTargetPlatformIdentifier 辅助，如判断 `net6.0-windows10.0.19041` 版本

```xml
 <ItemGroup Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'windows'">

 </ItemGroup>
```

判断系统和架构平台可使用 `$(RuntimeIdentifier)` 属性进行判断。此 `$(RuntimeIdentifier)` 属性在日常 Build 构建过程中是没有值的，一般只有发布的时候或其他带上 `-r` 参数构建的时候才会有值，如以下常见发布命令

```bash
dotnet publish -r linux-loongarch64
```

- 判断 Windows x86 平台： `Condition="$(RuntimeIdentifier) == 'win-x86'"`
- 判断 Windows x64 平台： `Condition="$(RuntimeIdentifier) == 'win-x64'"`
- 判断 Linux arm64 平台： `Condition="$(RuntimeIdentifier) == 'linux-arm64'"`
- 判断 Linux x64 平台： `Condition="$(RuntimeIdentifier) == 'linux-x64'"`
- 判断 Linux 龙芯平台：`Condition="$(RuntimeIdentifier) == 'linux-loongarch64'"`

## 多个判断

需要同时生效有两个写法，如判断只有在 .NET Framework 4.5 同时在调试下才生效，第一个写法的代码如下

```xml
Condition="'$(Configuration)|$(TargetFramework)'=='DEBUG|net45'"
```

第二个方法是使用关键字 And 连接

```xml
Condition=" '$(TargetFramework)'=='net45' And $(Configuration)=='Debug'"
```

显然以上代码里面第一个方法代码比较简短。第二个方法比较灵活，可以进行更复杂的逻辑

两个条件的或判断使用关键字 Or 连接

```xml
Condition=" '$(TargetFramework)'=='net45' or $(Configuration)=='Debug'"
```

## 包裹单引号

大家可以看到常写的判断里面会包裹单引号，但又似乎可以不写。什么时候应该写？对于空值的判断、包含特殊符号的情况等。什么时候可以不写？单引号对于简单的字母数字字符串是不必要的，对于布尔值来说也是不必要的。如果记不清的话，推荐判断符号左右两边都包裹单引号

如简单的字母数字字符串：

```xml
Condition="$(TargetFramework)==net45"
```

如上例子，等于号左右两边都不加单引号是可以的。在以上例子里面使用的都是简单的字母数字字符串，单引号是可选的

但下面的例子包含了 `-` 字符，推荐加上单引号

```xml
Condition="$(RuntimeIdentifier) == 'linux-arm64'"
```

很多时候为了平衡等于号左右两边，也会给左值加单引号，即如下代码

```xml
Condition="'$(RuntimeIdentifier)' == 'linux-arm64'"
```

对于组合判断来说，左右值需加上单引号，如下面例子

```xml
Condition="'$(Configuration)|$(TargetFramework)'=='DEBUG|net9.0'"
```

如不加单引号，则会将 `|` 字符当成控制字符之类，导致提示 MSB4090 错误

简单说就是啥都不管，判断条件的左右都给加上单引号好了。值得说明的是，如果判断条件的值内部已经包含单引号了，如 `Exists('Lindexi.txt')` 代码，则不能再使用单引号包裹表达式。如下面代码是错误的

```xml
'Exists(Lindexi.txt)' == 'true'
```

## 判断宏

如以下代码即可判断是否定义了名为 `NET30` 的宏或条件编译符

```xml
Condition="$(DefineConstants.Contains(NET30))"
```

## 判断框架版本大于或等于

使用 VersionGreaterThanOrEquals 的方式，传入 TargetFrameworkVersion 进行判断。如以下代码，判断 TargetFramework 是否大于或等于 3.0 的版本

```xml
$([MSBuild]::VersionGreaterThanOrEquals('$(TargetFrameworkVersion)', '3.0')
```

以上的 `TargetFrameworkVersion` 是在 SDK 获取的，代码如下

```xml
<TargetFrameworkVersion>v$([MSBuild]::GetTargetFrameworkVersion('$(TargetFramework)', 2))</TargetFrameworkVersion>
```

## 判断文件夹是否以斜杠结尾

如下面代码，判断 BaseIntermediateOutputPath 是否以斜杠结尾，否则就加上斜杠

```xml
    <BaseIntermediateOutputPath Condition="!HasTrailingSlash('$(BaseIntermediateOutputPath)')">$(BaseIntermediateOutputPath)\</BaseIntermediateOutputPath>
```

## 判断文件存在

在条件判可以用 `Exists` 判断文件或文件夹是否存在，如以下代码

```xml
  <Target Name="StanalurJikecair" AfterTargets="CoreCompile" Condition="Exists('$(OutputPath)')">
    <Message Text="$(OutputPath)"></Message>
  </Target>

  <Target Name="ZurwelSowselnu" AfterTargets="CoreCompile" Condition="!Exists('$(OutputPath)')">
    <Message Text="不存在$(OutputPath)"></Message>
  </Target>
```

可以看到两个代码的不相同，使用 `!` 可以判断为 原来是相同的就返回`false`的值，即取反的方式，这里的 `$(OutputPath)` 是存在的，所以编译会输出类似下面代码

```
StanalurJikecair:
  bin\Debug\netcoreapp2.0\
```

## 判断当前构建的系统平台

判断当前运行 msbuild 或 dotnet 命令所在的系统平台，可使用 `[MSBuild]::IsOSPlatform` 进行判断。如下面判断是跑在 Windows 平台

```xml
Condition="$([MSBuild]::IsOSPlatform('Windows'))"
```

如下面判断是跑在 Linux 平台上

```xml
Condition="$([MSBuild]::IsOSPlatform('Linux'))"
```

以上为判断当前构建时所运行的平台，而不是当前正在构建哪个平台的包

有什么作用呢？比如我有一些平台相关的工具，需要根据当前构建时所运行的平台，决定使用哪个工具进行打包。那么判断系统平台就是正确使用这些工具的前提

举个例子，我有一个生成特殊格式签名的工具，这个工具的作用就是根据我的某个生成的资源文件，生成平台相关的签名文件。这个生成特殊格式签名的工具本身是平台相关的，我的 dotnet 应用会在多个系统上进行构建，如在一台 Windows 机器、一台 Linux-x64 机器、一台龙芯 Linux 机器上构建。在不同的机器上需要使用不同的平台版本的生成特殊格式签名的工具。于是就期望能够判断构建版本选用不同的工具

实现的方法就是使用 `[MSBuild]::IsOSPlatform` 判断系统，和使用 `$([System.Runtime.InteropServices.RuntimeInformation]::OSArchitecture)` 判断架构，例子代码如下

```xml
    <PropertyGroup>
      <FooToolName Condition="$([MSBuild]::IsOSPlatform('Windows')) And $([System.Runtime.InteropServices.RuntimeInformation]::OSArchitecture)=='x64'">Tool_win-x64.exe</FooToolName>
      <FooToolName Condition="$([MSBuild]::IsOSPlatform('Linux')) And $([System.Runtime.InteropServices.RuntimeInformation]::OSArchitecture)=='x64'">Tool_linux-x64</FooToolName>
      <FooToolName Condition="$([MSBuild]::IsOSPlatform('Linux')) And $([System.Runtime.InteropServices.RuntimeInformation]::OSArchitecture)=='loongarch64'">Tool_linux-loongarch64</FooToolName>
    </PropertyGroup>

    <Exec Command="$(FooToolName) --sign-resource lindexi.g.app" />
```

## 判断当前构建 dotnet sdk 版本

当前构建的 dotnet sdk 版本不同于项目所使用的 TargetFramework 版本，这点还请大家分清楚。构建的 dotnet sdk 版本是当前构建机器所安装的 dotnet sdk 版本

判断当前构建过程中使用的是 dotnet 9 或以上的 SDK 版本：

```xml
Condition="$([MSBuild]::VersionGreaterThanOrEquals($(NETCoreSdkVersion), 9.0))"
```

## 更多判断

[Roslyn 在项目文件使用条件判断](https://blog.lindexi.com/post/Roslyn-%E5%9C%A8%E9%A1%B9%E7%9B%AE%E6%96%87%E4%BB%B6%E4%BD%BF%E7%94%A8%E6%9D%A1%E4%BB%B6%E5%88%A4%E6%96%AD.html )

[MSBuild 如何编写带条件的属性、集合和任务 Condition？ - walterlv](https://blog.walterlv.com/post/how-to-write-msbuild-conditions.html )

[Target frameworks](https://docs.microsoft.com/en-us/dotnet/standard/frameworks?wt.mc_id=MVP )

[手把手教你写 Roslyn 修改编译](https://blog.lindexi.com/post/roslyn.html )

[dotnet 打包 NuGet 的配置属性大全整理](https://blog.lindexi.com/post/dotnet-%E6%89%93%E5%8C%85-NuGet-%E7%9A%84%E9%85%8D%E7%BD%AE%E5%B1%9E%E6%80%A7%E5%A4%A7%E5%85%A8%E6%95%B4%E7%90%86.html )

[项目文件中的已知 NuGet 属性（使用这些属性，创建 NuGet 包就可以不需要 nuspec 文件啦） - walterlv](https://blog.walterlv.com/post/known-nuget-properties-in-csproj )

[项目文件中的已知属性（知道了这些，就不会随便在 csproj 中写死常量啦） - walterlv](https://blog.walterlv.com/post/known-properties-in-csproj.html )