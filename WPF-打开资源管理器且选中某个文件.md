
# WPF 打开资源管理器且选中某个文件

本文将和大家介绍如何在 Windows 系统上使用 SHOpenFolderAndSelectItems 方法打开资源管理器且选中给定的文件

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

打开资源管理器且选中某个文件可以使用 cmd 调用 explorer 带上 select 参数，如下面命令行所示

```bash
explorer.exe /select,"C:\Folder\file.txt"
```

但有很多情况下，用户可能使用其他资源管理器，此时将会导致应用软件打开的是 explorer 而不是用户默认的资源管理器

通过 shell32.dll 提供的 SHOpenFolderAndSelectItems 方法，可以直接使用函数调用的方式打开资源管理器且选中某个文件，且使用的是用户设置的默认的资源管理器

以下是我创建的简单的 WPF 例子程序的界面，可以看到界面非常简单，就是输入一个文件，然后点击按钮就可以打开资源管理器选中输入的文件

```xml
    <Grid>
        <Grid VerticalAlignment="Center">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="Auto"></ColumnDefinition>
                <ColumnDefinition Width="*"></ColumnDefinition>
                <ColumnDefinition Width="Auto"></ColumnDefinition>
            </Grid.ColumnDefinitions>
            <TextBlock Text="文件路径：" Margin="50,0,0,0" VerticalAlignment="Center"/>
            <TextBox x:Name="InputTextBox"  Grid.Column="1" Margin="10,0,10,0" VerticalAlignment="Center"/>
            <Button Grid.Column="2" Content="打开" Margin="10,0,50,0" VerticalAlignment="Center" Click="Button_OnClick"/>
        </Grid>
    </Grid>
```

按钮的后台代码将需要使用 PInvoke 调用 Win32 函数。对于 dotnet 7 以前的程序，可使用如下方式定义

```csharp
    [DllImport("shell32.dll", ExactSpelling = true)]
    private static extern void ILFree(IntPtr pidlList);

    [DllImport("shell32.dll", CharSet = CharSet.Unicode, ExactSpelling = true)]
    private static extern IntPtr ILCreateFromPathW(string pszPath);

    [DllImport("shell32.dll", ExactSpelling = true)]
    private static extern int SHOpenFolderAndSelectItems(IntPtr pidlList, uint cild, IntPtr children, uint dwFlags);
```

对于 dotnet 7 以及更高版本的项目，可使用 LibraryImportAttribute 特性辅助定义。如以下 C# 代码所示

```csharp
    [LibraryImport("shell32.dll")]
    private static partial void ILFree(IntPtr pidlList);

    [LibraryImport("shell32.dll", StringMarshalling = StringMarshalling.Utf16)]
    private static partial IntPtr ILCreateFromPathW(string pszPath);

    [LibraryImport("shell32.dll")]
    private static partial int SHOpenFolderAndSelectItems(IntPtr pidlList, uint cild, IntPtr children, uint dwFlags);
```

过程中别忘了在 csproj 项目文件里面开启不安全代码，开启之后的项目文件代码大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net9.0-windows</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <UseWPF>true</UseWPF>
    <AllowUnsafeBlocks>True</AllowUnsafeBlocks>
  </PropertyGroup>

</Project>
```

后台 C# 代码的按钮点击事件里面将调用 SHOpenFolderAndSelectItems 方法打开资源管理器选中输入的文件

```csharp
    private void Button_OnClick(object sender, RoutedEventArgs e)
    {
        var filePath = InputTextBox.Text;
        filePath = System.IO.Path.GetFullPath(filePath);

        IntPtr pidlList = ILCreateFromPathW(filePath);
        if (pidlList != IntPtr.Zero)
        {
            try
            {
                Marshal.ThrowExceptionForHR(SHOpenFolderAndSelectItems(pidlList, 0, IntPtr.Zero, 0));
            }
            finally
            {
                ILFree(pidlList);
            }
        }
    }

```

以上代码里面的 ILCreateFromPathW 要求传入绝对路径，需要调用 `System.IO.Path.GetFullPath` 方法转换传入路径为绝对路径

如果不知道代码如何写的话，可以拉取我的例子项目代码跑跑看

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/6988631e41226832c3b83cf62529eb7d7892e0b2/WPFDemo/WilinojearcheWheyecearhire) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/6988631e41226832c3b83cf62529eb7d7892e0b2/WPFDemo/WilinojearcheWheyecearhire) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 6988631e41226832c3b83cf62529eb7d7892e0b2
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 6988631e41226832c3b83cf62529eb7d7892e0b2
```

获取代码之后，进入 WPFDemo/WilinojearcheWheyecearhire 文件夹，即可获取到源代码

更多 WPF 博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

## 参考文档

[c# - How to open Explorer with a specific file selected? - Stack Overflow](https://stackoverflow.com/questions/13680415/how-to-open-explorer-with-a-specific-file-selected )

[file - C#: How to use SHOpenFolderAndSelectItems - Stack Overflow](https://stackoverflow.com/questions/3018002/c-how-to-use-shopenfolderandselectitems )

[c#: 打开文件夹并选中文件 - 楚人无衣 - 博客园](https://www.cnblogs.com/crwy/p/SHOpenFolderAndSelectItems.html )

[SHOpenFolderAndSelectItems 函数 (shlobj_core.h) - Win32 apps - Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/api/shlobj_core/nf-shlobj_core-shopenfolderandselectitems )

[【C#】在Windows资源管理器打开文件夹，并选中指定的文件或文件夹 - Tod's - 博客园](https://www.cnblogs.com/tods/p/17614343.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。