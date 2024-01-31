Packaging.DebUOS 是我所在的团队开发开源的一款专门用在为 dotnet 的应用制作成为符合要求的 UOS 统信系统软件安装包的工具，此工具可以辅助开发者使用现有的工具链经过简单的配置即可完成安装包的制作

<!--more-->


<!-- CreateTime:2024/1/29 8:58:31 -->

<!-- 发布 -->
<!-- 博客 -->

## 设计思想

Packaging.DebUOS 旨在通过使用 csproj 项目文件等方式进行配置，避免直接处理deb包的打包细节，从而降低学习成本。它内置了大量默认属性，可以直接开箱即用而无需进行大量配置。此外，它依赖于 dotnet 构建命令，能与现有工具链无缝集成，方便接入团队旧有的CI/CD服务。除了提供简单易用的方法外，还提供了高级的命令行方法，以满足高定制 deb 包制作的需求。在高级命令行用法部分将对这些高级方法进行补充介绍

## 使用方法

第一步，安装 NuGet 库。通过 NuGet 管理器或采用如下代码编辑 csproj 项目文件安装 Packaging.DebUOS 库

```xml
  <ItemGroup>
    <PackageReference Include="Packaging.DebUOS" Version="3.11.0"/>
  </ItemGroup>
```

安装 Packaging.DebUOS 库不会影响到你的应用代码逻辑，仅仅只用来收集构建 UOS 应用安装包所需的配置信息，以及深度集成到 dotnet 构建命令里面

第二步，配置 UOS 的 AppId 属性值。如下面代码，编辑 csproj 项目文件，设置 AppId 属性。按照 UOS 的规范，请务必使用厂商的倒置域名+产品名作为应用包名，如 `com.example.demo` 格式，前半部分为厂商域名倒置，后半部分为产品名，只允许小写字母

```xml
  <PropertyGroup>
    <AppId>com.example.demo</AppId>
  </PropertyGroup>
```

具体更改可参阅 FileDownloader 项目的更改内容： [配置 ID 用来打包 - dotnet-campus/dotnetCampus.FileDownloader@2d5769b - GitHub](https://github.com/dotnet-campus/dotnetCampus.FileDownloader/commit/2d5769bd5d93a673bf1f7dfa70b0905057c35138)

第三步，执行命令行打包。在原有的 dotnet publish 命令里面，带上 `-t:CreateDebUOS` 参数，即可在正常发布完成之后，自动使用发布的输出文件制作成为符合要求的 UOS 统信系统软件安装包

```
dotnet publish -c release -r linux-x64 --self-contained true -t:CreateDebUOS
```

如此即可完成一个最为简单的符合要求的 UOS 统信系统软件安装包的制作。输出的安装包将会放在发布文件夹里面，可以将其拷贝到开发者的 UOS 系统上进行测试安装，预计正常都能安装成功

以上就是通过 Packaging.DebUOS 制作 UOS 统信系统软件安装包的最简单步骤。但相信对于大部分开发者来说，仅仅只是打出安装包还不能满足需求，开发者们还期望能够更好的配置安装包的更多信息，比如说安装到开始菜单的图标、开始菜单显示的应用名、配置安装包版本号等等，以下更多配置部分将会给出更多打包属性以满足开发者的更多定制需求

## 更多配置

以下列举出一些常用配置属性，更多的可配置属性请参阅 [DebUOSConfiguration.cs](https://github.com/dotnet-campus/dotnetcampus.DotNETBuildSDK/blob/85e73ae4d600bf0ba223d7c35ec34a15e7a4cc77/DebUOS/Packaging.DebUOS/Contexts/Configurations/DebUOSConfiguration.cs) 代码文件列举的属性

- UOSDebVersion : 安装包的版本号。默认将会取 Version 属性
- AppName : 应用名，英文名。默认取 AssemblyName 程序集名属性
- AppNameZhCN : 应用名，中文名，可不写。将在开始菜单中显示。如不写，将在开始菜单中显示英文的应用名
- DesktopComment : 放入到 desktop 里面的 Comment 属性，作为关于本程序的通用简述
- DesktopCommentZhCN : 放入到 desktop 里面的 Comment 属性，功能和 DesktopComment 相同，只是这是其中文版本
- DebControlDescription : 放入到 `DEBIAN\control` 文件的 Description 属性。如不填写，默认将使用 Description 属性的值。可在安装包双击时看到此描述内容
- Png32x32IconFile : 用于配置图标，配置完成的图标可在开始菜单作为应用程序图标。对应的还有 Png16x16IconFile Png24x24IconFile 等等不同尺寸的属性配置，以及矢量图 svg 的 SvgIconFile 属性配置

以下是对这些属性进行配置的 csproj 代码文件示例

```xml
  <PropertyGroup>
    <!-- 打出来的 UOS 包的 AppId 和版本号 -->
    <!-- 其中 AppId 是应用的唯一标识。按照 UOS 的规范，请务必使用厂商的倒置域名+产品名作为应用包名，如 `com.example.demo` 格式，前半部分为厂商域名倒置，后半部分为产品名，只允许小写字母 -->
    <AppId>com.dotnetcampus.unofiledownloader</AppId>
    <UOSDebVersion>1.0.1</UOSDebVersion>
    <!-- 应用名，英文名 -->
    <AppName>UnoFileDownloader</AppName>
    <!-- 应用名，中文名，可不写。将在开始菜单中显示 -->
    <AppNameZhCN>下载器</AppNameZhCN>
    <Png32x32IconFile>Download32x32Icon.png</Png32x32IconFile>
    <DesktopComment>The file downloader.</DesktopComment>
    <DesktopCommentZhCN>文件下载器，代码完全开源</DesktopCommentZhCN>
    <DebControlDescription>The UNO file downloader.</DebControlDescription>
    <DebControlHomepage>https://github.com/dotnet-campus/dotnetCampus.FileDownloader</DebControlHomepage>
    <DebControlMaintainer>dotnet-campus</DebControlMaintainer>
  </PropertyGroup>
```

以上的示例对应的真实项目代码变更请参阅： [加上更多打包属性 - dotnet-campus/dotnetCampus.FileDownloader@a834677 - GitHub](https://github.com/dotnet-campus/dotnetCampus.FileDownloader/commit/a83467712337b3fad303a30eb91126a4096f8dfd)

如期望自动在发布之后输出符合 UOS 规范的 deb 包，期望不需要在命令行添加 `-t:CreateDebUOS` 参数，则可以通过配置 `<AutoCreateDebUOSAfterPublish>true</AutoCreateDebUOSAfterPublish>` 属性到 csproj 从而实现在发布之后，自动执行打包，如以下代码

```xml
  <PropertyGroup>
    <AutoCreateDebUOSAfterPublish>true</AutoCreateDebUOSAfterPublish>
  </PropertyGroup>
```

如果仅通过属性配置依然无法达到预期的安装包制作要求，可以参考以下的高级命令行用法

## 高级命令行用法

命令行的使用方法是通过 dotnet tool 调用 Packaging.DebUOS.Tool 工具进行打包。通过 Packaging.DebUOS.Tool 工具可以作为 `dpkg-deb` 工具在 Windows 上的部分功能替代品

使用命令行工具比较适合创建构建更为复杂的 deb 安装包，可以有更强的定制化，适合对 UOS 安装包规范较熟悉的开发者使用

使用以下命令进行更新或安装工具：

```
dotnet tool update -g Packaging.DebUOS.Tool
```

将已经准备好的符合 UOS 安装包文件组织规范的文件夹打包为 deb 安装包：

```
dotnet dpkg-debuos -b C:\lindexi\DebPacking -o C:\lindexi\UOS\Foo.deb
```

以上的 `C:\lindexi\DebPacking` 为已准备好的符合 UOS 安装包文件组织规范的文件夹，以上的 `C:\lindexi\UOS\Foo.deb` 为打包输出的文件。其中 `-o` 指定打包输出文件参数可以忽略，如忽略此参数，则将会在打包文件夹输出 deb 安装包

通过命令行工具打包的方法，可以让开发者自行组织 UOS 安装包文件夹，可以让开发者有更自由更高级的定制

## 开源地址

此工具使用最友好的 MIT 协议在 GitHub 上完全开源，详细请看： <https://github.com/dotnet-campus/dotnetcampus.DotNETBuildSDK>

如有问题欢迎到 GitHub 上提 Issus 交流

## 参考文档

- [一步步教你在 Windows 上构建 dotnet 系应用的 UOS 软件安装包](https://blog.lindexi.com/post/%E4%B8%80%E6%AD%A5%E6%AD%A5%E6%95%99%E4%BD%A0%E5%9C%A8-Windows-%E4%B8%8A%E6%9E%84%E5%BB%BA-dotnet-%E7%B3%BB%E5%BA%94%E7%94%A8%E7%9A%84-UOS-%E8%BD%AF%E4%BB%B6%E5%AE%89%E8%A3%85%E5%8C%85.html )
- [应用打包规范 文档中心-统信UOS生态社区](https://doc.chinauos.com/content/M7kCi3QB_uwzIp6HyF5J )

## 感谢

- [https://github.com/quamotion/dotnet-packaging](https://github.com/quamotion/dotnet-packaging)
