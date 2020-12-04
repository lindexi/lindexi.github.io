# WPF 从零手动创建承载 Xamarin Forms 项目

现在完全开源的 Xamarin Forms 是支持使用 WPF 进行承载，也就是使用 Xamarin 开发的控件等是可以在 WPF 项目使用的。本文来告诉大家如何在 WPF 中运行 Xamarin Forms 项目，让 Xamarin Forms 构建为 WPF 应用

<!--more-->
<!-- CreateTime:2020/8/8 9:14:15 -->



默认的 VS 没有加上 WPF 的模版，而官方文档 [WPF Platform Setup - Xamarin](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/platform/other/wpf ) 旧了一点，因为社区的开发比较激进，而文档没有更新

如果你按照官方文档玩，预计会在构建的时候看到如下提示

```csharp
 App.xaml : error :  : XamlC error XFC0000 : Cannot resolve type "Application"
```

当然，现在是 2020.07.31 也许你看本文的博客，官方文档更新了，而本文也失效了。此时请不要以为我在骗你

上面代码的原因是此时不需要使用 App.xaml 了，也不需要使用 MainWindow.xaml 了，让咱手动从零开始创建

当然，需要先存在一个 Xamarin Forms 项目哈，最好这是一个使用模版重新创建的项目，使用的版本都是 4.8 以上。我推荐是新创建一个，这样你通过之后，才进行修改，能解决因为自己原有的 Xamarin Forms 项目的坑让代码构建失败

新建一个叫 Xx.WPF.csproj 的项目，请将 Xx 替换为你自己的名字。使用 WPF 项目没有安卓项目那么弱，对命名长度要求比较多，在安卓项目里面如果你敢将名字命名比较长，那么将会因为路径太长炸掉，详细请看 [Xamarin 构建安卓失败 因为路径太长](https://blog.lindexi.com/post/Xamarin-%E6%9E%84%E5%BB%BA%E5%AE%89%E5%8D%93%E5%A4%B1%E8%B4%A5-%E5%9B%A0%E4%B8%BA%E8%B7%AF%E5%BE%84%E5%A4%AA%E9%95%BF.html)

在 Xx.WPF.csproj 添加如下代码

```
<Project Sdk="Microsoft.NET.Sdk.WindowsDesktop">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net472</TargetFramework>
    <UseWPF>true</UseWPF>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Xamarin.Forms.Platform.WPF" Version="4.8.0.1269" />
  </ItemGroup>

</Project>
```

可以看到上面代码的是十分简单的逻辑代码。但是如上面代码写的，使用的 TargetFramework 版本是 .NET Framework 4.7.2 这就意味着最低支持的系统是 Win7 带 Sp1 的系统。因为 Win7 非 sp1 最高版本 .NET Framework 是 4.5.2 同时不支持 .NET Core 任何版本，而 Win7 加上 Sp1 的系统能支持到 .NET Framework 4.8 的版本和 .NET Core 版本

那为什么需要采用 .NET Framework 4.7.2 的版本？因为 需要有 [OpenTK](https://github.com/dotnet-campus/opentk) 的支持，而 OpenTK 最低是 .NET Framework 4.6.1 因此暂时无法降级到 .NET Framework 4.5 版本用来支持 Win7 非 sp1 系统

好，继续写一个叫 Program.cs 的类，小伙伴可以看到，一个 WPF 程序是只有 csproj 文件和 Program.cs 文件就可以完成对 Xamarin Forms 项目的承载

在 Program.cs 创建主函数

```csharp
        static void Main(string[] args)
        {

        }
```

注意需要给 Main 添加 STA 线程

```csharp
        [STAThread]
        static void Main(string[] args)
        {

        }
```

如果没有加上这个特性，那么将会在运行提示如下代码

```csharp
System.InvalidOperationException:“调用线程必须为 STA，因为许多 UI 组件都需要。”
```

接下来就是创建 Application 创建 WPF 应用，然后运行消息调度，接着加载 Xamarin Forms 应用作为界面

```csharp
            var application = new Application();
            Forms.Init();
            var formsApplicationPage = new FormsApplicationPage();
            formsApplicationPage.LoadApplication(new XamarinNeller.App());
            application.Run(formsApplicationPage);
```

此时就完成了，试试运行一下



代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/96c9063fdba9fe318eb099da67422de5cc9ae5af/XamarinNeller/XamarinNeller.WPF) 欢迎小伙伴访问


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
