# dotnet 使用 WpfAnalyzers 辅助分析 WPF 应用代码缺陷

引入 WpfAnalyzers 代码分析工具，相当于给团队加入一个免费的代码审查工具人，可以帮忙在日常开发找到很多代码缺陷。加入 WpfAnalyzers 代码分析工具，可以减少代码编写里的低级缺陷，提升开发质量

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

这是一个基于 Roslyn 的免费开源的 WPF 代码分析工具，专门给 WPF 应用使用，此工具包含了约 200 条 WPF 应用编写规则，在 GitHub 上使用 MIT 最友好协议开放所有源代码，请看 [https://github.com/DotNetAnalyzers/WpfAnalyzers](https://github.com/DotNetAnalyzers/WpfAnalyzers)

此工具是存放在 [.NET Analyzers](https://github.com/DotNetAnalyzers) 组织下的应用，此开源组织不仅包含了 WPF 代码分析工具，还包含了其他很多应用的分析工具

使用 WpfAnalyzers 代码分析工具的方法特别简单，此代码分析工具是基于 Roslyn 代码分析编写的工具，可以通过 NuGet 分发，咱只需要在 VisualStudio 里面，通过 NuGet 管理安装 WpfAnalyzers 即可

![](http://image.acmx.xyz/lindexi%2F20211226922153469.jpg)

或者是编辑 csproj 项目文件，加入以下代码

```xml
  <ItemGroup>
    <PackageReference Include="WpfAnalyzers" Version="4.0.1">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
  </ItemGroup>
```

安装完成 NuGet 之后，将自动启用此代码分析工具。此时看看自己编写的逗比代码，将会看到分析器开始工作，告诉大家代码是否存在坑。如以下逗比代码

```csharp
    public static readonly DependencyProperty Foo1Property
        = DependencyProperty.Register
        (
            nameof(Title),
            typeof(string),
            typeof(MainWindow),
            new PropertyMetadata(default(string))
        );

    public string Foo2
    {
        get => (string) GetValue(Foo1Property);
        set => SetValue(Foo1Property, value);
    }
```

大家看出有几个锅了？代码分析器可以帮助咱看到如下图的问题，大家觉得这个代码分析工具好用不

![](http://image.acmx.xyz/lindexi%2F20211226924546566.jpg)

更有趣的是，如果大家看到分析器告诉咱的代码存在缺陷，但是咱就是不懂为什么。此时可以点击一下前面的 WPFxxx 链接。例如点击上图的 WPF0001 链接，将会跳转到 GitHub 上的对应页面。例如 WPF0001 的内容如下

![](http://image.acmx.xyz/lindexi%2F20211226926369862.jpg)

通过 GitHub 上的描述，咱就可以知道，要求依赖属性的 name 内容和依赖属性的定义相同。如上面代码里面，应该改掉 `nameof(Title)` 的代码。不过，改哪里和如何改还不用咱费心，此分析工具也带入了修补工具的功能，只需要点点鼠标即可进行自动修复更改，如下图

![](http://image.acmx.xyz/lindexi%2F202112269283981.jpg)

好，广告结束，五毛钱到手

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
