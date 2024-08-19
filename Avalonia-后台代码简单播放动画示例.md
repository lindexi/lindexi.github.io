
# Avalonia 后台代码简单播放动画示例

本文将演示如何在 Avalonia 的后台代码里面创建 Animation 执行播放

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

本文演示的内容是将界面里面的一个 TextBlock 控件，通过修改控件的 RenderTransform 的 TranslateTransform 执行平移

为了方便演示，先在 MainView.axaml 里面添加一个 TextBlock 控件，如下面代码。大家可以在本文末尾找到本文所有的代码的下载方法

```xml
  <TextBlock Text="{Binding Greeting}" HorizontalAlignment="Center" VerticalAlignment="Center">
      <TextBlock.RenderTransform>
          <TranslateTransform X="700"/>
      </TextBlock.RenderTransform>
  </TextBlock>
```

接下来演示如何在后台代码里面创建动画和播放动画

演示的代码放在 Loaded 事件里面，强行使用 Task.Delay 模拟业务的延迟。当然了，如果只是动画本身期望延迟，可以使用 Animation 的 Delay 属性执行延迟

```csharp
public partial class MainView : UserControl
{
    public MainView()
    {
        InitializeComponent();

        Loaded += MainView_Loaded;
    }

    private async void MainView_Loaded(object? sender, Avalonia.Interactivity.RoutedEventArgs e)
    {
        // 这里的延迟换成 Animation 的 Delay 也对的，且换成 Animation 的更好。这里的延迟非必须
        await Task.Delay(100);

        ... // 忽略其他代码
    }
```

先创建 Animation 对象，设置动画的过程时间，如下面代码所示

```csharp
        var animation = new Animation()
        {
            Duration = TimeSpan.FromSeconds(10),
            ... // 忽略其他代码
        };
```

接着添加核心的逻辑，通过关键帧动画，设置 TranslateTransform 的 XProperty 在第 0 秒的时候从 0 开始，在第 10 秒的时候为 500d 的值。如此即可作出平缓的动画，实现 TranslateTransform 的 X 从 0 移动到 500 的值，且过程动画用 10 秒

```csharp
        var animation = new Animation()
        {
            Duration = TimeSpan.FromSeconds(10),
            IterationCount = new IterationCount(5),
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
```

这里需要小心一个细节，那就是 XProperty 是定义了 double 类型的属性，要求传入的参数也是 double 类型的。如果这里没有写 0d 而是写 0 将会在后续播放动画步骤啥都没有发生。我认为这里是 Avalonia 的一个设计缺陷，应该在框架层做好转换类型逻辑

完成动画定义之后，现在动画还没附加在某个控件进行播放，咱就从用户控件里面获取刚才界面定义的控件，例子代码如下

```csharp
        var content = Content;
        var textBlock = (TextBlock)content!;
```

拿到控件之后，使用 Animation 的 RunAsync 方法进行播放，这个方法可以不等待返回值。只有在期望等待到动画播放结束的时候，才需要等待返回值的 Task 完成

在 Avalonia 里面存在一个设计缺陷是没有提供和 WPF 一样的故事板，如果有多个控件同时播放动画只好多次调用 RunAsync 方法，且多次调用之间不要加上 await 等待

```csharp
        _ = animation.RunAsync(textBlock);
```

全部的后台创建和播放动画代码如下

```csharp
        var content = Content;
        var textBlock = (TextBlock)content!;

        var animation = new Animation()
        {
            Duration = TimeSpan.FromSeconds(10),
            IterationCount = new IterationCount(5),
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

        _ = animation.RunAsync(textBlock);
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/77ef3ad873b021c71c80ca08cfbff5cadda5e3fc/AvaloniaIDemo/QarnahedajaceYawemcehem) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/77ef3ad873b021c71c80ca08cfbff5cadda5e3fc/AvaloniaIDemo/QarnahedajaceYawemcehem) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 77ef3ad873b021c71c80ca08cfbff5cadda5e3fc
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 77ef3ad873b021c71c80ca08cfbff5cadda5e3fc
```

获取代码之后，进入 AvaloniaIDemo/QarnahedajaceYawemcehem 文件夹，即可获取到源代码

更多 AvaloniaI 博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。