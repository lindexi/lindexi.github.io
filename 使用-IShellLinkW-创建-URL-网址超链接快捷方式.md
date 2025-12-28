
# 使用 IShellLinkW 创建 URL 网址超链接快捷方式

本文记录违规地采用 IShellLinkW 创建 URL 网址超链接快捷方式的方法

<!--more-->


<!-- CreateTime:2025/12/27 07:10:26 -->

<!-- 发布 -->
<!-- 博客 -->

开始之前，必须说明的是，在微软文档上明确说明不能用来创建指向 URL 的快捷方式

> This interface cannot be used to create a link to a URL.

详细请看 <https://learn.microsoft.com/en-us/windows/win32/api/shobjidl_core/nn-shobjidl_core-ishelllinkw>

但实际上是可以在 SetPath 里面设置 URL 路径的。尽管微软文档上明确说了不行，但实际上是可以的

为了方便地使用 IShellLinkW 接口，本文使用了 CsWin32 库。此库的使用方法请参阅 [dotnet 使用 CsWin32 库简化 Win32 函数调用逻辑](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-CsWin32-%E5%BA%93%E7%AE%80%E5%8C%96-Win32-%E5%87%BD%E6%95%B0%E8%B0%83%E7%94%A8%E9%80%BB%E8%BE%91.html )

引用之后的 csproj 项目文件的代码大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <PublishAot>true</PublishAot>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Windows.CsWin32" PrivateAssets="all" Version="0.3.264" />
  </ItemGroup>

</Project>
```

添加 NativeMethods.txt 文件，添加如下代码

```csharp
ICustomDestinationList
IObjectCollection
IShellLinkW
IPersistFile
IPropertyStore

PKEY_Title

PropVariantClear
CoCreateInstance
```

添加 `NativeMethods.json` 文件，添加如下代码

```csharp
{
  "$schema": "https://aka.ms/CsWin32.schema.json",
  "allowMarshaling": false
}
```

封装 ShellLinkProvider 类，用来获取 IShellLinkW 对象，代码如下

```csharp
internal static class ShellLinkProvider
{
    public static unsafe IShellLinkW* CreateShellLink()
    {
        IShellLinkW* link = CreateCom<IShellLinkW>(CLSID_IShellLinkW);
        return link;
    }

    private static readonly Guid CLSID_IShellLinkW = new Guid("00021401-0000-0000-C000-000000000046");

    private static unsafe T* CreateCom<T>(in Guid clsid)
        where T : unmanaged
    {
        int hr = PInvoke.CoCreateInstance<T>(in clsid, /* No aggregation */ null, CLSCTX.CLSCTX_INPROC_SERVER, out var ptr);
        Marshal.ThrowExceptionForHR(hr);

        return ptr;
    }
}
```

再封装 ShortcutHelper 类，用来辅助创建快捷方式

```csharp
internal static class ShortcutHelper
{
    /// <summary>
    /// 创建一个快捷方式
    /// </summary>
    /// <param name="lnkFilePath">快捷方式的完全限定路径。</param>
    /// <param name="workDir"></param>
    /// <param name="args">快捷方式启动程序时需要使用的参数。</param>
    /// <param name="targetPath"></param>
    /// <param name="iconFile"></param>
    public static unsafe void CreateShortcut(string lnkFilePath, string targetPath, string workDir, string args = "", string iconFile = "")
    {
        IShellLinkW* shellLinkW = ShellLinkProvider.CreateShellLink();

        shellLinkW->SetPath(targetPath);
        shellLinkW->SetArguments(args);
        shellLinkW->SetWorkingDirectory(workDir);

        shellLinkW->SetIconLocation(iconFile, -1);

        shellLinkW->QueryInterface(out IPersistFile* persistFile);
        persistFile->Save(lnkFilePath, false);
    }
}
```

尝试创建快捷方式，代码如下

```csharp
var shortcutFile = Path.Join(AppContext.BaseDirectory, "1.lnk");

// 尽管文档上明确说明不能用来创建指向 URL 的快捷方式，但实际上是可以的
// > This interface cannot be used to create a link to a URL.
ShortcutHelper.CreateShortcut(shortcutFile, "https://blog.lindexi.com/", Directory.GetCurrentDirectory());
```

经过实际测试，创建出来的是 .lnk 格式的文件，而不是 .url 格式的文件，且创建出来的快捷方式双击可以打开超链接

再次说明，微软官方文档说明了，不应该使用 IShellLinkW 创建 URL 超链接

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/8dc0e09a60e1bf0e5a6d88ab63bfeb64da7b3dde/Workbench/BecearyernaDurfodejefela) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/8dc0e09a60e1bf0e5a6d88ab63bfeb64da7b3dde/Workbench/BecearyernaDurfodejefela) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 8dc0e09a60e1bf0e5a6d88ab63bfeb64da7b3dde
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 8dc0e09a60e1bf0e5a6d88ab63bfeb64da7b3dde
```

获取代码之后，进入 Workbench/BecearyernaDurfodejefela 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。