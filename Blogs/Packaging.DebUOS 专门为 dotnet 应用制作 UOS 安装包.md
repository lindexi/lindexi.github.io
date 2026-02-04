---
title: Packaging.DebUOS 专门为 dotnet 应用制作 UOS 安装包
description: Packaging.DebUOS 是我所在的团队开发开源的一款专门用在为 dotnet 的应用制作成为符合要求的 UOS 统信系统软件安装包的工具，此工具可以辅助开发者使用现有的工具链经过简单的配置即可完成安装包的制作
tags: dotnet
category: 
---

<!-- CreateTime:2024/1/29 8:58:31 -->

<!-- 发布 -->
<!-- 博客 -->

## 设计思想

Packaging.DebUOS 旨在通过使用 csproj 项目文件等方式进行配置，避免直接处理deb包的打包细节，从而降低学习成本。它内置了大量默认属性，可以直接开箱即用而无需进行大量配置。此外，它依赖于 dotnet 构建命令，能与现有工具链无缝集成，方便接入团队旧有的CI/CD服务。除了提供简单易用的方法外，还提供了高级的命令行方法，以满足高定制 deb 包制作的需求。在高级命令行用法部分将对这些高级方法进行补充介绍

## 使用方法

第一步，安装 NuGet 库。通过 NuGet 管理器或采用如下代码编辑 csproj 项目文件安装 Packaging.DebUOS 库

```xml
  <ItemGroup>
    <PackageReference Include="Packaging.DebUOS" Version="3.17.10"/>
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

以下列举出一些常用配置属性，更多的可配置属性请参阅 [DebUOSConfiguration 配置大全](https://github.com/dotnet-campus/dotnetcampus.DotNETBuildSDK/blob/master/DebUOS/README.md) 文档列举的属性，或参考本文末尾的全部配置项章节

- UOSDebVersion : 安装包的版本号。可不写，默认将会取 Version 属性
- AppName : 应用名，英文名。可不写，默认取 AssemblyName 程序集名属性
- AppNameZhCN : 应用名，中文名，可不写。将在开始菜单中显示。如不写，将在开始菜单中显示英文的应用名
- DesktopComment : 放入到 desktop 里面的 Comment 属性，作为关于本程序的通用简述
- DesktopCommentZhCN : 放入到 desktop 里面的 Comment 属性，可不写，功能和 DesktopComment 相同，只是这是其中文版本
- DebControlDescription : 放入到 `DEBIAN\control` 文件的 Description 属性。可不写，如不填写，默认将使用 Description 属性的值。可在安装包双击时看到此描述内容
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

完成以上配置即可构建出一个比较完备的 deb 安装包了

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

将已经准备好的符合 UOS 安装包文件组织规范的文件夹（`C:\lindexi\DebPacking`）打包为 deb 安装包：

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
- [商店打包规范 - deepin开发者平台](https://docs.deepin.org/info/%E5%BC%80%E5%8F%91%E8%BF%9B%E9%98%B6/%E5%BA%94%E7%94%A8%E5%95%86%E5%BA%97/%E5%BA%94%E7%94%A8%E6%89%93%E5%8C%85%E8%A7%84%E8%8C%83/%E7%9B%B8%E5%85%B3%E8%A7%84%E8%8C%83/%E5%95%86%E5%BA%97%E6%89%93%E5%8C%85%E8%A7%84%E8%8C%83 )

## 感谢

- [https://github.com/quamotion/dotnet-packaging](https://github.com/quamotion/dotnet-packaging)

如使用过程有问题，欢迎加入国产应用开发群交流： 810052083

## 全部配置项

大部分配置都是可选项，以下仅仅作为示例参考使用

```xml
<!-- 自定义的 DEBIAN\control 文件路径，将直接使用该文件作为 control 文件，忽略其他配置。这是比较高级的配置，一般不需要使用，可以用来满足更多的定制化需求 -->
<DebControlFile>Assets\control</DebControlFile>

<!-- 自定义的 DEBIAN\postinst 文件路径，将直接使用该文件作为 postinst 文件，忽略其他配置。这是比较高级的配置，一般不需要使用，可以用来满足更多的定制化需求

 postinst：软件安装时执行的脚本

 按照 UOS 的规范，除对本程序根目录文件进行必要的操作修改外，禁止使用deb的postinst等钩子对系统文件进行修改，包含这些脚本 的软件包都无法上架 -->
<DebPostinstFile>Assets\PostInstall.sh</DebPostinstFile>

<!-- 自定义的 DEBIAN\prerm 文件路径，将直接使用该文件作为 prerm 文件，忽略其他配置。这是比较高级的配置，一般不需要使用，可以用来满足更多的定制化需求

 prerm：软件卸载前执行的脚本

 按照 UOS 的规范，除对本程序根目录文件进行必要的操作修改外，禁止使用deb的postinst等钩子对系统文件进行修改，包含这些脚本 的软件包都无法上架 -->
<DebPrermFile>Assets\PreRm.sh</DebPrermFile>

<!-- 自定义的 DEBIAN\postrm 文件路径，将直接使用该文件作为 postrm 文件，忽略其他配置。这是比较高级的配置，一般不需要使用，可以用来满足更多的定制化需求

 postrm：软件卸载后执行的脚本

 按照 UOS 的规范，除对本程序根目录文件进行必要的操作修改外，禁止使用deb的postinst等钩子对系统文件进行修改，包含这些脚本 的软件包都无法上架 -->
<DebPostrmFile>Assets\PostRm.sh</DebPostrmFile>

<!-- 自定义的 DEBIAN\preinst 文件路径，将直接使用该文件作为 preinst 文件，忽略其他配置。这是比较高级的配置，一般不需要使用，可以用来满足更多的定制化需求 -->
<DebPreinstFile>Assets\preinst.sh</DebPreinstFile>

<!-- 自定义的 opt\apps\${AppId}\info 文件路径，将直接使用该文件作为 info 文件，忽略其他配置。这是比较高级的配置，一般不 需要使用，可以用来满足更多的定制化需求 -->
<DebInfoFile>Assets\Info.json</DebInfoFile>

<!-- 自定义的 opt\apps\${AppId}\entries\applications\${AppId}.desktop 文件路径，将直接使用该文件作为 desktop 文件，忽略 其他配置。这是比较高级的配置，一般不需要使用，可以用来满足更多的定制化需求 -->
<DebDesktopFile>Assets\Demo.desktop</DebDesktopFile>

<!-- 应用的 AppId 值，用来组织应用的安装路径，同时也是应用的唯一标识。按照 UOS 的规范，请务必使用厂商的倒置域名+产品名作为应用包名，如 `com.example.demo` 格式，前半部分为厂商域名倒置，后半部分为产品名，如果使用非拥有者的域名作为前缀，可能会引起该域名拥有者进行申诉，导致软件被申诉下架或者删除，只允许小写字母。不写默认和 AssemblyName 属性相同 -->
<AppId>com.example.demo</AppId>

<!-- 应用的 AppId 值，用来组织应用的安装路径，同时也是应用的唯一标识。按照 UOS 的规范，请务必使用厂商的倒置域名+产品名作为应用包名，如 `com.example.demo` 格式，前半部分为厂商域名倒置，后半部分为产品名，如果使用非拥有者的域名作为前缀，可能会引起该域名拥有者进行申诉，导致软件被申诉下架或者删除，只允许小写字母。不写默认和 AppId 属性相同

 与 AppId 属性不同的是，该属性明确给制作 UOS 的包使用，不会和其他的逻辑的 AppId 混淆 -->
<UOSAppId>com.example.demo</UOSAppId>

<!-- 版本号，默认是 1.0.0 版本 -->
<Version>1.2.3</Version>

<!-- 专门给制作 UOS 的包使用的版本号，不写将使用 Version 属性的值。可使用 a.b.c 格式，也可以比较复杂的语义版本号格式，如 `1.2.3-2+b1` 等格式 -->
<UOSDebVersion>1.2.3</UOSDebVersion>

<!-- 配置放入到 DEBIAN\control 文件的 Section 属性，可以选用 utils，admin, devel, doc, libs, net, 或者 unknown 等等，代 表着该软件包在 Debian 仓库中将被归属到什么样的逻辑子分类中。默认是 utils 类型 -->
<DebControlSection>utils</DebControlSection>

<!-- 配置放入到 DEBIAN\control 文件的 Priority 属性，可以选用 required, important, standard, optional, extra 等等，代表 着该软件包在 Debian 仓库中的优先级，optional 优先级适用于与优先级为 required、important 或 standard 的软件包不冲突的新软件包。也可以做其它取值。若是不明了，请使用 optional。默认是 optional 类型 -->
<DebControlPriority>optional</DebControlPriority>

<!-- 配置放入到 DEBIAN\control 文件的 Architecture 属性，以及 opt\apps\${AppId}\info 文件的 arch 属性。可以选用 amd64, i386, arm64, armel, armhf, mips, mips64el, mipsel, ppc64el, s390x, 或者 all 等等，代表着该软件包在 Debian 仓库中的架构，amd64 代表着 64 位的 x86 架构，i386 代表着 32 位的 x86 架构，arm64 代表着 64 位的 ARM 架构，armel 代表着 32 位的 ARM 架构，armhf 代表着 32 位的 ARM 架构，mips 代表着 32 位的 MIPS 架构，mips64el 代表着 64 位的 MIPS 架构，mipsel 代表着 32 位的 MIPS 架构，ppc64el 代表着 64 位的 PowerPC 架构，s390x 代表着 64 位的 IBM S/390 架构，all 代表着所有架构。目前商店支持以下的 amd64, mips64el, arm64, sw_64, loongarch64 几种架构。默认将根据 RuntimeIdentifier 属性决定是 amd64 、arm64类型 -->
<Architecture>amd64</Architecture>

<!-- 配置放入到 DEBIAN\control 文件的 Multi-Arch 属性。默认是 foreign 类型 -->
<DebControlMultiArch>foreign</DebControlMultiArch>

<!-- 配置放入到 DEBIAN\control 文件的 Build-Depends 属性。默认是 debhelper (>=9) 类型 -->
<DebControlBuildDepends>debhelper (>=9)</DebControlBuildDepends>

<!-- 配置放入到 DEBIAN\control 文件的 Standards-Version 属性。默认是 3.9.6 的值 -->
<DebControlStandardsVersion>3.9.6</DebControlStandardsVersion>

<!-- 配置放入到 DEBIAN\control 文件的 Maintainer 属性。如不填写，默认将会按照 Authors Author Company Publisher 的顺序， 找到第一个不为空的值，作为 Maintainer 的值。如最终依然为空，可能导致打出来的安装包在用户端安装之后，不能在开始菜单中找到应用的图标 -->
<DebControlMaintainer>dotnet-campus</DebControlMaintainer>

<!-- 配置放入到 DEBIAN\control 文件的 Homepage 属性。如不填写，将尝试使用 PackageProjectUrl 属性，如依然为空则采用默认值。默认是 https://www.uniontech.com 的值 -->
<DebControlHomepage>https://github.com/dotnet-campus/dotnetcampus.DotNETBuildSDK</DebControlHomepage>

<!-- 配置放入到 DEBIAN\control 文件的 Description 属性。如不填写，默认将使用 Description 属性的值 -->
<DebControlDescription>The file downloader.</DebControlDescription>

<!-- 配置放入到 DEBIAN\control 文件的 Depends 属性。如不填写，则忽略。用于配置软件依赖，比如填写入 vlc,libvlc-dev 即可在声明安装包依赖 vlc 组件 -->
<DebControlDepends></DebControlDepends>

<!-- 应用名，英文名。将作为 opt\apps\${AppId}\entries\applications\${AppId}.desktop 和 opt\apps\${AppId}\info 的 Name 属性的值，不写默认和 AssemblyName 属性相同 -->
<AppName>UnoFileDownloader</AppName>

<!-- 应用名，中文名，可不写。将在开始菜单中显示 -->
<AppNameZhCN>下载器</AppNameZhCN>

<!-- 配置放入到 opt\apps\${AppId}\info 文件的 permissions 属性，可不写，可开启的属性之间使用分号 ; 分割。可选值有：autostart, notification, trayicon, clipboard, account, bluetooth, camera, audio_record, installed_apps 等。默认为不开启任何权限 -->
<InfoPermissions>autostart;notification;trayicon;clipboard;account</InfoPermissions>

<!-- 配置放入到 opt\apps\${AppId}\entries\applications\${AppId}.desktop 文件的 Categories 属性，可选值有：AudioVideo, Audio, Video, Development, Education, Game, Graphics, Network, Office, Science, Settings, System, Utility, Other 等。默认 为 Other 的值 -->
<DesktopCategories>Other</DesktopCategories>

<!-- 配置放入到 opt\apps\${AppId}\entries\applications\${AppId}.desktop 文件的 Keywords 属性，作为程序的通用关键搜索词，当在启动器中搜索该词而非程序名称时，即可索引出该程序的快捷方式。多个关键词之间使用分号 ; 分割，关键词使用英文。如需添加 中文关键词，请设置 DesktopKeywordsZhCN 属性。默认为 deepin 的值 -->
<DesktopKeywords>deepin;downloader</DesktopKeywords>

<!-- 配置放入到 opt\apps\${AppId}\entries\applications\${AppId}.desktop 文件的 Keywords[zh_CN] 属性，可不填，作为程序的 通用关键搜索词，当在启动器中搜索该词而非程序名称时，即可索引出该程序的快捷方式。多个关键词之间使用分号 ; 分割，关键词使 用中文 -->
<DesktopKeywordsZhCN>工具;下载器</DesktopKeywordsZhCN>

<!-- 配置放入到 opt\apps\${AppId}\entries\applications\${AppId}.desktop 文件的 Comment 属性，作为关于本程序的通用简述， 在没有单独设置语言参数的情况下，默认显示该段内容。不填将使用 UOSAppId 属性的值 -->
<DesktopComment>The file downloader.</DesktopComment>

<!-- 配置放入到 opt\apps\${AppId}\entries\applications\${AppId}.desktop 文件的 Comment[zh_CN] 属性，作为关于本程序的通用中文简述，可不填 -->
<DesktopCommentZhCN>这是一个下载器</DesktopCommentZhCN>

<!-- 配置放入到 opt\apps\${AppId}\entries\applications\${AppId}.desktop 文件的 NoDisplay 属性，如果设置为 true 则表示当 前的应用不放在开始菜单里面，即在开始菜单隐藏应用。一般用于一些不想让用户直接碰到的，直接运行的应用。可不填，默认是 false 的值 -->
<DesktopNoDisplay>false</DesktopNoDisplay>

<!-- 配置放入到 opt\apps\${AppId}\entries\applications\${AppId}.desktop 文件的 Exec 属性，作为程序的启动命令，可不填，且推荐不填，除非有特殊需求。默认为 /opt/apps/${AppId}/files/bin/${AssemblyName} 的值 -->
<DesktopExec>/opt/apps/$(AppId)/files/bin/$(AssemblyName)</DesktopExec>

<!-- 配置放入到 opt\apps\${AppId}\entries\applications\${AppId}.desktop 文件的 Icon 属性，作为程序的图标，可不填，且推荐不填，除非有特殊需求。默认为 UOSAppId 的值 -->
<DesktopIcon>$(UOSAppId)</DesktopIcon>

<!-- 配置放入到 opt\apps\${AppId}\entries\applications\${AppId}.desktop 文件的 Type 属性，作为程序的类型，按照 UOS 的规 范，必须为 Application 的值，推荐不更改，即不填 -->
<DesktopType>Application</DesktopType>

<!-- 配置放入到 opt\apps\${AppId}\entries\applications\${AppId}.desktop 文件的 Terminal 属性，用来决定程序是否以终端的形式运行，默认是 false 关闭状态 -->
<DesktopTerminal>false</DesktopTerminal>

<!-- 配置放入到 opt\apps\${AppId}\entries\applications\${AppId}.desktop 文件的 StartupNotify 属性，用来决定程序是否允许 桌面环境跟踪应用程序的启动，提供用户反馈和其他功能。例如鼠标的等待动画等，按照 UOS 规范建议，为保障应用使用体验，默认是 true 开启状态，推荐不更改，即不填 -->
<DesktopStartupNotify>true</DesktopStartupNotify>

<!-- 配置放入到 opt\apps\${AppId}\entries\applications\${AppId}.desktop 文件的 MimeType 属性，用来配置程序支持的关联文件类型，根据实际需求来填写。如果没有需要支持关联文件，则不填。多个文件类型之间使用分号 ; 分割 -->
<DesktopMimeType>audio/aac;application/aac;</DesktopMimeType>

<!-- 进行打包的文件夹，用来组织打包的文件。可不填，且推荐不填，将被打包工具自动填充 -->
<PackingFolder>obj\DebUOSPacking\Packing\</PackingFolder>

<!-- 工作文件夹，用来存放打包过程中的临时文件。可不填，且推荐不填，将被打包工具自动填充 -->
<WorkingFolder>obj\DebUOSPacking\</WorkingFolder>

<!-- 项目的发布输出文件夹。可不填，且推荐不填，将被打包工具自动填充 -->
<ProjectPublishFolder>$([MSBuild]::NormalizePath($(MSBuildProjectDirectory), $(PublishDir)))</ProjectPublishFolder>

<!-- 打包输出文件路径。可不填，默认将放在发布文件夹里 -->
<DebUOSOutputFilePath>bin\Foo.deb</DebUOSOutputFilePath>

<!-- 表示图标文件夹路径，文件夹里面按照 UOS 的 deb 规范组织图标文件，文件夹里面存放的内容将会被原原本本拷贝到 opt/apps/${AppId}/entries/icons/ 文件夹里面。此属性属于高级配置，一般不需要使用，可以用来满足更多的定制化需求。默认不填，且推荐在 充分理解 UOS 的 deb 规范的情况下再进行使用。此属性存在值时，将会忽略 SvgIconFile 和 Png16x16IconFile 等属性的设置 -->
<UOSDebIconFolder>Assets\Icons\</UOSDebIconFolder>

<!-- 应用图标文件，表示矢量图标文件。将被放入到 opt/apps/${AppId}/entries/icons/hicolor/scalable/apps/${appid}.svg 里面 。矢量图标文件与 png 非矢量格式二选一，如果同时存在，优先使用矢量图标文件。
 当 UOSDebIconFolder 属性存在值时，本属性设置无效 -->
<SvgIconFile>Assets\Icons\Logo.svg</SvgIconFile>

<!-- 应用图标文件，表示 png 非矢量格式文件。将被放入到 opt/apps/${AppId}/entries/icons/hicolor/${分辨率}/apps/${appid}.png 里面。请确保实际图片分辨率正确，且是 png 格式。矢量图标文件与 png 非矢量格式二选一，如果同时存在，优先使用矢量图标文 件。
 当 UOSDebIconFolder 属性存在值时，本属性设置无效 -->
<Png16x16IconFile>Assets\Icons\Logo16x16.png</Png16x16IconFile>

<!-- 打包时应该有哪些后缀被排除，默认包括 .pdb .dbg .md 文件
 如果有其他特殊规则，请自行编写 Target 在 CreateDebUOS 之前删除掉 -->
<ExcludePackingDebFileExtensions>.pdb;.dbg;.md</ExcludePackingDebFileExtensions>
```

## FAQ

### 如何在 deb 包里面添加符号 pdb 文件

添加 ExcludePackingDebFileExtensions 属性配置，重新指定打包时应该有哪些后缀被排除。因为默认的 ExcludePackingDebFileExtensions 属性包含了 .pdb .dbg .md 文件，因此符号 pdb 文件将被排除。使用如下代码重设 ExcludePackingDebFileExtensions 属性，减少对 `.pdb` 文件的排除，自然就可以在 deb 包里面打进符号 pdb 文件

```xml
    <PropertyGroup>
      <ExcludePackingDebFileExtensions>.dbg;.md</ExcludePackingDebFileExtensions>
    </PropertyGroup>
```

### 如何添加更多文件加入 deb 打包里

一般情况下，能够输出到发布路径的，就能加入到 deb 包里面。比如在 csproj 配置某些文件如果较新则拷贝等

如果需要动态编写构建逻辑，则可在 Publish 之后，在 CreateDebUOS 之前，进行动态加入文件。如以下例子，添加的是构建信息 Version.txt 文件到打包的 deb 里面

```xml
  <Target Name="_BuildVersionInfoTarget" BeforeTargets="CreateDebUOS" DependsOnTargets="Publish">
    <PropertyGroup>
      <BuildVersionInfoFile>$([System.IO.Path]::Combine($(PublishDir), "Version.txt"))</BuildVersionInfoFile>
      <BuildTimeInfo>$([System.DateTimeOffset]::get_Now().ToString())</BuildTimeInfo>
    </PropertyGroup>
    <ItemGroup>
      <BuildVersionInfoWriteArgLine Include="&gt;" />
      <BuildVersionInfoWriteArgLine Include="GitCommit" />
      <BuildVersionInfoWriteArgLine Include="$(GitCommit)" />
      <BuildVersionInfoWriteArgLine Include="&gt;" />

      <BuildVersionInfoWriteArgLine Include="BuildTime" />
      <BuildVersionInfoWriteArgLine Include="$(BuildTimeInfo)" />
      <BuildVersionInfoWriteArgLine Include="&gt;" />
    </ItemGroup>

    <WriteLinesToFile File="$(BuildVersionInfoFile)" Lines="@(BuildVersionInfoWriteArgLine)" Overwrite="true" />
  </Target>

  <Target Name="_GitCommit" Returns="$(GitCommit)" BeforeTargets="_BuildVersionInfoTarget" Condition="'$(GitCommit)' == ''">
    <Exec Command="git rev-parse HEAD" EchoOff="true" StandardErrorImportance="low" StandardOutputImportance="low" ConsoleToMSBuild="true" ContinueOnError="true" StdOutEncoding="utf-8">
      <Output TaskParameter="ConsoleOutput" PropertyName="GitCommit" />
      <Output TaskParameter="ExitCode" PropertyName="MSBuildLastExitCode" />
    </Exec>
  </Target>
```

### 如何添加 vlc 依赖

在 PropertyGroup 里的 DebControlDepends 属性的添加 `vlc,libvlc-dev` 依赖，代码如下

```xml
  <PropertyGroup>
    <!-- 软件的依赖包
      vlc,libvlc-dev 依赖原因：https://code.videolan.org/videolan/LibVLCSharp/-/blob/3.x/docs/linux-setup.md -->
    <DebControlDepends>vlc,libvlc-dev</DebControlDepends>
  </PropertyGroup>
```

由于 LibVLCSharp 难以维护 Linux 复杂的 VLC 版本，因此软件带 VLC 的方式是推荐使用声明依赖的方式。声明依赖之后，将在安装 deb 安装包的时候要求依赖负载。如使用 dpkg 命令，则在缺失依赖时不给安装，且给出其依赖说明。如使用图形界面的安装器，比如麒麟系统的 kylin-installer 安装器，一般都会自动从软件包源安装依赖
