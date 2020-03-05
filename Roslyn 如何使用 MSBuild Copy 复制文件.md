# Roslyn 如何使用 MSBuild Copy 复制文件

本文告诉大家如何在 MSBuild 里使用 Copy 复制文件

<!--more-->
<!-- CreateTime:2020/1/19 14:56:19 -->

<!-- 标签：Roslyn,MSBuild,编译器 -->

需要知道 Rosyln 是 MSBuild 的 dotnet core 版本。

在 MSBuild 里可以使用很多命令，本文告诉大家如何使用 Copy 这个 Task 来复制文件

在开始本文之前，希望大家已经知道了一些关于 csproj 文件格式，如果还是不知道，请看[理解 C# 项目 csproj 文件格式的本质和编译流程 - walterlv](https://walterlv.github.io/post/understand-the-csproj.html )

最简单的复制命令请看代码

```xml
  <Copy SourceFiles="lindexi.txt" DestinationFolder="LetirNuhe\" ></Copy>
```

需要注意，不要把 Copy 直接写在 Project 下，如下面的代码

```
<Project Sdk="Microsoft.NET.Sdk">
     <!-- 忽略代码 -->
   <Copy SourceFiles="lindexi.txt" DestinationFolder="LetirNuhe\" ></Copy>
</Project>

```

就会出现下面异常

```csharp
D:\林德熙\代码\测试代码\CemfeetoQewasXaiki\CemfeetoQewasXaiki.csproj : error  : 无法识别元素 <Project> 下面的元素 <Copy>。  D:\林德熙\代码\测试代码\CemfeetoQewasXaiki
```

为了运行 Copy 需要使用下面代码

```

  <Target Name="Copy" BeforeTargets="CoreCompile">
    <Copy SourceFiles="LekeexelSurgooHerkassayyayTowjome.txt" DestinationFolder="LetirNuhe\"></Copy>
  </Target>
```

需要知道 Target 需要给 Name 并且告诉他在什么时候运行，这里使用 BeforeTargets 告诉在开始编译前，也就是复制的文件会被编译。

对于复制资源文件或需要编译的资源，就设置 BeforeTargets 在编译前，如果是不需要进行编译的文件，如 dll 就可以设置在编译后运行。

重新生成项目，可以看到文件夹存在文件

![](http://image.acmx.xyz/lindexi%2F2018710103796940.jpg)

如果刚才没有创建 文件，复制时找不到文件，就会出现在重新编译出现无法编译

```csharp
error MSB3030: 无法复制文件“lindexi.txt”，原因是找不到该文件
```

复制有多个方式，下面让我来一个个和大家说

## 文件到文件

第一个方法是最简单的，复制文件到文件

例如我需要复制 lindiexi.txt 到 `LetirNuhe\lindexi` ，可以使用下面代码

```csharp
  <Target Name="Copy" BeforeTargets="CoreCompile">
    <Copy SourceFiles="lindexi.txt" DestinationFiles="LetirNuhe\lindexi.txt"></Copy>
  </Target>
```

那么如果需要复制多个文件到多个文件？

可以看到 SourceFiles 是可以输入多个文件，只需要使用`;`作为多个文件

下面复制 `lindexi.txt` 和 `lindexi.gitee.io.txt` 到 `LetirNuhe` 文件夹下

```
  <Target Name="Copy" BeforeTargets="CoreCompile">
    <Copy SourceFiles="lindexi.txt;lindexi.gitee.io.txt" DestinationFiles="LetirNuhe\lindexi.txt;LetirNuhe\lindexi.gitee.io.txt"></Copy>
  </Target>
```

这里的文件是对应的，也就是第一个文件是 `lindexi.txt`在 DestinationFiles 也需要写第一个文件是`lindexi.txt`的，如果写为`lindexi2.txt` 会自动把 `lindexi.txt` 复制并且修改名字。第一个文件对应 DestinationFiles 写的第一个文件，也就是项对应。

因为从文件复制到文件的代码太多了，如果只是需要把文件都放在相同的文件夹，可以使用下面的方法

## 文件到文件夹

如果需要把文件都复制到相同的文件夹，可以使用下面代码

```
  <Target Name="Copy" BeforeTargets="CoreCompile">
    <Copy SourceFiles="lindexi.txt;lindexi.gitee.io.txt" DestinationFolder="LetirNuhe\"></Copy>
  </Target>
```

使用 DestinationFolder 指定文件夹，在文件夹不存在的时候会自动创建，刚才的代码也是。

## 文件列表到文件夹

实际上刚才是写 SourceFiles ，但是实际这样写无法使用通配，也就是`*.txt`的方法，如果需要使用就需要用文件列表

```xml
  <ItemGroup>
    <Txt Include="*.txt"></Txt>
  </ItemGroup>

  <Target Name="Copy" BeforeTargets="CoreCompile">
    <Copy SourceFiles="@(Txt)" DestinationFolder="LetirNuhe\"></Copy>
  </Target>
```

多个文件的列表是在 ItemGroup 里添加 一个新的标签，这个标签是可以自己定义名字的，我这里定义了 Txt ，让他包含了 `*.txt` ，现在就可以在 SourceFiles 使用。使用数组的方法是 `@(Txt)` ，通过 @ 和 标签名就可以拿到标签的文件。如果这时输出`@(Txt)` 会看到下面代码

```csharp
xx\lindexi.txt;xx\lindexi.gitee.io.txt
```

因为 ItemGroup 可以写多个标签，可以修改下面代码

```xml
  <ItemGroup>
    <Txt Include="lindexi.txt"></Txt>
    <Txt Include="lindexi.gitee.io.txt"></Txt>
  </ItemGroup>

  <Target Name="Copy" BeforeTargets="CoreCompile">
    <Copy SourceFiles="@(Txt)" DestinationFolder="LetirNuhe\"></Copy>
  </Target>
```

## 较新才复制

如果不想每次编译都复制，可以设置`SkipUnchangedFiles="True"` 只有在发现文件较新才复制。

判断文件较新使用的是判断两个文件的最后更改时间和文件大小。

## 软连接

可以通过设置 `UseHardlinksIfPossible="True"`不复制文件，而是设置文件的软连接，也就是修改一个文件可以两个地方生效

设置软连接可以做到在多个项目看起来都有自己的文件，但是实际都是指向相同的文件

需要说的是，这个是软连接，但是在系统是硬连接方式。

## 判断文件存在就不复制

如果需要判断文件存在就不复制，可以使用 `Condition` 判断

```xml
    <Copy SourceFiles="@(Txt)" DestinationFolder="LetirNuhe\" SkipUnchangedFiles="True" OverwriteReadOnlyFiles="True" Condition="!Exists('LetirNuhe\lindexi.txt')"></Copy>
```

通过 `Exists` 判断文件是否存在，如果存在就不复制。

更多 MSBuild 相关博客请看

[理解 C# 项目 csproj 文件格式的本质和编译流程 - walterlv](https://walterlv.github.io/post/understand-the-csproj.html )

[如何创建一个基于命令行工具的跨平台的 NuGet 工具包 - walterlv](https://walterlv.github.io/post/create-a-cross-platform-command-based-nuget-tool.html )

[如何使用 MSBuild Target（Exec）中的控制台输出 - walterlv](https://walterlv.github.io/post/exec-task-of-msbuild-target.html )

更多关于 Roslyn 请看 [手把手教你写 Roslyn 修改编译](https://lindexi.oschina.io/lindexi/post/roslyn.html ) 

参见：[Roslyn 入门 - CSDN博客](https://blog.csdn.net/column/details/23159.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
