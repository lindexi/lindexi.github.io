# Roslyn 在项目文件使用条件判断

本文告诉大家如何在项目文件通过不同的条件使用不同的方法运行

<!--more-->
<!-- CreateTime:2019/7/3 17:07:32 -->

<!-- csdn -->
<!-- 标签：Roslyn,MSBuild,编译器 -->

本文是 [手把手教你写 Roslyn 修改编译](https://lindexi.oschina.io/lindexi/post/roslyn.html ) 的文章，在阅读本文之前，希望已经知道了大多数关于 msbuild 的知识

为了告诉大家如何使用判断，需要创建一个简单的程序来写，这里创建一个 dotnet core 控制台项目，如果还没安装 dotnet core 可以到 [dotnet sdk 2.1.300 winx64-CSDN下载](https://download.csdn.net/download/lindexi_gd/10582416 )

从 VisualStudio 安装文件夹打开开发人员工具命令行，打开这个是因为如果没有将 `msbuild` 加入到环境变量，就需要使用这个方法来调用 `msbuild` 调用 msbuild 的原因是为了编译可以看到输出。

在 `Target` 如果用 `Message` 的输出，除了设置为错误，其他的输出在 VisualStudio 的输出是无法看到的，只能通过 `msbuild` 才可以看到。

打开开发人员工具，先进入到刚才创建的项目所在的文件夹，然后执行`msbuild`就可以看到输出

```csharp
C:\lindexi\github\RaskerYadeacorLalmi\RaskerYadeacorLalmi>msbuild
用于 .NET Framework 的 Microsoft (R) 生成引擎版本 15.7.180.61344
版权所有(C) Microsoft Corporation。保留所有权利。

生成启动时间为 2018/8/3 20:05:47。
节点 1 上的项目“C:\lindexi\github\RaskerYadeacorLalmi\RaskerYadeacorLalmi\RaskerYadeacorLalmi.csproj”(默认目标)。
GenerateTargetFrameworkMonikerAttribute:
正在跳过目标“GenerateTargetFrameworkMonikerAttribute”，因为所有输出文件相对于输入文件而言都是最新的。
CoreGenerateAssemblyInfo:
正在跳过目标“CoreGenerateAssemblyInfo”，因为所有输出文件相对于输入文件而言都是最新的。
CoreCompile:
正在跳过目标“CoreCompile”，因为所有输出文件相对于输入文件而言都是最新的。
StanalurJikecair:
  123
GenerateBuildDependencyFile:
正在跳过目标“GenerateBuildDependencyFile”，因为所有输出文件相对于输入文件而言都是最新的。
GenerateBuildRuntimeConfigurationFiles:
正在跳过目标“GenerateBuildRuntimeConfigurationFiles”，因为所有输出文件相对于输入文件而言都是最新的。
CopyFilesToOutputDirectory:
  RaskerYadeacorLalmi -> C:\lindexi\github\RaskerYadeacorLalmi\RaskerYadeacorLalmi\bin\Debug\netcoreapp2.0\RaskerYadeac
  orLalmi.dll
已完成生成项目“C:\lindexi\github\RaskerYadeacorLalmi\RaskerYadeacorLalmi\RaskerYadeacorLalmi.csproj”(默认目标)的操作。


已成功生成。
    0 个警告
    0 个错误
```

上面代码创建的是 `RaskerYadeacorLalmi` 项目，在这个项目所在的文件夹进行编译，这个项目是新创建的，只是在项目上写了下面代码

```csharp
  <Target Name="StanalurJikecair" AfterTargets="CoreCompile">
    <Message Text="123"></Message>
  </Target>
```

这是一个空白的`Target`，一个`Target`有一个 Name 和一个属性告诉在什么时候运行这个`Target`在这个`Target`里使用`Message`，就可以在编译时看到下面代码

```csharp
StanalurJikecair:
  123
```

如果重新运行`msbuild`会发现有很多都是跳过，具体请看[每次都要重新编译？太慢！让跨平台的 MSBuild/dotnet build 的 Target 支持差量编译 - walterlv](https://walterlv.github.io/post/msbuild-incremental-build.html )

```csharp
C:\lindexi\github\RaskerYadeacorLalmi\RaskerYadeacorLalmi>msbuild
用于 .NET Framework 的 Microsoft (R) 生成引擎版本 15.7.180.61344
版权所有(C) Microsoft Corporation。保留所有权利。

生成启动时间为 2018/8/3 20:05:47。
节点 1 上的项目“C:\lindexi\github\RaskerYadeacorLalmi\RaskerYadeacorLalmi\RaskerYadeacorLalmi.csproj”(默认目标)。
GenerateTargetFrameworkMonikerAttribute:
正在跳过目标“GenerateTargetFrameworkMonikerAttribute”，因为所有输出文件相对于输入文件而言都是最新的。
CoreGenerateAssemblyInfo:
正在跳过目标“CoreGenerateAssemblyInfo”，因为所有输出文件相对于输入文件而言都是最新的。
CoreCompile:
正在跳过目标“CoreCompile”，因为所有输出文件相对于输入文件而言都是最新的。
StanalurJikecair:
  warning:123
GenerateBuildDependencyFile:
正在跳过目标“GenerateBuildDependencyFile”，因为所有输出文件相对于输入文件而言都是最新的。
GenerateBuildRuntimeConfigurationFiles:
正在跳过目标“GenerateBuildRuntimeConfigurationFiles”，因为所有输出文件相对于输入文件而言都是最新的。
CopyFilesToOutputDirectory:
  RaskerYadeacorLalmi -> C:\lindexi\github\RaskerYadeacorLalmi\RaskerYadeacorLalmi\bin\Debug\netcoreapp2.0\RaskerYadeac
  orLalmi.dll
已完成生成项目“C:\lindexi\github\RaskerYadeacorLalmi\RaskerYadeacorLalmi\RaskerYadeacorLalmi.csproj”(默认目标)的操作。


已成功生成。
    0 个警告
    0 个错误
```

如果需要清理，重新编译，可以输入下面命令

```csharp
msbuild clean
```

现在可以尝试使用 `Conditions` 判断条件

使用 `Conditions` 很多时候都是使用字符串判断，如使用下面代码

```csharp
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp2.0</TargetFramework>
  </PropertyGroup>

  <Target Name="StanalurJikecair" AfterTargets="CoreCompile" Condition="$(TargetFramework)=='netcoreapp2.0'">
    <Message Text="123"></Message>
  </Target>
```

因为`TargetFramework`是 `netcoreapp2.0` 会运行这个 Target ，如果这时修改 `TargetFramework` 为 `net45` 就不会运行这个代码

```csharp
  <Target Name="StanalurJikecair" AfterTargets="CoreCompile" Condition="$(TargetFramework)=='net45'">
    <Message Text="123"></Message>
  </Target>
```

如果需要同时判断多个条件，如在 `Debug` 而且是 `net45` 就运行，可以使用下面代码

```csharp
  <Target Name="StanalurJikecair" AfterTargets="CoreCompile" Condition="'$(Configuration)|$(TargetFramework)'=='DEBUG|net45'">
    <Message Text="123"></Message>
  </Target>
```

注意在原来的`$(Configuration)`是可以不使用引号，但是现在使用了`|`就需要添加引号，表示这时字符串

实际的`|`不是语法，可以换为其他的字符，如下面的代码

```csharp

    <Target Name="StanalurJikecair" AfterTargets="CoreCompile" Condition="'$(TargetFramework)-$(OutputType)'=='netcoreapp2.0-Exe'">
      <Message Text="123"></Message>
    </Target>
```

上面代码使用 `'$(TargetFramework)-$(OutputType)'` 判断

## 判断不相等

如果需要判断不相同，只需要修改`==`为不相等

```csharp
    <Target Name="StanalurJikecair" AfterTargets="CoreCompile" Condition="'$(TargetFramework)'!='net45'">
      <Message Text="123"></Message>
    </Target>
```

现在运行`msbuild`可以看到输出了`123`如果修改为`'$(TargetFramework)'=='net45'`就判断不相等

## 判断大小

除了判断字符串，还可以判断字符串的大小，只能用来判断数值字符串，如果对于 16 进制的字符串，需要使用 `0x` 开始，如下面代码

```csharp
  <Target Name="StanalurJikecair" AfterTargets="CoreCompile" Condition="'60'&gt;'10'">
    <Message Text="因为60大于10所以会运行"></Message>
  </Target>

  <Target Name="LamaciswhaJoopisJerall" AfterTargets="CoreCompile" Condition="'60'&gt;='60'">
    <Message Text="因为60大于60所以会运行"></Message>
  </Target>

  <Target Name="TasyenatarReresetelRagearsu" AfterTargets="CoreCompile" Condition="'10' &lt; '60'">
    <Message Text="因为10小于60所以会运行"></Message>
  </Target>

  <Target Name="FurkeeneLafer" AfterTargets="CoreCompile" Condition="'60'&lt;='60'">
    <Message Text="因为60小于等于60所以会运行"></Message>
  </Target>

  <Target Name="JoudewalljeeZeargeaday" AfterTargets="CoreCompile" Condition="'0xAA'&gt;'10'">
    <Message Text="如果使用16进制需要使用0x放在字符串最前"></Message>
  </Target>
```

因为使用的文件是 xml 所以需要将会`>`转换为`&gt;`，将`<`转为 `&gt;` 如果输入的数值是16进制就需要使用 0x 放在开始，下面的代码就会出现下面的错误

```csharp
  <Target Name="SiscooLecem" AfterTargets="CoreCompile" Condition="'AA'&gt;'10'">
    <Message Text="如果使用16进制需要使用0x放在字符串最前"></Message>
  </Target>
```

```csharp
error MSB4086: 尝试在条件“'AA
'>'10'”中对计算结果为“AA”而不是数字的“AA”进行数值比较
```

## 判断文件存在

在条件判可以用 `Exists` 判断文件 文件夹是否存在

```csharp
  <Target Name="StanalurJikecair" AfterTargets="CoreCompile" Condition="Exists('$(OutputPath)')">
    <Message Text="$(OutputPath)"></Message>
  </Target>

  <Target Name="ZurwelSowselnu" AfterTargets="CoreCompile" Condition="!Exists('$(OutputPath)')">
    <Message Text="$(OutputPath)"></Message>
  </Target>
```

可以看到两个代码的不相同，使用 `!` 可以判断为 原来是相同的就返回`false`，这里的 `$(OutputPath)` 是存在的，所以编译会输出下面代码

```csharp
StanalurJikecair:
  bin\Debug\netcoreapp2.0\
```

## 判断多个条件

除了使用开始的使用 `-` 等连接多个判断还可以使用 `And` `Or` 来判断多个条件，如下面代码

```csharp
  <Target Name="StanalurJikecair" AfterTargets="CoreCompile" Condition="Exists('$(OutputPath)') And $(Configuration)=='Debug'">
    <Message Text="$(OutputPath)"></Message>
  </Target>
```

同时判断存在输出的文件夹并且在测试下才运行

注意不能使用引号加上 And 如`'And'`，这时 `And` 会作为字符串

如果使用多个条件，建议使用`()`包括多个条件，如下面代码，同时进行多个判断

```csharp
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp2.0</TargetFramework>
    <Foo>123</Foo>
  </PropertyGroup>

  <Target Name="StanalurJikecair" AfterTargets="CoreCompile" Condition="(Exists('$(OutputPath)') And $(Configuration)=='Debug') or ($(Foo)=='123')">
    <Message Text="$(OutputPath)"></Message>
  </Target>
```

## 使用的范围

在很多地方都可以使用条件进行判断，如放在任意的`PropertyGroup`里，如果判断为 false 就不会定义这个属性

```csharp
  <PropertyGroup>
    <OutputType Condition="'德熙' == '逗比'">Exe</OutputType>
    <TargetFramework>netcoreapp2.0</TargetFramework>
  </PropertyGroup>
```

也可以直接放在`PropertyGroup`让整个`PropertyGroup`不定义

```csharp
  <PropertyGroup Condition="'德熙' == '逗比'">
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp2.0</TargetFramework>
  </PropertyGroup>
```

同样可以放在 `ItemGroup` 里，也可以放在 `ItemGroup` 控制是否定义

```csharp
  <ItemGroup>
    <Foo Condition="'德熙' == '逗比'" Include="123"></Foo>
  </ItemGroup>
```

```csharp
  <ItemGroup Condition="'德熙' == '逗比'">
    <Foo Include="123"></Foo>
  </ItemGroup>
```

还有上面写的 `Target` 也可以使用，在  `Target` 里也可以使用条件

```csharp
  <Target Name="StanalurJikecair" AfterTargets="CoreCompile">
    <Message Condition="'德熙' == '逗比'" Text="$(OutputPath)"></Message>
  </Target>
```

还可以写在 `Import` 一般在 `Import` 都需要先判断是否存在文件

```csharp
  <Import Condition="Exists('../lindexi.txt')" Project="../lindexi.txt"></Import>

```

特别感谢 [HaibaraAi](https://github.com/AiHaibara) 大佬的阅读

![](https://i.loli.net/2018/08/04/5b64febec0b8a.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
