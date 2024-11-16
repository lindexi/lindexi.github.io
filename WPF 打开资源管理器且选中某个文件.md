# WPF 打开资源管理器且选中某个文件

本文将和大家介绍如何在 Windows 系统上使用 SHOpenFolderAndSelectItems 方法打开资源管理器且选中给定的文件

<!--more-->
<!-- CreateTime:2024/11/16 07:29:42 -->

<!-- 发布 -->
<!-- 博客 -->

## 命令行方法

打开资源管理器且选中某个文件可以使用 cmd 调用 explorer 带上 select 参数，如下面命令行所示

```bash
explorer.exe /select,"C:\Folder\file.txt"
```

但有很多情况下，用户可能使用其他资源管理器，此时将会导致应用软件打开的是 explorer 而不是用户默认的资源管理器

## SHOpenFolderAndSelectItems 单文件

通过 shell32.dll 提供的 [SHOpenFolderAndSelectItems](https://learn.microsoft.com/en-us/windows/win32/api/shlobj_core/nf-shlobj_core-shopenfolderandselectitems ) 方法，可以直接使用函数调用的方式打开资源管理器且选中某个文件，且使用的是用户设置的默认的资源管理器

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

更多一些细节信息是调用 SHOpenFolderAndSelectItems 之前请确保已经初始化 COM 组件，即调用过 [CoInitialize](https://learn.microsoft.com/zh-cn/windows/win32/api/combaseapi/nf-combaseapi-coinitializeex) 方法。在 WPF 里面为了和 DirectX 等交互，在按钮点击之前就已经调研过了 COM 初始化了，因此在 WPF 里面可以省略此逻辑。但是在控制台应用里面，需要手动调用一下，代码如下

```csharp
CoInitialize(0, 0);

    [LibraryImport("Ole32.dll")]
    private static partial int CoInitialize(IntPtr pvReserved, uint dwCoInit);
```

我再次更新 WPF 例子项目的代码，在按钮点击的方法里面调用。不过在按钮点击方法里面调用是必然返回失败的，如上文所述，这是因为 WPF 早已初始化过了。好在这个方法失败了也没有什么问题，可以放心调用

```csharp
    private void Button_OnClick(object sender, RoutedEventArgs e)
    {
        // 必定返回失败，因为 WPF 已经调用过了
        var result = CoInitialize(0, 0);

        ... // 忽略其他代码
    }
```

根据 [SHOpenFolderAndSelectItems](https://learn.microsoft.com/en-us/windows/win32/api/shlobj_core/nf-shlobj_core-shopenfolderandselectitems ) 文档如下描述，如果没有先调用 [CoInitialize](https://learn.microsoft.com/zh-cn/windows/win32/api/combaseapi/nf-combaseapi-coinitializeex) 则会失败

> CoInitialize or CoInitializeEx must be called before using SHOpenFolderAndSelectItems. Not doing so causes SHOpenFolderAndSelectItems to fail.

## SHOpenFolderAndSelectItems 选中多个文件

有时候咱的需求是打开文件夹，选中里面多个文件，此时依然可以使用 [SHOpenFolderAndSelectItems](https://learn.microsoft.com/en-us/windows/win32/api/shlobj_core/nf-shlobj_core-shopenfolderandselectitems ) 方法，只是咱需要修改一下刚才的函数签名。修改之后的代码如下

```csharp
// 修改前：
    [LibraryImport("shell32.dll")]
    private static partial int SHOpenFolderAndSelectItems(IntPtr pidlList, uint cild, IntPtr children, uint dwFlags);

// 修改后：
    [LibraryImport("shell32.dll")]
    private static partial int SHOpenFolderAndSelectItems(IntPtr pidlList, uint cild, [MarshalAs(UnmanagedType.LPArray)] IntPtr[] children, uint dwFlags);
```

可以看到修改后的差别只是将 `children` 参数的类型修改为 `IntPtr[]` 指针数组类型，且标记了作为 LPArray 方式传入而已。如果不想改的话，那也可以自己使用 `System.Runtime.InteropServices.Marshalling.ArrayMarshaller<nint, nint>.ManagedToUnmanagedIn` 的 GetPinnableReference 方法将指针数组转换为指针传入。只不过此时的指针对应在 C 的定义是指针的指针而已

选中多个文件的使用方法就是在 `pidlList` 参数传入多个文件所在的文件夹，在 `children` 参数里面传入需要选中的文件

传入的这些路径都需要经过 ILCreateFromPathW 处理，以下是我修改之后的按钮点击事件代码，可以全选文件夹里面的所有文件

```csharp
    private void Button_OnClick(object sender, RoutedEventArgs e)
    {
        // 必定返回失败，因为 WPF 已经调用过了
        var result = CoInitialize(0, 0);

        var folderPath = InputTextBox.Text;

        folderPath = System.IO.Path.GetFullPath(folderPath);

        IntPtr pidlList = ILCreateFromPathW(folderPath);


        if (pidlList != IntPtr.Zero)
        {
            var fileList = Directory.GetFiles(folderPath);

            var selectedFileList = new IntPtr[fileList.Length];

            for (var i = 0; i < fileList.Length; i++)
            {
                var file = fileList[i];
                selectedFileList[i] = ILCreateFromPathW(file);
            }

            try
            {
                // Open parent folder and select item
                Marshal.ThrowExceptionForHR(SHOpenFolderAndSelectItems(pidlList, (uint) fileList.Length, selectedFileList, 0));
            }
            finally
            {
                ILFree(pidlList);

                foreach (var nint in selectedFileList)
                {
                    ILFree(nint);
                }
            }
        }
    }

    [LibraryImport("Ole32.dll")]
    private static partial int CoInitialize(IntPtr pvReserved, uint dwCoInit);

    [LibraryImport("shell32.dll")]
    private static partial void ILFree(IntPtr pidlList);

    [LibraryImport("shell32.dll", StringMarshalling = StringMarshalling.Utf16)]
    private static partial IntPtr ILCreateFromPathW(string pszPath);

    [LibraryImport("shell32.dll")]
    private static partial int SHOpenFolderAndSelectItems(IntPtr pidlList, uint cild, [MarshalAs(UnmanagedType.LPArray)] IntPtr[] children, uint dwFlags);
```

尝试替换以上代码到项目里，运行项目即可进行测试打开资源管理器选中输入的文件夹的所有文件

那如果需要选中多个文件夹呢？自然只需将以上代码的 `fileList` 替换为文件夹列表就可以了。在 SHOpenFolderAndSelectItems 的 `children` 参数里面可以的内容是传入文件或文件夹。可以混合多选多个文件和文件夹同时

## 参考文档

[c# - How to open Explorer with a specific file selected? - Stack Overflow](https://stackoverflow.com/questions/13680415/how-to-open-explorer-with-a-specific-file-selected )

[file - C#: How to use SHOpenFolderAndSelectItems - Stack Overflow](https://stackoverflow.com/questions/3018002/c-how-to-use-shopenfolderandselectitems )

[c#: 打开文件夹并选中文件 - 楚人无衣 - 博客园](https://www.cnblogs.com/crwy/p/SHOpenFolderAndSelectItems.html )

[SHOpenFolderAndSelectItems 函数 (shlobj_core.h) - Win32 apps - Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/api/shlobj_core/nf-shlobj_core-shopenfolderandselectitems )

[【C#】在Windows资源管理器打开文件夹，并选中指定的文件或文件夹 - Tod's - 博客园](https://www.cnblogs.com/tods/p/17614343.html )

[CoInitializeEx 函数 （combaseapi.h） - Win32 apps Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/api/combaseapi/nf-combaseapi-coinitializeex )

[SHOpenFolderAndSelectItems function (shlobj_core.h) - Win32 apps Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/api/shlobj_core/nf-shlobj_core-shopenfolderandselectitems )

更多 WPF 博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )