# dotnet 修复在 Linux 上使用 SkiaSharp 提示找不到 liblibSkiaSharp 库

本文告诉大家如何简单修复在 Linux 上使用 SkiaSharp 提示找不到 liblibSkiaSharp 库

<!--more-->
<!-- CreateTime:2022/6/27 15:30:46 -->

<!-- 发布 -->
<!-- 博客 -->

我的应用在 Windows 上跑的好好的，放在 Linux 上一运行就炸掉了，异常内容如下

```
Unhandled exception. System.TypeInitializationException: The type initializer for 'SkiaSharp.SKColorSpace' threw an exception.
 ---> System.DllNotFoundException: Unable to load shared library 'libSkiaSharp' or one of its dependencies. In order to help diagnose loading problems, consider setting the LD_DEBUG environment variable: liblibSkiaSharp: cannot open shared object file: No such file or directory
   at SkiaSharp.SkiaApi.sk_colorspace_new_srgb()
   at SkiaSharp.SKColorSpace..cctor()
   --- End of inner exception stack trace ---
   at SkiaSharp.SKColorSpace.CreateSrgb()
   at Program.<Main>$(String[] args) in d:\lindexi\Code\SkiaSharp\SkiaSharp\KebeninegeeWaljelluhi\KebeninegeeWaljelluhi\Program.cs:line 5
```

原因是 Linux 的版本众多，大家都很喜欢自己定义，这让 SkiaSharp 不知道包含哪个版本才是能让大家都开心的，如[官方文档](https://github.com/mono/SkiaSharp/wiki/SkiaSharp-Native-Assets-for-Linux) 所讲的故事

解决的方法是再安装上 `SkiaSharp.NativeAssets.Linux` 或 `SkiaSharp.NativeAssets.Linux.NoDependencies` 库即可

如在 csproj 上添加以下代码用来安装

```xml
  <ItemGroup>
    <PackageReference Include="SkiaSharp" Version="2.88.0" />
    <PackageReference Include="SkiaSharp.NativeAssets.Linux.NoDependencies" Version="2.88.0" />
  </ItemGroup>
```

以上的方法经过我在 WSL 的 Ubuntu 上测试

本文的例子放在[github](https://github.com/lindexi/lindexi_gd/tree/668b2acf5749a1e190733882ae49ad105877cb55/SkiaSharp/KebeninegeeWaljelluhi) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/668b2acf5749a1e190733882ae49ad105877cb55/SkiaSharp/KebeninegeeWaljelluhi) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 668b2acf5749a1e190733882ae49ad105877cb55
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 668b2acf5749a1e190733882ae49ad105877cb55
```

获取代码之后，进入 SkiaSharp/KebeninegeeWaljelluhi 文件夹

我建立了一个 SkiaSharp 的群： 788018852 欢迎大家加入讨论

更多请看 [Docker环境下使用SkiaSharp的2种方式 - 从零开始-DotNET技术 - 博客园](https://www.cnblogs.com/yycelsu/p/14048859.html )