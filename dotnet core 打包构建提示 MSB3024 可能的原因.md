# dotnet core 打包构建提示 MSB3024 可能的原因

如果是在 Linux 下发布，可能是因为发布的可执行文件和文件夹重名

<!--more-->
<!-- 发布 -->

在使用 `dotnet publish -c release` 在 Linux 服务器发布或使用 `-r linux-x64` 发布看到下面代码

```csharp
_CopyOutOfDateSourceItemsToOutputDirectory:

正在部分生成目标“_CopyOutOfDateSourceItemsToOutputDirectory”，因为某些输出文件相对于其输入文件而言已经过期。

2>C:\Program Files\dotnet\sdk\3.1.102\Microsoft.Common.CurrentVersion.targets(4570,5): error MSB3024: 未能将文件“f:\lindexi\foo\obj\Release\netcoreapp3.1\linux-x64\Foo”复制到目标文件“bin\Release\netcoreapp3.1\linux-x64\Foo”，因为该目标是文件夹而不是文件 。若要将源文件复制到文件夹中，请考虑使用 DestinationFolder 参数，而不使用 DestinationFiles
```

可能的原因是存在文件夹和可执行文件相同。为什么在 Windows 下没有问题，原因是在 Windows 下的可执行文件是带后缀名的，而在 Linux 下是不带后缀名的。如果此时有文件夹和可执行文件重名，如下面的代码

```csharp
│  KalllaijawwaiKemjaniqemchelye.csproj
│  Program.cs
│
└─KalllaijawwaiKemjaniqemchelye
        1.png
```

可以看到 KalllaijawwaiKemjaniqemchelye 文件夹将会和创建的可执行文件 KalllaijawwaiKemjaniqemchelye 重名，此时将会提示 error MSB3024 代码

解决方法是要么更改文件夹名要么更改可执行文件名

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/2b4a8ca4ff0e95f51c10c33cbab4a89037e6010e/KalllaijawwaiKemjaniqemchelye) 欢迎小伙伴访问


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
