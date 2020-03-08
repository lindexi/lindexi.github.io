# C# 使用转换语义版本号

本文告诉大家如何转换语义版本号，那么什么是语义版本号，语义版本号（semantic version）就是版本号带 alpha 等的版本号

<!--more-->
<!-- CreateTime:2018/12/25 9:25:41 -->


在以前的版本号都是这样 `1.2.1` 的格式，这个格式可以使用微软的 Version 类转换

```csharp
            var str = "1.2.1";

            var version = Version.Parse(str);
```

如果需要使用语义版本号如`1.2.1-alpha`的格式，或看起来就很难解析的`1.2.45-alpha-beta+nightly.23.43-bla` 就不能使用微软提供的 version 转换

不要看语义版本号很复杂，实际上也是有规则的，请看[语义版本号（Semantic Versioning） - walterlv](https://walterlv.github.io/post/semantic-version.html )， [官方文档](https://semver.org/lang/zh-CN/ )

可以安装 [semantic version library for .Net](https://github.com/maxhauser/semver ) 转换，安装方法是使用 Nuget 搜索 semver 或输入` Install-Package semver`安装

安装完成，如转换 `1.2.45-alpha-beta+nightly.23.43-bla` ，可以使用这个代码

```csharp
using Semver;

// 忽略代码

            var str = "1.2.45-alpha-beta+nightly.23.43-bla";

            var version = SemVersion.Parse(str);
```

如果不想安装库，可以复制代码<https://gist.github.com/yadyn/959467> ，如果觉得这个库不靠谱，可以复制 git tools 项目的代码

[GitVersion/SemanticVersion.cs at master](https://github.com/GitTools/GitVersion/blob/master/src/GitVersionCore/SemanticVersion.cs )

参见：[语义版本号（Semantic Versioning） - walterlv](https://walterlv.github.io/post/semantic-version.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
