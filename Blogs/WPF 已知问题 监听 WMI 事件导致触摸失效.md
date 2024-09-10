本文记录 WPF 的已知问题，在 .NET Core 版本的 WPF 应用里面，应用启动的过程监听 WMI 事件，将导致触摸模块 COM 接口获取失败，进而导致触摸失效

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

此问题仅在 .NET Core 版本复现，在 .NET Framework 框架下能正常工作

复现步骤如下：

1. 安装 System.Management 库用于使用 WqlEventQuery 监听 WMI 变更
2. 监听 TouchDown 事件输出断点信息

复现代码如下

```csharp
    public MainWindow()
    {
        InitializeComponent();

        AppDomain.CurrentDomain.FirstChanceException += (sender, args) =>
        {
            Debug.WriteLine(args.Exception);
        };

        WqlEventQuery insertQuery =
            new WqlEventQuery("SELECT * FROM __InstanceCreationEvent WITHIN 2 WHERE TargetInstance ISA 'Win32_USBHub'");
        ManagementEventWatcher insertWatcher = new ManagementEventWatcher(insertQuery);
        insertWatcher.Start(); // 如果注释掉这句话，那 TouchDown 事件将会被触发

        TouchDown += MainWindow_TouchDown;
    }

    private void MainWindow_TouchDown(object? sender, TouchEventArgs e)
    {
        Debugger.Break(); // 不会被命中
    }
```

加上 `insertWatcher.Start()` 这句代码时，可以从 FirstChanceException 看到如下异常

```
System.InvalidCastException: 没有注册接口


   at MS.Win32.Penimc.UnsafeNativeMethods.CoCreateInstance(Guid& clsid, Object punkOuter, Int32 context, Guid& iid)
```

此问题已经报告给 WPF 官方，请看 <https://github.com/dotnet/wpf/issues/9752>

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/308096e0c8ede019f6dbe5bfe974ae1a12d7de42/WPFDemo/YanerehaylemJeekalhebel) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/308096e0c8ede019f6dbe5bfe974ae1a12d7de42/WPFDemo/YanerehaylemJeekalhebel) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 308096e0c8ede019f6dbe5bfe974ae1a12d7de42
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 308096e0c8ede019f6dbe5bfe974ae1a12d7de42
```

获取代码之后，进入 WPFDemo/YanerehaylemJeekalhebel 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
