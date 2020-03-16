# WPF 讲讲 Microsoft.NET.Sdk.WindowsDesktop 的原理

在使用 .NET Core 3.0 和以上版本，可以使用新的 SDK 版本支持的 csproj 项目文件，这个 SDK 格式的项目文件可以支持 .NET Core 以及 .NET Framework 版本的 WPF 核心就是在 csproj 项目的第一句话。本文就来和大家聊这个 Microsoft.NET.Sdk.WindowsDesktop 的原理

<!--more-->
<!-- CreateTime:2020/3/16 8:31:10 -->

<!-- 发布 -->

现在 WPF 开源了，换句话说，遇到问题自己改代码也是可以的。或者说自己看代码也是很棒的

在新建的 SDK 格式的 csproj 项目文件里面，第一句话是引用 SDK 如下面代码

```csharp
<Project Sdk="Microsoft.NET.Sdk.WindowsDesktop">
```

请大杠不要说第一句话是 xml 和文件编码

通过 Microsoft.NET.Sdk.WindowsDesktop 就能从项目里面引入 SDK 支持构建 WPF 或 WinForms 应用

那么 Microsoft.NET.Sdk.WindowsDesktop 的源代码在哪？请点击 [github](https://github.com/dotnet/wpf/tree/165948b449e9de9fbba9843c2695f32a3212158f/packaging/Microsoft.NET.Sdk.WindowsDesktop) 打开，我特意用了一个commit的链接，因为可能后续文件路径会修改

这个 SDK 包含了两部分，一个是 Sdk 另一个是 targets 文件夹

打开 SDK 文件夹里面，可以 Sdk.props 和 Sdk.targets 两个文件，实际上这两个文件的实际作用就是被 VisualStudio 或 dotnet 的构建器发现，而核心逻辑都是通过 Import 引用 targets 文件夹里面的代码

```xml
<Project>

  <Import Sdk="Microsoft.NET.Sdk" Project="Sdk.targets" />
  
  <Import Project="$(MicrosoftWindowsDesktopSdkPath)\Microsoft.NET.Sdk.WindowsDesktop.targets "/>

</Project>
```

下面就来聊聊 targets 文件夹里面的内容，在这个文件夹里面包含了 xx.targets 和 xx.props 两个文件夹。按照编译器约定，其中的 props 文件用来定义属性，而 targets 文件用来定义执行逻辑

下面从 props 文件开始讲，在 Microsoft.NET.Sdk.WindowsDesktop.props 的功能分为以下部分

- 定义引用 App.xaml 作为应用定义
- 定义页面引用和构建方法
- 添加 FrameworkReference 框架依赖
- 引用框架需要的引用
- 移除不支持的框架版本

请让咱按照功能一步步看代码

第一个内容是初始化一些定义

```xml
  <PropertyGroup>
    <_MicrosoftNetSdkWindowsDesktop>true</_MicrosoftNetSdkWindowsDesktop>
    <EnableDefaultPageItems Condition="'$(EnableDefaultPageItems)' == ''">true</EnableDefaultPageItems> 
    <EnableDefaultApplicationDefinition Condition="'$(EnableDefaultApplicationDefinition)' == ''">true</EnableDefaultApplicationDefinition>
  </PropertyGroup>
```

默认定义了 EnableDefaultPageItems 和 EnableDefaultApplicationDefinition 分别表示将会自动添加 Page 文件和 App.xaml 文件，这两个只是属性，实际的逻辑在下面。这两个属性有一个细节就是 Condition 判断的代码，如果用户在自己的 csproj 文件里面更改了这个属性的值，那么这两个属性将不会在这里赋值，所以上面代码只是默认值

接下来就是定义 App.xaml 和 Page 的逻辑了，这个逻辑也就用到了上面定义的两个属性

```xml
  <ItemGroup Condition=" ('$(EnableDefaultItems)' == 'true') And ('$(UseWPF)' == 'true') And 
                         ('$(_TargetFrameworkVersionValue)' != '$(_UndefinedTargetFrameworkVersion)') And 
                         ('$(_TargetFrameworkVersionValue)' >= '$(_WindowsDesktopSdkTargetFrameworkVersionFloor)')">
    <ApplicationDefinition Include="App.xaml"
                           Condition="'$(EnableDefaultApplicationDefinition)' != 'false' And Exists('$(MSBuildProjectDirectory)/App.xaml') And '$(MSBuildProjectExtension)' == '.csproj'">
      <Generator>MSBuild:Compile</Generator>
    </ApplicationDefinition>
    <ApplicationDefinition Include="Application.xaml"
                           Condition="'$(EnableDefaultApplicationDefinition)' != 'false' And Exists('$(MSBuildProjectDirectory)/Application.xaml') And '$(MSBuildProjectExtension)' == '.vbproj'">
      <Generator>MSBuild:Compile</Generator>
    </ApplicationDefinition>

    <Page Include="**/*.xaml"
          Exclude="$(DefaultItemExcludes);$(DefaultExcludesInProjectFolder);@(ApplicationDefinition)"
          Condition="'$(EnableDefaultPageItems)' != 'false'" >
      <Generator>MSBuild:Compile</Generator>
    </Page>

    <None Remove="**/*.xaml"
          Condition="'$(EnableDefaultApplicationDefinition)' != 'false' And '$(EnableDefaultPageItems)' != 'false'" />
  </ItemGroup>
```

第一句话判断 Condition 的核心是 `('$(EnableDefaultItems)' == 'true') And ('$(UseWPF)' == 'true') ` 也就是是否加上默认的值，和采用 WPF 框架

如果采用 WPF 框架，那么将会初始化 ApplicationDefinition 的值，在 `ApplicationDefinition Include="App.xaml"` 就引用了 App.xaml 文件作为 ApplicationDefinition 也就是应用的定义。这个文件的引用有以下要求

- 存在这个 App.xaml 文件夹
- 用户没有设置 EnableDefaultApplicationDefinition 为false也就是不添加默认的文件
- 这个项目是 csproj 格式的，也就是 C# 代码的

而下面一行 `ApplicationDefinition Include="Application.xaml"` 和上面的逻辑相同，除了使用的是 Application.xaml 以及要求 vbproj 之外

接下来就是引用 Page 默认页面了，默认页面里面 EnableDefaultPageItems 决定是否引用所有页面

下一步就是引用框架了，通过源代码注释可以了解到不同的框架版本需要引用不同的框架如下

```
    .NET 3.x:   PresentationCore, PresentationFramework, WindowsBase 
    
    .NET 4.0:   PresentationCore, PresentationFramework, WindowsBase, System.Xaml, 
                UIAutomationClient, UIAutomationClientSideProviders, UIAutomationProvider, UIAutomationTypes
                
    .NET 4.5+:  PresentationCore, PresentationFramework, WindowsBase, System.Xaml, 
                UIAutomationClient, UIAutomationClientSideProviders, UIAutomationProvider, UIAutomationTypes
                System.Windows.Controls.Ribbon
```

这就是整个 props 文件的功能

在 targets 文件里面主要是用来更改构建步骤的，功能如下

- 移除重复的页面引用
- 提示构建项重复
- 提示 SDK 版本警告

移除重复的页面引用，需要移除页面里面引用的 `@(Resource);@(Content)` 的内容，大多数的资源和内容都不是 .xaml 格式的，而有一部分是声明为 Content 的 xaml 这些就是需要移除的。接下来就是移除 ApplicationDefinition 的内容，这个 xaml 是用来应用定义，如果没有移除 PresentationBuildTasks 将会构建这个文件两次，引用这个 xaml 的自动生成代码两次

接下来是一个 Target 用来提示构建项重复，核心是用到 CheckForDuplicateItems 这个 Task 做的逻辑，关于 Task 的使用和定义请看 [如何创建一个基于 MSBuild Task 的跨平台的 NuGet 工具包 - walterlv](https://blog.walterlv.com/post/create-a-cross-platform-msbuild-task-based-nuget-tool.html )

最后一部分就是 NetSdkWarning 用来提示版本警告

更多请看 [理解 C# 项目 csproj 文件格式的本质和编译流程 - walterlv](https://blog.walterlv.com/post/understand-the-csproj.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
