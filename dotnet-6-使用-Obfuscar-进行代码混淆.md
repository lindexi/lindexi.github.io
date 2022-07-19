
# dotnet 6 使用 Obfuscar 进行代码混淆

本文来安利大家 Obfuscar 这个好用的基于 MIT 协议开源的混淆工具。这是一个非常老牌的混淆工具，从 2014 年就对外分发，如今已有累计 495.5K 的 nuget 下载量。而且此工具也在不断持续迭代更新，完全支持 dotnet 6 版本，对 WPF 和 WinForms 等等的支持也是非常好，支持多个不同混淆方式和等级的配置，支持混淆之后生成符号文件。本文将来告诉大家如何使用此混淆工具，以及此工具能达成的效果和此工具混淆的原理

<!--more-->


<!-- CreateTime:2022/4/3 16:57:58 -->

<!-- 标签：dotnet，混淆 -->
<!-- 发布 -->

## 开源

此工具是由 [Lex Li](https://github.com/lextm) 主导开发的，在 GitHub 上使用 MIT 最友好协议开源，开源地址是 [https://github.com/obfuscar/obfuscar](https://github.com/obfuscar/obfuscar)

## 使用方法

此工具的使用方式有多个不同的方式，比较推荐的是采用 dotnet tool 的方式进行使用。因为使用 dotnet tool 可以非常方便接入自己已有的 CI CD 上，而且也可以实现非常方便的更新逻辑

按照约定，使用 dotnet tool 的第一步是进行安装，安装的方法就是在命令行输入以下代码

```
dotnet tool install --global Obfuscar.GlobalTool
```

如此即可完成安装。安装完成之后的使用方法是调用 `obfuscar.console` 命令，传入混淆配置 Obfuscar.xml 文件即可，如下面命令行例子

```
obfuscar.console Obfuscar.xml
```

也就是说最关键的只有两点，第一个就是命令行的工作路径，另一个就是混淆配置 Obfuscar.xml 文件

假定我有一个项目是 HeenerholiCeleehano 项目，此项目我需要对他进行混淆。此项目输出的是 HeenerholiCeleehano.dll 文件，放在 `C:\lindexi\Code\lindexi\HeenerholiCeleehano\HeenerholiCeleehano\bin\Release\net6.0-windows\` 文件夹下

那么在混淆之前，请设置好命令行的工作路径，如使用 `cd` 命令进入到输出文件夹，如以下命令

```
cd C\lindexi\Code\lindexi\HeenerholiCeleehano\HeenerholiCeleehano\bin\Release\net6.0-windows\
```

接下来是要在 `C:\lindexi\Code\lindexi\HeenerholiCeleehano\HeenerholiCeleehano\bin\Release\net6.0-windows\` 文件夹下放一个混淆配置 Obfuscar.xml 文件，文件内容可以是如下

```xml
<?xml version='1.0'?>
<Obfuscator>
  <Var name="InPath" value="." />
  <Var name="OutPath" value=".\Obfuscar" />
  <Var name="KeepPublicApi" value="true" />
  <Var name="HidePrivateApi" value="true" />
  <Var name="HideStrings" value="false" />
  <Var name="UseUnicodeNames" value="true" />
  <Var name="ReuseNames" value="true" />
  <Var name="RenameFields" value="true" />
  <Var name="RegenerateDebugInfo" value="true" />
  <Module file="$(InPath)\HeenerholiCeleehano.dll" />

  <AssemblySearchPath path="C:\Program Files\dotnet\shared\Microsoft.WindowsDesktop.App\6.0.1\" />
  <AssemblySearchPath path="C:\Program Files\dotnet\shared\Microsoft.NETCore.App\6.0.1\" />
</Obfuscator>
```

以上配置文件的含义如下，我一一带上注释标识


```xml
<?xml version='1.0'?>
<Obfuscator>
  <!-- 输入的工作路径，采用如约定的 Windows 下的路径表示法，如以下表示当前工作路径 -->
  <!-- 推荐使用当前工作路径，因为 DLL 的混淆过程，需要找到 DLL 的所有依赖。刚好当前工作路径下，基本都能满足条件 -->
  <Var name="InPath" value="." />
  <!-- 混淆之后的输出路径，如下面代码，设置为当前工作路径下的 Obfuscar 文件夹 -->
  <!-- 混淆完成之后的新 DLL 将会存放在此文件夹里 -->
  <Var name="OutPath" value=".\Obfuscar" />
  <!-- 以下的都是细节的配置，配置如何进行混淆 -->

  <!-- 使用 KeepPublicApi 配置是否保持公开的 API 不进行混淆签名，如公开的类型公开的方法等等，就不进行混淆签名了 -->
  <!-- 语法的写法就是 name 表示某个开关，而 value 表示值 -->
  <!-- 对于大部分的库来说，设置公开的 API 不进行混淆是符合预期的 -->
  <Var name="KeepPublicApi" value="true" />
  <!-- 设置 HidePrivateApi 为 true 表示，对于私有的 API 进行隐藏，隐藏也就是混淆的意思 -->
  <!-- 可以通过后续的配置，设置混淆的方式，例如使用 ABC 字符替换，或者使用不可见的 Unicode 代替 -->
  <Var name="HidePrivateApi" value="true" />
  <!-- 设置 HideStrings 为 true 可以设置是否将使用的字符串进行二次编码 -->
  <!-- 由于进行二次编码，将会稍微伤一点点性能，二次编码需要在运行的时候，调用 Encoding 进行转换为字符串 -->
  <Var name="HideStrings" value="false" />
  <!-- 设置 UseUnicodeNames 为 true 表示使用不可见的 Unicode 字符代替原有的命名，通过此配置，可以让反编译看到的类和命名空间和成员等内容都是不可见的字符 -->
  <Var name="UseUnicodeNames" value="true" />
  <!-- 是否复用命名，设置为 true 的时候，将会复用命名，如在不同的类型里面，对字段进行混淆，那么不同的类型的字段可以是重名的 -->
  <!-- 设置为 false 的时候，全局将不会有重复的命名 -->
  <Var name="ReuseNames" value="true" />
  <!-- 配置是否需要重命名字段，默认配置了 HidePrivateApi 为 true 将都会打开重命名字段，因此这个配置的存在只是用来配置为 false 表示不要重命名字段 -->
  <Var name="RenameFields" value="true" />
  <!-- 是否需要重新生成调试信息，生成 PDB 符号文件 -->
  <Var name="RegenerateDebugInfo" value="true" />

  <!-- 需要进行混淆的程序集，可以传入很多个，如传入一排排 -->
  <!-- <Module file="$(InPath)\Lib1.dll" /> -->
  <!-- <Module file="$(InPath)\Lib2.dll" /> -->
  <Module file="$(InPath)\HeenerholiCeleehano.dll" />

  <!-- 程序集的引用加载路径，对于 dotnet 6 应用，特别是 WPF 或 WinForms 项目，是需要特别指定引用加载路径的 -->
  <!-- 这里有一个小的需要敲黑板的知识点，应该让 Microsoft.WindowsDesktop.App 放在 Microsoft.NETCore.App 之前 -->
  <!-- 对于部分项目，如果没有找到如下顺序，将会在混淆过程中，将某些程序集解析为旧版本，从而失败 -->
  <AssemblySearchPath path="C:\Program Files\dotnet\shared\Microsoft.WindowsDesktop.App\6.0.1\" />
  <AssemblySearchPath path="C:\Program Files\dotnet\shared\Microsoft.NETCore.App\6.0.1\" />
</Obfuscator>
```

详细的配置，还请参阅[官方文档](https://docs.obfuscar.com/getting-started/configuration.html)

执行如上的命令行进行混淆，即可实现命名混淆效果

![](http://image.acmx.xyz/lindexi%2F2022431548139699.jpg)

使用 dotPeek 反编译可以看到字段被混淆为如下内容

```csharp
    private readonly int \u00A0;
    private readonly short \u00A0;
```

这里需要复习一下 IL 的知识，在 IL 里面，是允许不同的字段重名的，只要字段的类型不相同即可。因为使用的时候，是用类型名加字段名的方式使用的。这也能让代码更加混淆

以上就是我推荐的使用 dotnet tool 的方式

除了使用 dotnet tool 的方式之外，还可以自己将混淆过程嵌入到构建过程里面，如此可以实现在开发阶段对混淆的结果进行调试。也就是开发时调试的 DLL 就是混淆过后的

使用构建过程的方式需要编辑一下 csproj 项目文件，先在项目文件使用下面代码安装 Obfuscar 库，代码如下

```xml
  <ItemGroup>
    <PackageReference Include="Obfuscar" Version="2.2.33">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
  </ItemGroup>
```

在需要写构建过程下，预计实际需求都是千奇百怪的，很难有统一的方式，本文只是提供一个简单的例子

完成安装之后，在项目上放一个混淆配置 Obfuscar.xml 文件，设置此文件如果较新则输出

```xml
  <ItemGroup>
    <None Update="Obfuscar.xml">
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </None>
  </ItemGroup>
```

最后编写一个 Target 用来在构建完成之后调用命令行进行混淆，代码如下

```xml
  <Target Name="ObfuscarTask" AfterTargets="AfterBuild">
    <PropertyGroup>
      <ObfuscateCommand>$(Obfuscar) "Obfuscar.xml"</ObfuscateCommand>
    </PropertyGroup>
    <Exec WorkingDirectory="$(OutputPath)" Command="$(ObfuscateCommand)" />
  </Target>
```

如此即可实现在构建完成之后，自动调用

以上的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/b0b402abe4f32008d383d984bff677ac45cccde8/HeenerholiCeleehano) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/b0b402abe4f32008d383d984bff677ac45cccde8/HeenerholiCeleehano) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin b0b402abe4f32008d383d984bff677ac45cccde8
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 HeenerholiCeleehano 文件夹

以上代码的混淆配置 Obfuscar.xml 文件里使用的程序集引用路径写的是 `C:\Program Files\dotnet\shared\Microsoft.WindowsDesktop.App\6.0.1\` 路径，这是因为代码是之前写的，博客是鸽子很久才写的，还请大家自行更新

## 混淆原理

此混淆工具底层使用 [Mono.Cecil](https://github.com/jbevain/cecil/) 进行程序集的读取和编织，使用 [Mono.Cecil](https://github.com/jbevain/cecil/) 可以读取出程序集的信息，从读取到的信息进行更改，更改也就是混淆的核心逻辑，更改完成之后，再通过 [Mono.Cecil](https://github.com/jbevain/cecil/) 生成新的程序集文件，如此即可完成混淆

此工具在 GitHub 上完全开源，请看 [https://github.com/obfuscar/obfuscar](https://github.com/obfuscar/obfuscar)

更多细节逻辑还请自己去阅读源代码

## 更多阅读

其他的混淆工具还有非常多，可以从 [https://github.com/NotPrab/.NET-Obfuscator](https://github.com/NotPrab/.NET-Obfuscator) 找到更多的混淆工具列表

当然，有混淆工具，也就有反混淆工具。可以从 [https://github.com/NotPrab/.NET-Deobfuscator](https://github.com/NotPrab/.NET-Deobfuscator) 找到更多反混淆列表





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。