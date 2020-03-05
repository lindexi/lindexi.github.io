# Roslyn 开发 NuGet 包的 Task 编译可能遇到的问题

在写 msbuild 脚本的时候，或修改项目文件的时候，将会使用到很多的微软提供的 Task 命令。在需要复杂的编译的时候，可以通过自己定义一个任务用来定义编译

<!--more-->
<!-- CreateTime:2019/7/2 10:43:28 -->

<!-- csdn -->

如何入门开发请看 [如何创建一个基于 MSBuild Task 的跨平台的 NuGet 工具包 - walterlv](https://blog.walterlv.com/post/create-a-cross-platform-msbuild-task-based-nuget-tool.html ) 本文只会补充一些开发的过程会遇到的坑

## 版本选择

开发的 Task 需要引用以下两个库

- Microsoft.Build.Framework
- Microsoft.Build.Utilities.Core

这也就默认要求使用 .NET Framework 4.7.2 和 .NET Standard 2.0 以上

## 判断当前编译器平台

在跨平台的开发可能用户使用的是 dotnet core 版本，于是需要一个 dotnet core 版本的 Task 让他进行编译

使用下面代码可以判断当前点编译器是运行在哪个平台

```
'$(MSBuildRuntimeType)' == 'Core'
```

如果上面代码返回 true 那么编译器是 dotnet core 平台

以上面代码判断可以写出对使用 dotnet core 和 .NET Framework 两个平台的不同的程序集文件

## 调用方法

假定在程序集 UsingMSBuildCopyOutputFileToFastDebug.dll 里面包含 UsingMSBuildCopyOutputFileToFastDebug.SafeOutputFileCopyTask 类，这个类继承 Task 类

而 UsingMSBuildCopyOutputFileToFastDebug.dll 存放在 NuGet 包的 AssemblyFile 属性下，这个属性的定义请看代码

```xml
    <PropertyGroup>
        <!-- 我们使用 $(MSBuildRuntimeType) 来判断编译器是 .NET Core 的还是 .NET Framework 的。
         然后选用对应的文件夹。-->
        <NuGetUsingMSBuildCopyOutputFileToFastDebugTaskFolder Condition=" '$(MSBuildRuntimeType)' == 'Core'">$(MSBuildThisFileDirectory)..\tools\netcoreapp2.2\</NuGetUsingMSBuildCopyOutputFileToFastDebugTaskFolder>
        <NuGetUsingMSBuildCopyOutputFileToFastDebugTaskFolder Condition=" '$(MSBuildRuntimeType)' != 'Core'">$(MSBuildThisFileDirectory)..\tools\net48\</NuGetUsingMSBuildCopyOutputFileToFastDebugTaskFolder>
        <AssemblyFile>$(NuGetUsingMSBuildCopyOutputFileToFastDebugTaskFolder)\UsingMSBuildCopyOutputFileToFastDebug.dll</AssemblyFile>
    </PropertyGroup>
```

在调用之前需要先引用

```csharp
 <UsingTask TaskName="UsingMSBuildCopyOutputFileToFastDebug.SafeOutputFileCopyTask" 
               AssemblyFile="$(AssemblyFile)" />
```

之后可以在 Target 里面使用 SafeOutputFileCopyTask 类名

```csharp
    <Target Name="CopyOutputLibToFastDebug" AfterTargets="AfterBuild"
            Condition="$(MainProjectPath)!=''">
        <ItemGroup>
            <OutputFileToCopy Include="$(OutputPath)$(AssemblyName).dll"></OutputFileToCopy>
            <OutputFileToCopy Include="$(OutputPath)$(AssemblyName).pdb"></OutputFileToCopy>
        </ItemGroup>
        <SafeOutputFileCopyTask SourceFiles="@(OutputFileToCopy)" DestinationFolder="$(MainProjectPath)"></SafeOutputFileCopyTask>
    </Target>
```

## 如何添加属性

在继承 Microsoft.Build.Utilities.Task 的类里面添加属性就可以在直接使用，如我添加了 DestinationFolder 和 SourceFiles 属性

```csharp
    public class SafeOutputFileCopyTask : Microsoft.Build.Utilities.Task
    {
        public string[] SourceFiles { set; get; }
        public string DestinationFolder { set; get; }

        public override bool Execute()
        {
        	return true;
        }
    }

```

从上面代码可以看到，在 Task 里面添加列表数组的方法使用的是数组，如果使用的是列表那么在编译时将会提示

```csharp
C:\Users\lindexi.github.io\.nuget\packages\dotnetcampus.usingmsbuildcopyoutputfiletofastdebug\1.1.352\Build\dotnetCampus.UsingMSBuildCopyOutputFileToFastDebug.targets(18,33): error MSB4069: MSBuild 不支持“SafeOutputFileCopyTask”任务的“SourceFiles”参数的“System.Collections.Generic.List`1[[System.String, System.Private.CoreLib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=7cec85d7bea7798e]]”类型。
         C:\Users\lindexi.github.io\.nuget\packages\dotnetcampus.usingmsbuildcopyoutputfiletofastdebug\1.1.352\Build\dotnetCampus.UsingMSBuildCopyOutputFileToFastDebug.targets(18,33): error MSB4026: “SafeOutputFileCopyTask”任务的“SourceFiles=@(OutputFileToCopy)”参数无效。 [f:\temp\HogalcageBeganeqeale\HogalcageBeganeqeale.csproj]
         C:\Users\lindexi.github.io\.nuget\packages\dotnetcampus.usingmsbuildcopyoutputfiletofastdebug\1.1.352\Build\dotnetCampus.UsingMSBuildCopyOutputFileToFastDebug.targets(18,9): error MSB4063: 未能使用“SafeOutputFileCopyTask”任务的输入参数初始化该任务。
```

## 输出消息

输出消息请使用 Console.WriteLine 就可以输出

输出警告和错误也使用 Console.WriteLine 方法，只是需要添加前缀 `warning` 如下面代码

```csharp
                Console.WriteLine("warning: 用户没有传入需要复制的文件");
```

更多请看 [如何在 MSBuild Target（Exec）中报告编译错误和编译警告 - walterlv](https://blog.walterlv.com/post/standard-error-warning-format.html )

本文用到的代码开源在 Github 欢迎关注 [UsingMSBuildCopyOutputFileToFastDebug](https://github.com/dotnet-campus/UsingMSBuildCopyOutputFileToFastDebug ) 如有问题欢迎讨论

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
