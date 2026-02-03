
# Win32 使用 MoveFileEx 延迟到重启后删除文件

某些文件由于当前进程还在占用中，无法立刻删除，可通过 KERNEL32 提供的 MoveFileEx 方法延迟到下次开机启动时删除文件

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

在某些情况下，比如应用软件卸载等情况，由于当前进程还在占用着文件，无法立刻删除。或担心立刻删除文件时，导致一些异常情况。需要延迟文件删除动作到下次开机启动之后。此时可使用 MoveFileExW 方法执行延迟删除

本文将使用 CsWin32 库辅助对 MoveFileEx 的调用，详细请看 [dotnet 使用 CsWin32 库简化 Win32 函数调用逻辑](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-CsWin32-%E5%BA%93%E7%AE%80%E5%8C%96-Win32-%E5%87%BD%E6%95%B0%E8%B0%83%E7%94%A8%E9%80%BB%E8%BE%91.html )

先安装好 CsWin32 库，安装之后的 csproj 项目文件代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <PublishAot>true</PublishAot>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Windows.CsWin32" Version="0.3.269">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
    </PackageReference>
  </ItemGroup>

</Project>
```

添加 NativeMethods.txt 文件，在文件中添加如下代码

```
MoveFileExW
```

按照 <https://learn.microsoft.com/zh-cn/windows/win32/api/winbase/nf-winbase-movefileexw> 官方文档说明，首个参数传入要删除的文件的路径，第二个参数传入 NULL 空，第三个参数传入 MOVEFILE_DELAY_UNTIL_REBOOT 即可实现下次开机时删除文件

示例代码如下

```csharp
// See https://aka.ms/new-console-template for more information

using Windows.Win32;
using Windows.Win32.Storage.FileSystem;

if(args.Length == 0)
{
    Console.WriteLine($"请输入要删除的文件的路径");
}

var file = args[0];
file = Path.GetFullPath(file);

if (!File.Exists(file))
{
    Console.WriteLine($"找不到 '{file}' 文件"); // 可能为文件夹。文件夹也是可以调用 MoveFileEx 进行删除的
}

PInvoke.MoveFileEx(file, null, MOVE_FILE_FLAGS.MOVEFILE_DELAY_UNTIL_REBOOT);

Console.WriteLine($"重启后删除 '{file}' 文件");
```

注：

1. 调用 MoveFileEx 方法时，要求采用管理员权限（官方文档：仅当进程位于属于管理员组或 LocalSystem 帐户的用户的上下文中时，才能使用此值）。正常应用比较难获取管理员权限，但是对于安装包卸载器程序来说，一般都是采用管理员权限运行，自然也就能方便地使用

2. 加入重启后删除的文件或文件夹，将被写入 `计算机\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\PendingFileRenameOperations` 注册表项里

3. 可用 MoveFileEx 方法删除文件夹，但是要求文件夹是空文件夹，即文件夹内不能再有文件或文件夹。如果要删除有文件的文件夹，可先遍历文件夹内所有文件，先将文件逐一调用 MoveFileEx 方法删除，然后再调用 MoveFileEx 删除文件夹。调用顺序会按照先调用先删除，只需确保在删除到文件夹的时候，文件夹已是空文件夹即可

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/8dd402bf07229f4b189853f46416e3b5d7f6596d/Workbench/DedaykarkiLiwhileeno) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/8dd402bf07229f4b189853f46416e3b5d7f6596d/Workbench/DedaykarkiLiwhileeno) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 8dd402bf07229f4b189853f46416e3b5d7f6596d
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 8dd402bf07229f4b189853f46416e3b5d7f6596d
```

获取代码之后，进入 Workbench/DedaykarkiLiwhileeno 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。