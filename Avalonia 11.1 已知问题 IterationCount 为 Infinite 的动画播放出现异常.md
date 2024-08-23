# Avalonia 11.1 已知问题 IterationCount 为 Infinite 的动画播放出现异常

如果在 Avalonia 后台代码播放一个动画，这个动画的 Animation 的 IterationCount 被设置为 Infinite 那么将在播放的时候抛出 InvalidOperationException 异常

<!--more-->
<!-- CreateTime:2024/08/23 07:17:22 -->

<!-- 发布 -->
<!-- 博客 -->

本文所使用的 Avalonia 为 11.1.0 版本，由于 Avalonia 行为和 API 变动较多，如大家使用其他版本还请重新测试

如以下代码，将会抛出 InvalidOperationException 异常

```csharp
        var content = Content;
        var textBlock = (TextBlock) content!;

        var animation = new Animation()
        {
            Duration = TimeSpan.FromSeconds(10),
            IterationCount = IterationCount.Infinite,
            PlaybackDirection = PlaybackDirection.Alternate,
            Children =
            {
                new KeyFrame()
                {
                    Setters =
                    {
                        new Setter(TranslateTransform.XProperty, 0d),
                    },
                    KeyTime = TimeSpan.FromSeconds(0)
                },
                new KeyFrame()
                {
                    Setters =
                    {
                        new Setter(TranslateTransform.XProperty, 500d),
                    },
                    KeyTime = TimeSpan.FromSeconds(10)
                }
            }
        };

        textBlock.RenderTransform = new TranslateTransform();

        await animation.RunAsync(textBlock);
```

以上代码是从 [Avalonia 后台代码简单播放动画示例](https://blog.lindexi.com/post/Avalonia-%E5%90%8E%E5%8F%B0%E4%BB%A3%E7%A0%81%E7%AE%80%E5%8D%95%E6%92%AD%E6%94%BE%E5%8A%A8%E7%94%BB%E7%A4%BA%E4%BE%8B.html ) 修改的
<!-- [Avalonia 后台代码简单播放动画示例 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18368582 ) -->

运行以上代码，将可以看到如下错误信息

```
System.InvalidOperationException:“Looping animations must not use the Run method.”
```

经过阅读 Avalonia 的代码，我理解了 Avalonia 的意图。在 Avalonia 里面，认为如果一个动画是无限播放的，那就不应该使用 RunAsync 等待，防止等待的逻辑永不返回

然而 Avalonia 却允许调用 RunAsync 方法播放动画，换句话说就是只要使用 RunAsync 而不进行等待就不会炸掉

只需将 `await animation.RunAsync(textBlock);` 修改为 `_ = animation.RunAsync(textBlock);` 即可

如果即可运行一个无限播放的动画

我认为这是 Avalonia 的设计问题，但是不太确定，于是就在 Avalonia 提了一个讨论

详细请看 <https://github.com/AvaloniaUI/Avalonia/discussions/16757>

为什么不等待 RunAsync 也可以？这是因为在 RunAsync 里面是采用 TaskCompletionSource 作为等待机制，判断如果是无限循环的动画，则设置 TaskCompletionSource 异常而已

```csharp
            var run = new TaskCompletionSource<object?>();

            if (this.IterationCount == IterationCount.Infinite)
                run.SetException(new InvalidOperationException("Looping animations must not use the Run method."));
```

这就意味着在不等待 RunAsync 时，将只是 TaskCompletionSource 被设置异常状态，不会真的出现任何异常。也就是尽管 InvalidOperationException 被 new 了，但是没有地方抛出

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/2b7efe38fb648e446b13d9449c92ae5bd4e7cd9a/AvaloniaIDemo/LicajearyaWenewernichiji) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/2b7efe38fb648e446b13d9449c92ae5bd4e7cd9a/AvaloniaIDemo/LicajearyaWenewernichiji) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 2b7efe38fb648e446b13d9449c92ae5bd4e7cd9a
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 2b7efe38fb648e446b13d9449c92ae5bd4e7cd9a
```

获取代码之后，进入 AvaloniaIDemo/LicajearyaWenewernichiji 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )