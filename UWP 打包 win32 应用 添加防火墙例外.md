# UWP 打包 win32 应用 添加防火墙例外

我想要将一个 WPF 应用打包为 UWP 应用，然后用我自己的商店发布，在做 UWP 安装包的小伙伴会问如何和 nsis 一样添加防火墙例外

<!--more-->
<!-- CreateTime:2020/2/10 19:58:14 -->

<!-- 发布 -->

应用没有在安装添加例外，会在应用开放服务监听端口提示 Windows Defender 防火墙已组织所有共用网络和专用网络上的 xx 的某些功能，如果用户没有点击允许访问，那么一些功能访问可能会因为防火墙不让用，用 nsis 添加防火墙例外建议是添加一个应用可以通过

我的 WPF 程序是 WPF 程序加上 asp dotnet core 程序，我这是将 asp dotnet core 作为客户端使用，主要作用是 ipc 也就是进程通讯，用这个方法没有什么好处，下次有小伙伴要用，我推荐 [dotnet-campus/dotnetCampus.IPC.WCF 一个基于WCF适用于C#项目进行IPC通信的库](https://github.com/dotnet-campus/dotnetCampus.IPC.WCF )

在我的例子里，我用一个纯 asp dotnet core 代替，请看 [github](https://github.com/lindexi/lindexi_gd/tree/3e907b2ee99fbe61f43e07bd8fb43e9c056befc6/JadallhearfairBarwalbegear) 的 `Main\NarhedeachawhearWeargijawgowe` 项目，这个项目是这样创建的。先用 `dotnet new webapi -o NarhedeachawhearWeargijawgowe` 创建空白项目，然后打开 Program.cs 文件，添加下面代码

```csharp
webBuilder.UseUrls("http://0.0.0.0:12307");
```

这样就可以让这个应用在电脑上开启防火墙会提示用户是否允许让这个应用访问

我推荐用 UWP 打包的 win32 程序都在另一个启动器项目里面运行，这样就能做到我不需要改动原有的 win32 项目的代码，我可以给任意第一个win32应用打包上架到应用商店

方法是我创建一个 net461 的控制台项目，创建方法是通过 dotnet new console -o DalljalfonafearBairyekeregu 创建一个空白的控制台项目，然后打开 csproj 文件，更改下面代码

```
<Project Sdk="Microsoft.NET.Sdk">


    <PropertyGroup>
        <OutputType>WinExe</OutputType>
        <TargetFramework>net461</TargetFramework>
      
    </PropertyGroup>

    <ItemGroup>
        <None Include="..\Main\NarhedeachawhearWeargijawgowe\bin\Debug\netcoreapp3.1\**\**">
            <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
        </None>
    </ItemGroup>


</Project>

```

上面代码将 OutputType 修改为 WinExe 的作用是让这个控制台项目不会显示黑框，而 net461 的版本是为了在打包项目引用

用 net461 而不是 47 的原因是打包项目现在用的是 .NET Core 5 不能引用 47 而用 .NET Core 需要设置独立部署这样打包的应用很大

上面代码的 ItemGroup 的内容就是引用另一个应用的所有文件夹，这里的 CopyToOutputDirectory 设置将所有文件输出，这样就能在打包时输出应用

打开 DalljalfonafearBairyekeregu 也就是启动器项目的 Program.cs 文件，给这个文件添加下面代码

```csharp
        static void Main(string[] args)
        {
            var file = Path.Combine(Assembly.GetExecutingAssembly().Location, @"..\NarhedeachawhearWeargijawgowe.exe");
            Process.Start(file);
        }
```

这样就能调用实际的应用

关于启动器代码请看 [github](https://github.com/lindexi/lindexi_gd/tree/3e907b2ee99fbe61f43e07bd8fb43e9c056befc6/JadallhearfairBarwalbegear) 的 DalljalfonafearBairyekeregu 文件

这个项目的细节是 WinExe 和 net461 的设置，还有 Main 里面记得调用原先的应用

创建打包项目 JadallhearfairBarwalbegear 这个项目需要右击应用程序，添加启动器的引用

如果是一个不需要添加防火墙例外的应用，这样做就完成了

需要添加防火墙例外的需要右击 Package.appxmanifest 查看代码，添加下面代码

```
    <Extensions>
        <desktop2:Extension Category="windows.firewallRules">
            <desktop2:FirewallRules Executable="DalljalfonafearBairyekeregu\NarhedeachawhearWeargijawgowe.exe">
                <desktop2:Rule Direction="in" IPProtocol="TCP" Profile="all"/>
                <desktop2:Rule Direction="in" IPProtocol="UDP"
                               Profile="all"/>
                <desktop2:Rule Direction="out"
                               IPProtocol="TCP"
                               Profile="all"/>
                <desktop2:Rule Direction="out"
                               IPProtocol="UDP"
                               Profile="all"/>
            </desktop2:FirewallRules>
        </desktop2:Extension>
    </Extensions>
```

上面代码需要添加命名空间

```
  xmlns:desktop2="http://schemas.microsoft.com/appx/manifest/desktop/windows10/2"
```

详细代码请看 [github](https://github.com/lindexi/lindexi_gd/tree/3e907b2ee99fbe61f43e07bd8fb43e9c056befc6/JadallhearfairBarwalbegear) 的 JadallhearfairBarwalbegear\Package.appxmanifest 文件

上面代码的细节是需要放在 Extensions 里面，而 Executable 的路径是放在启动器文件夹里面，如果输入的文件找不到在点击发布提示没有文件

添加防火墙规则建议添加 tcp 和 udp 和出站入站请看上面代码，通过 Profile 可以设置文件例外可以访问端口

用上面方法可以给win32应用，不管这个应用是不是咱的，打包为 UWP 应用

打包为 UWP 应用可以用自己做的应用商店发布，自动更新

如何自己做一个 UWP 应用商店，请看 [加强版在国内分发 UWP 应用正确方式 通过win32安装UWP应用](https://blog.lindexi.com/post/%E5%8A%A0%E5%BC%BA%E7%89%88%E5%9C%A8%E5%9B%BD%E5%86%85%E5%88%86%E5%8F%91-UWP-%E5%BA%94%E7%94%A8%E6%AD%A3%E7%A1%AE%E6%96%B9%E5%BC%8F-%E9%80%9A%E8%BF%87win32%E5%AE%89%E8%A3%85UWP%E5%BA%94%E7%94%A8.html )

如果你看代码不知道如何打包，我推荐你从[github](https://github.com/lindexi/lindexi_gd/tree/3e907b2ee99fbe61f43e07bd8fb43e9c056befc6/JadallhearfairBarwalbegear)下载我的代码，用 VisualStudio 2019 打开，先使用 dotnet build 构建 Main\NarhedeachawhearWeargijawgowe 代码

然后双击 Package.appxmanifest 点击打包，创建一个你自己的证书，然后右击 JadallhearfairBarwalbegear 项目发布，试试发布后的项目。如果成功那么再将这个项目修改为你需要的项目

从法律上，更改非自己的应用然后发布是不合法的，但是在中国一堆盗版软件，这个可以忽略，等你的应用商店做起来之后再找对应的公司讨论

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
