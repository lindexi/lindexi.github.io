---
title: Avalonia 已知问题 过早创建 App 对象将抛出 PlatformNotSupportedException 异常
description: 本文记录 Avalonia 的一个已知问题，过早创建 App 对象将抛出 PlatformNotSupportedException 异常
tags: Avalonia
category: 
---

<!-- 发布 -->
<!-- 博客 -->

此问题能够在 Avalonia 的 11.3 以及更早版本复现

只需要让 App 对象在 `AppBuilder.Configure` 委托之外创建，那么在调用 StartWithClassicDesktopLifetime 方法时，将在 `Avalonia.Threading.Dispatcher.MainLoop` 方法抛出异常

```
System.PlatformNotSupportedException: Operation is not supported on this platform.
   at Avalonia.Threading.Dispatcher.MainLoop(CancellationToken cancellationToken)
   at Avalonia.Controls.ApplicationLifetimes.ClassicDesktopStyleApplicationLifetime.StartCore(String[] args)
   at Avalonia.Controls.ApplicationLifetimes.ClassicDesktopStyleApplicationLifetime.Start(String[] args)
   at Avalonia.ClassicDesktopStyleApplicationLifetimeExtensions.StartWithClassicDesktopLifetime(AppBuilder builder, String[] args, Action`1 lifetimeBuilder)
   at RemhemlaidejeheWhahaheenalira.Program.Main(String[] args)
```

点进去 `Dispatcher.MainLoop` 看，可以发现进入了 `_controlledImpl == null` 的分支，如以下代码所示

```csharp
	public void MainLoop(CancellationToken cancellationToken)
	{
		if (_controlledImpl == null)
		{
			throw new PlatformNotSupportedException();
		}
		DispatcherFrame frame = new DispatcherFrame();
		cancellationToken.Register(delegate
		{
			frame.Continue = false;
		});
		PushFrame(frame);
	}
```

大概问题原因就是因为 App 过早初始化，导致碰了静态的 `Dispatcher.UIThread` 属性，进而导致了 Dispatcher 过早初始化，导致 `_controlledImpl` 没有被初始化，这才导致出现此异常

```csharp
public partial class Dispatcher : IDispatcher
{
    ...
    public static Dispatcher UIThread
    {
        [MethodImpl(MethodImplOptions.AggressiveInlining)]
        get
        {
            return s_uiThread ??= CreateUIThreadDispatcher();
        }
    }

    ...
}
```

我认为这是 Avalonia 的一个设计问题，但可以按照它现在的框架，也没有什么可以优化

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/266f08d8df93a15051573c88676671d48136aeb0/AvaloniaIDemo/RemhemlaidejeheWhahaheenalira) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/266f08d8df93a15051573c88676671d48136aeb0/AvaloniaIDemo/RemhemlaidejeheWhahaheenalira) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 266f08d8df93a15051573c88676671d48136aeb0
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 266f08d8df93a15051573c88676671d48136aeb0
```

获取代码之后，进入 AvaloniaIDemo/RemhemlaidejeheWhahaheenalira 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

