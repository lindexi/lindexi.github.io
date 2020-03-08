# 在 GitHub 仓库添加 NuGet 版本图标和构建图标

其实这两篇博客我都写过，但是放在一起方便我新建项目的时候复制代码。在 GitHub 的首页上，很多开源项目都会写出当前构建是通过还是不通过，如果是提供 NuGet 包的还添加 NuGet 版本图标

<!--more-->
<!-- 发布 -->

我推荐在 Github 上使用 Action 构建，此时可以通过 [Github 添加 Action 编译图标](https://blog.lindexi.com/post/Github-%E6%B7%BB%E5%8A%A0-Action-%E7%BC%96%E8%AF%91%E5%9B%BE%E6%A0%87.html) 这个方法添加构建图标

写法是 `![](https://github.com/组织或个人/仓库/workflows/执行Action的Name注意转码/badge.svg)` 

而添加 NuGet 版本图标可以使用以下格式

```csharp
[![](https://img.shields.io/nuget/v/NuGet包的Id字符串.svg)](https://www.nuget.org/packages/NuGet包的Id字符)
```

看起来上面这个链接复杂的原因是包含了图片和图片点击跳转的链接

仅图片代码是 `![](https://img.shields.io/nuget/v/NuGet包的Id字符串.svg)` 而仅链接代码是 `[这是链接显示文字](https://www.nuget.org/packages/NuGet包的Id字符)` 将链接里面的显示文字替换为图片就是上面代码了

我推荐在首页放下面这个表格

```csharp
| Build | NuGet |
|--|--|
|![](https://github.com/组织或个人/仓库/workflows/执行Action的Name注意转码/badge.svg)|[![](https://img.shields.io/nuget/v/NuGet包的Id字符串.svg)](https://www.nuget.org/packages/NuGet包的Id字符)|
```

显示效果大概如下

| Build | NuGet |
|--|--|
|![](https://github.com/dotnet-campus/dotnetCampus.TagToVersion/workflows/.NET%20Core/badge.svg)|[![](https://img.shields.io/nuget/v/dotnetCampus.TagToVersion.svg)](https://www.nuget.org/packages/dotnetCampus.TagToVersion)|

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
