
# WPF 如何找到资源文件路径包含 # 号的文件

本文告诉大家如何在 WPF 获取资源文件包含 # 号的文件资源

<!--more-->



<!-- 发布 -->

我遇到一个有意思的设计师小伙伴，他的文件命名喜欢使用 `#数字` 的方式命名，例如写一个图片文件，他的命名是 `Image#1.png` 和 `Image#2.png` 的格式

如果在 WPF 中拖入的图片，通过属性设置作为资源，默认是可以在 XAML 里面进行引用，使用相对或绝对路径引用，如下面代码

```xml
  <Grid>
    <Image x:Name="Image" Width="200" Height="200" Stretch="Fill" Source="lindexidoubi.png" />
  </Grid>
```

以上代码需要在解决方案里面放一个 lindexidoubi.png 文件，同时设置属性生成作为资源，可以在 csproj 上看到代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk.WindowsDesktop">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>netcoreapp3.1</TargetFramework>
    <UseWPF>true</UseWPF>
  </PropertyGroup>

  <ItemGroup>
    <Resource Include="lindexidoubi.png" />
  </ItemGroup>

</Project>
```

在后台代码可以使用如下代码获取

```csharp
 var streamResourceInfo = Application.GetResourceStream(new Uri("lindexidoubi.png", UriKind.Relative));
```

看起来这样的代码清真，但是我遇到的设计师给了我一堆图片，这些图片的命名都有 # 号

于是我更换了 xaml 的代码如下

```xml
  <Grid>
    <Image x:Name="Image" Width="200" Height="200" Stretch="Fill" Source="lindexi#doubi.png" />
  </Grid>
```

运行时将不会有任何显示，在 Loaded 事件里面尝试获取图片的 Source 拿到的是空

```csharp
        public MainWindow()
        {
            InitializeComponent();

            Loaded += MainWindow_Loaded;
        }

        private void MainWindow_Loaded(object sender, RoutedEventArgs e)
        {
            var imageSource = Image.Source;
        }
```

而通过后台代码，如下面代码获取将提示找不到文件

```csharp
            var streamResourceInfo = Application.GetResourceStream(new Uri("lindexi#doubi.png", UriKind.Relative));
```

提示的内容如下

```
System.IO.IOException:“找不到资源“lindexi”。”
```

可以看到，传入的是 `lindexi#doubi.png` 但忽略了 `#` 后面的内容。这是因为 Uri 转义的问题，需要使用如下代码才能拿到

```csharp
 var streamResourceInfo = Application.GetResourceStream(new Uri(Uri.EscapeDataString("lindexi#doubi.png"), UriKind.Relative));
```

同理，需要在 XAML 将 `#` 转义，通过 Uri.EscapeDataString 可以了解到 `#` 可以的转义如下

```xml
  <Grid>
    <Image x:Name="Image" Width="200" Height="200" Stretch="Fill" Source="lindexi%23doubi.png" />
  </Grid>
```

于是我就不用和设计师打起来了

在 WPF 中是支持资源的文件路径名包含了 # 号的，但是在使用的时候需要进行转义，通过 Uri 的 EscapeDataString 方法而不是 EscapeUriString 方法进行转换才能拿到资源

本文上面代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/9b8e547f21e4a36d4c5aafec87e08d0ef4bcacb5/CarqawlawyofuwairfuJalbeewhaidearheebee) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/9b8e547f21e4a36d4c5aafec87e08d0ef4bcacb5/CarqawlawyofuwairfuJalbeewhaidearheebee) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 9b8e547f21e4a36d4c5aafec87e08d0ef4bcacb5
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 CarqawlawyofuwairfuJalbeewhaidearheebee 文件夹





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。