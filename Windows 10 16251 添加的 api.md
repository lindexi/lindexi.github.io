# Windows 10 16251 添加的 api

本文主要讲微软最新的sdk添加的功能，暂时还不能下载，到 7月29 ，现在可以下载是 16232 ，支持Neon效果。

实际上设置软件最低版本为 16232 就自动支持 Neon 效果。

主要添加了 AppDataPaths SystemDataPaths 等

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


Windows 

 - Storage 

  - AppDataPaths 
  - Cookies { get; } : String
  - Desktop { get; } : String
  - Documents { get; } : String
  - Favorites { get; } : String
  - GetDefault() static : AppDataPaths
  - GetForUser(User user) static : AppDataPaths
  - History { get; } : String
  - InternetCache { get; } : String
  - LocalAppData { get; } : String
  - ProgramData { get; } : String
  - RoamingAppData { get; } : String

 - SystemDataPaths 
  - Fonts { get; } : String
  - GetDefault() static : SystemDataPaths
  - ProgramData { get; } : String
  - Public { get; } : String
  - PublicDesktop { get; } : String
  - PublicDocuments { get; } : String
  - PublicDownloads { get; } : String
  - PublicMusic { get; } : String
  - PublicPictures { get; } : String
  - PublicVideos { get; } : String
  - System { get; } : String
  - SystemArm { get; } : String
  - SystemHost { get; } : String
  - SystemX64 { get; } : String
  - SystemX86 { get; } : String
  - UserProfiles { get; } : String
  - Windows { get; } : String

 - UserDataPaths 
  - CameraRoll { get; } : String
  - Cookies { get; } : String
  - Desktop { get; } : String
  - Documents { get; } : String
  - Downloads { get; } : String
  - Favorites { get; } : String
  - GetDefault() static : UserDataPaths
  - GetForUser(User user) static : UserDataPaths
  - History { get; } : String
  - InternetCache { get; } : String
  - LocalAppData { get; } : String
  - LocalAppDataLow { get; } : String
  - Music { get; } : String
  - Pictures { get; } : String
  - Profile { get; } : String
  - Recent { get; } : String
  - RoamingAppData { get; } : String
  - SavedPictures { get; } : String
  - Screenshots { get; } : String
  - Templates { get; } : String
  - Videos { get; } : String


[win10.16241.to.win10.16251.fulldiff](https://martinsuchan.github.io/ApiPeek/Diffs/win10.16241.to.win10.16251.fulldiff.html)


## 工具

介绍一下好用的工具

[HappyStudio.UwpToolsLibrary.Auxiliarys 1.0.3](https://www.nuget.org/packages/HappyStudio.UwpToolsLibrary.Auxiliarys/) UWP工具库的辅助类库

[HappyStudio.UwpToolsLibrary.Information 1.0.2](https://www.nuget.org/packages/HappyStudio.UwpToolsLibrary.Information/) UWP 工具库的信息类库

[HappyStudio.UwpToolsLibrary.Control 1.1.0](https://www.nuget.org/packages/HappyStudio.UwpToolsLibrary.Control/) UWP 工具类库的控件库

[HappyStudio.UwpToolsLibrary 1.0.4](https://www.nuget.org/packages/HappyStudio.UwpToolsLibrary/) 给UWP工具类库的其他模块使用的依赖类库

[WinRT XAML Toolkit for Windows 10 2.3.0](https://www.nuget.org/packages/WinRTXamlToolkit.UWP/)
