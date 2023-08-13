# Fiddler 插件开发 将插件放在独立子文件夹防止 DLL 冲突

我的 Fiddler 安装了许多插件，有一些插件存在 DLL 名冲突问题，比如多个不同的插件都存在名为 PluginCore.dll 但实际实现逻辑完全不相同的程序集。这就导致了多个插件的安装之间，如果没有将其放入到单独的文件夹内，将会因为文件名相同而冲突，让插件不能同时都安装。本文将和大家介绍 Fiddler 官方提供的将插件放在独立子文件夹的方法，用来解决 DLL 命名冲突

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

在 Fiddler 里，安装自定义插件给到 Fiddler 的最通用的方法就是将插件 DLL 和插件的依赖程序集拷贝到 `%USERPROFILE%\Documents\Fiddler2\Scripts` 文件夹里面，也就是 `我的文档\Fiddler2\Scripts` 文件夹里面

直接拷贝 DLL 到 `我的文档\Fiddler2\Scripts` 文件夹里面将可能遇到本文开始提到的 DLL 名冲突的问题。解决方法就是将插件放入到 Scripts 文件夹里面的里层文件夹内。放入到里层文件夹内需要遵循以下的规则才会被 Fiddler 当成插件加载

- 子文件夹必须是采用 `.ext` 结尾
- 放入到子文件夹的插件需要使用 `Fiddler` 开头，满足 `Fiddler*.dll` 通配符

接下来是详细的介绍，我所在的团队开发了一个名为 Friday 的插件，这个插件就是周五了，差不多快周六了的意思。我将使用这个插件和大家介绍如何将这个插件放入到 Fiddler 的 Scripts 文件夹里面的里层文件夹内

先将插件程序集的命名修改满足 `Fiddler*.dll` 规则，如将 Firday 插件命名为 FiddlerFridayPlugin 程序集。修改程序集名，可以右击项目属性，修改程序集。也可以编辑 csproj 项目文件，修改 AssemblyName 属性，如以下代码

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <!-- 根据 Fiddler 插件的要求，放入到子文件夹里面的插件，必须采用 Fiddler 开头，符合 Fiddler*.dll 才能被加载。因此这里修改程序集名，让输出程序集满足要求 -->
    <AssemblyName>FiddlerFridayPlugin</AssemblyName>
  </PropertyGroup>
</Project>
```

接着在 `我的文档\Fiddler2\Scripts` 文件夹里面新建一个子文件夹，要求这个子文件夹使用 `.ext` 结尾，比如 `FiddlerFridayPlugin.ext` 类似的命名格式

将构建输出的插件 DLL 等文件拷贝到上一步创建的子文件夹，完成之后的文件夹大概如下

```plain
C:\Users\YourName\Documents\Fiddler2
│
└─Scripts
   │
   └─FiddlerFridayPlugin.ext
          ├FiddlerFridayPlugin.dll
          ├FiddlerFridayPlugin.pdb
          ├HandyControl.dll
          ├Jgrass.FiddlerPlugin.dll
          └Newtonsoft.Json.dll
```

如此即可将插件放在独立的文件夹里面，防止和其他插件命名冲突

在日常开发中，如果想要让开发更加方便，可以编辑 csproj 项目文件，让插件构建输出到子文件夹里面，以下是项目文件代码

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Library</OutputType>
    <TargetFramework>net48</TargetFramework>
    <Nullable>enable</Nullable>
    <UseWPF>true</UseWPF>
    <LangVersion>latest</LangVersion>
    <!-- 根据 Fiddler 插件的要求，放入到子文件夹里面的插件，必须采用 Fiddler 开头，符合 Fiddler*.dll 才能被加载。因此这里修改程序集名，让输出程序集满足要求 -->
    <AssemblyName>FiddlerFridayPlugin</AssemblyName>
    <!-- 不要添加 net48 文件夹 -->
    <AppendTargetFrameworkToOutputPath>false</AppendTargetFrameworkToOutputPath>
    <!-- 根据 Fiddler 插件的要求，放入到子文件夹里面的插件，必须放入到后缀为 .ext 文件夹里面 -->
    <OutputPath>..\Bin\$(Configuration)\$(AssemblyName).ext\</OutputPath>
  </PropertyGroup>

  <ItemGroup>
    <Reference Include="Fiddler">
      <HintPath>你的Fiddler文件夹\Fiddler.exe</HintPath>
      <Private>false</Private>
    </Reference>
    <Reference Include="System.Web" />
  </ItemGroup>

  <!--<Target Name="KillFiddler" BeforeTargets="PreBuildEvent">
    <Exec Command="taskkill /F /IM Fiddler.exe" />
  </Target>-->

  <Target Name="CopyOutput" AfterTargets="PostBuildEvent">
    <Exec Command="xcopy ..\\Bin\\Debug %USERPROFILE%\\Documents\\Fiddler2\\Scripts  /s /e /y" />
  </Target>

</Project>
```

大家可以拷贝以上的 csproj 项目格式文件替换自己插件的 csproj 项目文件内容，记得更改 AssemblyName 为你期望的插件名即可，记得替换的时候保持使用 Fiddler 开头

另外还需要替换 `你的Fiddler文件夹\Fiddler.exe` 为你真正的 Fiddler 安装路径，如此才能让项目对 Fiddler 进行引用

使用以上的 csproj 项目格式制作的插件，可以在构建完成之后，自动给 Fiddler 安装上

细心的伙伴还可以看到注释掉的 KillFiddler 代码，这是因为插件的安装是需要重启 Fiddler 的，而且如果 Fiddler 正在运行过程中，旧版本插件还被引用，需要杀掉 Fiddler 才能覆盖插件文件。根据大家自己的需求，可以自己去掉注释，这样就可以在构建过程自动杀掉 Fiddler 应用