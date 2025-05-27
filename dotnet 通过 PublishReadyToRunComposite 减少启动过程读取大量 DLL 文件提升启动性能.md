# dotnet 通过 PublishReadyToRunComposite 减少启动过程读取大量 DLL 文件提升启动性能

在 dotnet 应用程序发布的时候，可以选择带上 PublishReadyToRunComposite 选项，带上之后可以将 ReadyToRun 的内容合并到一个入口 DLL 文件里面，可以减少启动过程中访问大量的 DLL 文件导致的 IO 缓慢问题，从而提升启动性能

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

开始之前，先聊聊 dotnet 里面的 [ReadyToRun](https://learn.microsoft.com/en-us/dotnet/core/deploying/ready-to-run) 是在解决什么问题。众所周知，在 dotnet 里面可以走 JIT 实时将 IL 转换为机器码进行运行，在启动过程中，这部分转换损耗会让一些开发者感觉到心疼。此时很多开发者开始寻求 dotnet 的 AOT 技术。开启 AOT 技术可以在从 C# 代码编译的过程里面，直接转换为对应平台的机器码，没有中间的 IL 过程，运行过程中也不需要 JIT 实时翻译。于是开启 AOT 的应用的启动速度，从理论上讲就会快很多。但是 AOT 不是银弹，也会带来问题，比如 AOT 就是完全静态的啦，无法实现一些动态代码调用，比如动态反射、动态代码织入，开发过程中有许多限制等等。那么有没有一个技术能够同时满足这两个条件：一要启动时候足够快，提前在 C# 代码编译的过程里面就生成对应的平台代码，无需启动过程中执行实时翻译；二是要支持动态代码调用，支持编运行边根据设备环境优化代码。对强大的 dotnet 来说，自然是有的啦，那就是本文的主角—— ReadyToRun 技术

使用 ReadyToRun 技术时，将在从 C# 代码编译的过程里面，先尝试将绝大部分的代码编成了平台代码，然后依然保留 IL 代码，用于后续在运行过程中利用 IL 代码进行边运行边优化和支持很多动态代码调用技术

那为什么默认发布的时候，不给咱开 ReadyToRun 方式发布呢？这是因为如上文所述，最终构建出来的产物 DLL 将会包含两部分内容，一个是 IL 代码，另一个就是平台代码（机器码），自然会让 DLL 体积更大了

了解应用软件启动性能优化的伙伴在看到 DLL 体积更大的时候，自然也会想到，许多应用程序启动性能的瓶颈是在 IO 上。更大体积的 DLL 就意味着更长的载入时间

默认情况下的 dotnet 运行时 BCL 各种库都是开了 ReadyToRun 方式发布的，但是自己编写的项目默认是没有开启的。于是 dotnet 运行时的各种 BCL 库就会比自己自然编出来的大了一些些。应用软件启动过程中，将会四处加载库，加载的库越多，就意味着 IO 越多，启动性能越慢

那有没有办法解决这个问题呢？回顾一下咱的启动性能问题，启动过程中，加载大量的小 DLL 文件时，将不能应用硬盘的顺序读写性能。即使是固态非机械硬盘，随机的小文件的读取性能也是打不过一个大文件顺序读取的。基于这个推论，不难可以想到一个综合的解决方案。那就是在进行 PublishReadyToRun 的时候，将所有的 DLL 里面的 ReadyToRun 内容全部合入到一个 DLL 里面。如此即可在启动过程中，尽量只读取这一个大的 DLL 文件，减少对其他离散的 DLL 文件的加载和处理。这项技术就是采用 PublishReadyToRunComposite 的方式发布，其 csproj 项目文件配置代码如下


```xml
    <PublishReadyToRun>true</PublishReadyToRun>
    <PublishReadyToRunComposite>true</PublishReadyToRunComposite>
```

应用了 PublishReadyToRunComposite 技术之后会不会大幅提升体积？不会，因为默认的 dotnet 运行时里面带的 DLL 都是加上了 ReadyToRun 的，带上 PublishReadyToRunComposite 之后，会剥离原本的 DLL 的 ReadyToRun 内容，将其全部功力都集中到一个 `.r2r.dll` 文件里面

如此实现的效果就是发布的时候多带了一个名为 `<入口程序集>.r2r.dll` 的文件，且其他的如 dotnet 运行时里面带的 DLL 文件都变小了，具体解析去看就是这些 DLL 里面的 ReadyToRun 部分都被集中到 `.r2r.dll` 文件里面

于是启动的时候，将会优先加载 `<入口程序集>.r2r.dll` 程序集，加载之后就可以获得绝大部分 IL 代码的已经提前构建的平台代码，直接就可以用上，不需要经过 JIT 的过程。且这是一个大的 DLL 程序集，对磁盘来说可以开森地进行顺序读取。对于杀毒软件来说也能开森的一次性扫描完，但至于杀毒软件开不开森扫描大 DLL 文件，那就看具体的杀毒软件心情啦

实际测试时我发现 PublishReadyToRunComposite 对低至 dotnet 6 的应用程序有不错的效果，对于越是大型的项目，启动过程越是逻辑复杂的项目，其提升越大。为什么说是低至 dotnet 6 呢？因为 .NET Core 3.1 等更底版本我就没有测试了，且根据我的粗略考古，本文介绍的 PublishReadyToRunComposite 技术是从 .NET 6 才开始引入的

之前在 dotnet 6 里面的较低版本里面，还不能很好支持 WPF 程序，应用上 PublishReadyToRunComposite 时就会出现 DWriteFactory 类型加载异常，详细请看 <https://github.com/dotnet/wpf/issues/5316>

好在这个问题在后续的 dotnet SDK 版本已经修复了，需要明确的是 PublishReadyToRunComposite 等技术是更加和 dotnet SDK 相关联的，而不是更多和具体的 dotnet 版本相挂钩的。这就意味着依然可以保持 TargetFramework 版本不变，只是提升打包构建时所用的 dotnet SDK 版本就可以了。我当前使用了 dotnet 9.0.203 版本的 SDK 进行构建 .NET 6 的 WPF 程序，带上 PublishReadyToRunComposite 进行发布之后，没有遇到什么问题。应用程序的启动性能也能得到微弱的提升

是否决定在项目里面使用 PublishReadyToRunComposite 技术，需要自行测试，测试开启和不开启 ReadyToRun 技术时的启动性能对比。进行启动性能对比是比较复杂的，毕竟在 Windows 上还有各种内存缓存逻辑，不小心就全走了内存缓存，影响测试结果。还有杀毒软件在捣乱也会有许多的影响。在开始测试之前，还请自行确保测试环境的稳定性

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/2a8c750c4000470169c95c5ff21c190728fb57ac/WPFDemo/BallweanayheyaylarBaberwhikani) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/2a8c750c4000470169c95c5ff21c190728fb57ac/WPFDemo/BallweanayheyaylarBaberwhikani) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 2a8c750c4000470169c95c5ff21c190728fb57ac
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 2a8c750c4000470169c95c5ff21c190728fb57ac
```

获取代码之后，进入 WPFDemo/BallweanayheyaylarBaberwhikani 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )