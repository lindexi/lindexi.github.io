# win10 uwp 安装文件 appinstaller 格式

本文详细告诉大家 appinstaller 的格式，和如何通过 appinstaller 设置自动更新和安装

<!--more-->
<!-- CreateTime:2019/2/11 8:55:31 -->

<!-- csdn -->

在使用 appinstaller 安装 UWP 应用之前，需要系统的版本是 Windows 10 Fall Creators Update 以上，通过 appinstaller 文件可以用来分发应用给用户，不需要通过微软的商店。

在微软安装的应用，需要知道应用的三个部分，第一个是应用的本身，第二个是应用使用的可选包，第三个是依赖文件。

那么这三个分开的如何在安装的时候找到？就是使用 appinstaller 文件

这里 appinstaller 不是实体，不是包含代码的安装的应用，是一个 xml 文件，里面包含了链接，应用安装器通过这个文件可以进行安装应用。

这样只需要给用户一个 appinstaller 文件，就可以通过应用安装器找到应用需要的文件安装

这是从官方文档找到的完整代码

```
<?xml version="1.0" encoding="utf-8"?>
<AppInstaller
    xmlns="http://schemas.microsoft.com/appx/appinstaller/2017/2"
    Version="1.0.0.0" 
    Uri="http://mywebservice.azurewebsites.net/appset.appinstaller" > 

    <MainBundle 
        Name="Contoso.MainApp"
        Publisher="CN=Contoso"
        Version="2.23.12.43"
        Uri="http://mywebservice.azurewebsites.net/mainapp.appxbundle" />

    <OptionalPackages>
        <Bundle
            Name="Contoso.OptionalApp1"
            Publisher="CN=Contoso"
            Version="2.23.12.43"
            Uri="http://mywebservice.azurewebsites.net/OptionalApp1.appxbundle" />

        <Bundle
            Name="Contoso.OptionalApp2"
            Publisher="CN=Contoso"
            Version="2.23.12.43"
            Uri="http://mywebservice.azurewebsites.net/OptionalApp2.appxbundle" />

        <Package
            Name="Fabrikam.OptionalApp3"
            Publisher="CN=Fabrikam"
            Version="10.34.54.23"
            Uri="http://mywebservice.azurewebsites.net/OptionalApp3.appx" />

    </OptionalPackages>

</AppInstaller>
```

在用户通过应用安装器使用 appinstaller 安装的时候，就可以每个包的 URI 里面的链接的应用，通过 Name 和 Publisher 校验，所以这里的 Name 和 Publisher 和 Version 需要和应用的包的设置对应

下面从零开始创建一个 appinstaller 文件

首先通过 SublimeText 或 VSC 创建一个后缀是 appinstaller 的文件

一个最简单的 appinstaller 文件需要有下面的内容

```
<?xml version="1.0" encoding="utf-8"?>
<AppInstaller
    xmlns="http://schemas.microsoft.com/appx/appinstaller/2017/2"
    Version="1.0.0.0" 
    Uri="http://mywebservice.azurewebsites.net/appset.appinstaller" > 
</AppInstaller>
```

注意文件使用 utf8 格式

这里的 AppInstaller 就是告诉应用安装器这个文件的链接，就是上面的 Uri 里面的链接，还告诉应用安装器版本，这样就可以通过比较读取这个链接然后比较版本就知道是不是可以更新应用。

通过安装器的文件链接，可以告诉应用，下一次更新去哪个链接访问，如当前是从 abc.com/xx.appinstaller 访问，但是这个域名就快过期了，于是可以在这个文件里面修改 uri 为 新的域名，这样更新之后的应用就会去新的域名。

这里有一个细节是 xmlns 命名空间，这里的命名空间在给不同的系统使用是不相同的

- 给 win10 1809 和以上使用的是 http://schemas.microsoft.com/appx/appinstaller/2018

- 给 win10 1803 和以上使用的是 http://schemas.microsoft.com/appx/appinstaller/2017/2

- 给 win10 1709 和以上使用的是 http://schemas.microsoft.com/appx/appinstaller/2017

在 UWP 应用有主要包和可选包的区别，主要包就是核心的程序，很多 uwp 程序只有主要包。现在有 appxbundle 和 	msixbundle 两个不同的格式，但是这两个格式相同的写法，通过 MainBundle 如下代码。

```
<?xml version="1.0" encoding="utf-8"?>
<AppInstaller
    xmlns="http://schemas.microsoft.com/appx/appinstaller/2017/2"
    Version="1.0.0.0" 
    Uri="http://mywebservice.azurewebsites.net/appset.appinstaller" > 

    <MainBundle 
        Name="Contoso.MainApp"
        Publisher="CN=Contoso"
        Version="2.23.12.43"
        Uri="http://mywebservice.azurewebsites.net/mainapp.appxbundle" />

</AppInstaller>
```

如果主要包的格式是 appx 或 msix 文件，就使用 `MainPackage` 替换代码的 `MainBundle` 代码

```
<?xml version="1.0" encoding="utf-8"?>
<AppInstaller
    xmlns="http://schemas.microsoft.com/appx/appinstaller/2017/2"
    Version="1.0.0.0" 
    Uri="http://mywebservice.azurewebsites.net/appset.appinstaller" > 

    <MainPackage 
        Name="Contoso.MainApp"
        Publisher="CN=Contoso"
        Version="2.23.12.43"
        Uri="http://mywebservice.azurewebsites.net/mainapp.appxbundle" />

</AppInstaller>
```

在 AppInstaller 包含了 MainBundle 元素，在里面有 Name 表示主要包的名，发布者信息和版本，这里的 Uri 就是 appbudle 等文件的下载地址，看到了这里是不是就知道了如何通过 CDN 提供下载的方法？就是通过修改这个 URI 属性为 文件服务的链接，这样就可以从文件服务下载。如果不同的用户访问的时候返回不同的 appinstaller 文件，不同的文件的 uri 不相同，那么就可以让不同的用户在不同的服务器下载

上面的包的 Name 和 发布者版本信息需要和应用程序包清单里面的信息对应

这里的 Name 的要求是 3 到 50 个字符，发布者的信息要求是 1-8192 个字符，同时满足下面的正则表达式

```
(CN|L|O|OU|E|C|S|STREET|T|G|I|SN|DC|SERIALNUMBER|(OID\.(0|[1-9][0-9]*)(\.(0|[1-9][0-9]*))+))=(([^,+="<>#;])+|".*")(, ((CN|L|O|OU|E|C|S|STREET|T|G|I|SN|DC|SERIALNUMBER|(OID\.(0|[1-9][0-9]*)(\.(0|[1-9][0-9]*))+))=(([^,+="<>#;])+|".*")))*
```

之所以会说到这里就是因为我给小伙伴的演示随意写，然后发现无法安装，如果发现无法安装，请看一下自己的 appinstaller 文件是否没符合规则，特别需要小心 uri 的链接的长度是 1-2048 个字符



很少有开发者使用可选包，但是原因官网有告诉大家如何添加可选包，所以本文这里也写了。下面可选包内容我只是复制官网的内容，我自己没有试过，如果发现问题了，请到官网问大佬

可选包可以使用很多包，放在 OptionalPackages 里面，如下代码

```
<?xml version="1.0" encoding="utf-8"?>
<AppInstaller
    xmlns="http://schemas.microsoft.com/appx/appinstaller/2017/2"
    Version="1.0.0.0" 
    Uri="http://mywebservice.azurewebsites.net/appset.appinstaller" > 

    <MainBundle 
        Name="Contoso.MainApp"
        Publisher="CN=Contoso"
        Version="2.23.12.43"
        Uri="http://mywebservice.azurewebsites.net/mainapp.appxbundle" />

    <OptionalPackages>
        <Bundle
            Name="Contoso.OptionalApp1"
            Publisher="CN=Contoso"
            Version="2.23.12.43"
            Uri="http://mywebservice.azurewebsites.net/OptionalApp1.appxbundle" />

        <Bundle
            Name="Contoso.OptionalApp2"
            Publisher="CN=Contoso"
            Version="2.23.12.43"
            Uri="http://mywebservice.azurewebsites.net/OptionalApp2.appxbundle" />

        <Package
            Name="Fabrikam.OptionalApp3"
            Publisher="CN=Fabrikam"
            Version="10.34.54.23"
            Uri="http://mywebservice.azurewebsites.net/OptionalApp3.appx" />

    </OptionalPackages>

</AppInstaller>
```

每一个可选包带的信息都包括 Name 发布者和版本，下载的链接，这部分就不用多说，不同的包可以有不同的发布者和版本等，需要这些信息和可选包的信息相同

如果使用的是 appxbundle 和  msixbundle 两个不同的格式，就使用 Bundle 表示，如果使用  appx 或 msix 就使用 Package 表示

在安装 UWP 很重要就是依赖包，有开发桌面应用的小伙伴就知道，很多用户都有环境问题，在 UWP 安装就会下载 UWP 需要的依赖包

很多的 UWP 包都依赖 VC 的库，如需要引用 VC 的库，可以使用下面的代码

```
    <Dependencies>
        <Package Name="Microsoft.VCLibs.140.00" Publisher="CN=Microsoft Corporation, O=Microsoft Corporation, L=Redmond, S=Washington, C=US" Version="14.0.24605.0" ProcessorArchitecture="x86" Uri="http://foobarbaz.com/fwkx86.appx" />
    </Dependencies>
```

在这里也可以知道，不同的 UWP 可以引用相同的依赖库，所以通过 uri 可以让不同的安装程序使用相同的链接的依赖包，这样可以减少一点空间

现在看起来的 appinstaller 文件的代码已经很长

```
<?xml version="1.0" encoding="utf-8"?>
<AppInstaller
    xmlns="http://schemas.microsoft.com/appx/appinstaller/2017/2"
    Version="1.0.0.0" 
    Uri="http://mywebservice.azurewebsites.net/appset.appinstaller" > 

    <MainBundle 
        Name="Contoso.MainApp"
        Publisher="CN=Contoso"
        Version="2.23.12.43"
        Uri="http://mywebservice.azurewebsites.net/mainapp.appxbundle" />

    <OptionalPackages>
        <Bundle
            Name="Contoso.OptionalApp1"
            Publisher="CN=Contoso"
            Version="2.23.12.43"
            Uri="http://mywebservice.azurewebsites.net/OptionalApp1.appxbundle" />

        <Bundle
            Name="Contoso.OptionalApp2"
            Publisher="CN=Contoso"
            Version="2.23.12.43"
            Uri="http://mywebservice.azurewebsites.net/OptionalApp2.appxbundle" />

        <Package
            Name="Fabrikam.OptionalApp3"
            Publisher="CN=Fabrikam"
            Version="10.34.54.23"
            ProcessorArchitecture="x86"
            Uri="http://mywebservice.azurewebsites.net/OptionalApp3.appx" />

    </OptionalPackages>

    <Dependencies>
        <Package Name="Microsft.VCLibs.140.00" Publisher="CN=Microsoft Corporation, O=Microsoft Corporation, L=Redmond, S=Washington, C=US" Version="14.0.24605.0" ProcessorArchitecture="x86" Uri="http://foobarbaz.com/fwkx86.appx" />
        <Package Name="Microsoft.VCLibs.140.00" Publisher="CN=Microsoft Corporation, O=Microsoft Corporation, L=Redmond, S=Washington, C=US" Version="14.0.24605.0" ProcessorArchitecture="x64" Uri="http://foobarbaz.com/fwkx64.appx" />
    </Dependencies>

</AppInstaller>
```

在应用安装文件可以告诉安装应用程序更新的频率，也就是多少小时之后访问这个文件，判断是不需要更新，这个属性是可选的。在 OnLaunch 指定时间多少小时访问文件，如 HoursBetweenUpdateChecks=10 就是 10 个小时访问一次这个文件，判断是不是需要更新。如果没有设置 HoursBetweenUpdateChecks 默认就是一天，如果设置为 0 就是每次启动应用的时候访问这个文件

```
    <UpdateSettings>
        <OnLaunch HoursBetweenUpdateChecks="12" />
    </UpdateSettings>
```

自动更新除了使用 OnLaunch 还可以使用 AutomaticBackgroundTask 通过这个可以在 8 小时，即使应用没有启动的时候都会访问是否更新

```
<UpdateSettings>
    <AutomaticBackgroundTask/>
</UpdateSettings>
```

有时候更新了程序发现这个程序有坑，需要回滚，但是默认的自动更新只是判断文件版本比当前的新才更新，需要通过 ForceUpdateFromAnyVersion 才可以判断 appinstaller 文件的版本和当前的不同就更新，支持更新到新的版本或比当前小的版本

```
<UpdateSettings>
    <OnLaunch HoursBetweenUpdateChecks="12"/>
    <AutomaticBackgroundTask/>
    <ForceUpdateFromAnyVersion>true</ForceUpdateFromAnyVersion>
</UpdateSettings>
```

如果需要使用 UpdateSettings 就需要使用命名空间为 http://schemas.microsoft.com/appx/appinstaller/2017/2 这在 1803 和以上的系统才能使用

在 1809 之后的 OnLaunch 提供特别复杂的功能，本文在这里就不告诉大家

总结一下，在 appinstaller 一个 xml 文件，根元素是 AppInstaller 包括命名空间和版本，这里的版本的格式是 主版本.次版本.构建号.修订号 里面可以包含下面内容

- MainPackage 或 MainBundle 主要包

- Dependencies 依赖包，不是必须

- OptionalPackages 可选包 不是必须

- RelatedPackages 相关库，不是必须

- UpdateSettings 更新设置 不是必须

在这里需要注意，因为 appinstaller 可以指定 uri 告诉应用安装程序去哪里更新这个 appinstaller 文件，如果这里的 uri 和当前访问的不相同，那么应用安装程序就会去新的 uri 访问，如果访问新的 uri 发现这个 appinstaller 文件里面的 uri 和当前的 uri 还是不相同，那么应用安装程序会继续访问新的 uri 但是这个跳转只能三次，如果再次发现链接和当前的路劲不相同，那么就会告诉用户安装失败

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
