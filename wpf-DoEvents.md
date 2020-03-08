# wpf DoEvents 

如果在执行一段卡UI的代码，这时如何让UI响应。如果存在代码需要获得依赖属性，那么代码就需要在UI线程执行，但是这时就会卡UI，为了让UI响应，所以就需要使用`DoEvents`来让UI响应。

首先需要知道，`DoEvents`是在 WinForm 有的，在 WPF 没有这个函数，但是可以自己写出来。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:51 -->

<div id="toc"></div>

<!-- csdn -->

<!-- 标签: wpf,doevents,性能优化 -->

先做一个例子让大家知道`DoEvents`的作用，使用的呆磨很简单，请看代码

```csharp
<Window x:Class="ZuindmMbx.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:ZuindmMbx"
        mc:Ignorable="d"
        Title="MainWindow" Height="350" Width="525">
    <Grid>
        <ListView ItemsSource="{Binding KatudefZubpobryk}">
            <ListView.ItemTemplate>
                <DataTemplate>
                    <TextBlock Text="{Binding}"></TextBlock>
                </DataTemplate>
            </ListView.ItemTemplate>
        </ListView>
        <Button Content="确定" HorizontalAlignment="Left" Margin="424,292,0,0" VerticalAlignment="Top" Width="75" Click="Button_OnClick"/>
    </Grid>
</Window>

   public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
            DataContext = this;
        }

        public ObservableCollection<string> KatudefZubpobryk { get; set; } = new ObservableCollection<string>();

        private void Button_OnClick(object sender, RoutedEventArgs e)
        {
            for (int i = 0; i < 10; i++)
            {
                Foo(10);
                KatudefZubpobryk.Add(i.ToString());
            }
        }

        private void Foo(int n)
        {
            for (int i = 0; i < n; i++)
            {
                Foo(n - 1);
            }
        }
    }
```

这时点击确定可以看到，需要等待一些时间才可以响应界面

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F%25E6%2580%25A7%25E8%2583%25BD1.gif)

如果加上了 DoEvents 就可以看到下图的效果

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F%25E6%2580%25A7%25E8%2583%25BD2.gif)


## 用法

在呆磨的程序做一些修改，请看代码

```csharp
        private void Button_OnClick(object sender, RoutedEventArgs e)
        {
            for (int i = 0; i < 10; i++)
            {
                Foo(10);
                KatudefZubpobryk.Add(i.ToString());
                DoEvents();
            }
        }

        public static void DoEvents()
        {
            DispatcherFrame frame = new DispatcherFrame();
            Dispatcher.CurrentDispatcher.BeginInvoke(DispatcherPriority.Background, new DispatcherOperationCallback(ExitFrame), frame);
            Dispatcher.PushFrame(frame);
        }

        private static Object ExitFrame(Object state)
        {
            ((DispatcherFrame) state).Continue = false;
            return null;
        }
```

所以只需要在循环加上代码就可以了。可以复制下面的两个方法到需要使用让UI响应的地方，在需要的地方调用，使用的方法很简单。

建议在下面的地方使用：

 - 后台操作比较耗时，未完全加载也能正常使用
 - 性能已经没有办法优化
 - 性能没有时间优化，可作为临时性方案
 - DoEvents建议一定是在主线程上使用

## 原理

请看一下底层的`PushFrameImpl` 下面的代码有删减

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F%25E6%2580%25A7%25E8%2583%25BD120171017105233.jpg)

会导致UI重绘的消息：0xC25A及0xC262 所以发送这个消息就可以让UI响应

## 存在的坑

这里的坑是 PushFrame 的坑，关于他的原理，请看 [https://walterlv.github.io/post/dotnet/2017/09/26/dispatcher-push-frame.html](https://walterlv.github.io/post/dotnet/2017/09/26/dispatcher-push-frame.html)

如果点击确定按钮之后，再次点击确定按钮，那么就会出现很多个重复的数。如果使用这个方法，那么需要禁用确定按钮，小心用户多次点击。

在使用方法的时候拖动窗口，可能让窗口卡死。

复现步骤：

修改上面呆磨代码，加上`OnLoaded`，里面使用`Dispatcher.Invoke`或`DoEvents`，然后运行拖动窗口，这时窗口卡死 

```csharp
        public MainWindow()
        {
            InitializeComponent();
            DataContext = this;
            Loaded += OnLoaded;
        }

        private async void OnLoaded(object sender, RoutedEventArgs e)
        {
            await Task.Delay(2000);
            Dispatcher.Invoke(() => { }, DispatcherPriority.Background);
        }
```

但是这时使用 Alt+Tab 到其他窗口，然后回来，可以看到窗口正常

实际上尝试改变窗口大小也会让窗口卡死，请看[WPF application intermittently hangs when using Dispatcher.Invoke and/or Dispatcher.PushFrame while user is resizing or draging window ](https://connect.microsoft.com/VisualStudio/feedback/details/807292/wpf-application-intermittently-hangs-when-using-dispatcher-invoke-and-or-dispatcher-pushframe-while-user-is-resizing-or-draging-window )

### OnLoad 上其他坑

我必须说，不仅是 OnLoad 会出现这些坑，在很多情况也会，但是我还不知道条件。

请把`await Task.Delay(2000)`换为`Foo(10);`进行一些计算，这时在软件启动的时候，尝试拖动窗口，可以看到窗口是没有显示内容，但是鼠标放开的时候，就可以看到界面显示。

```csharp
        private void OnLoaded(object sender, RoutedEventArgs e)
        {
            Foo(10);

            Dispatcher.Invoke(() =>
            {
            }, DispatcherPriority.Background);
        }
```

接着把`Invoke`换为`DoEvents`，结果相同，在启动拖动窗口，窗口没有内容。

### 使用 DispatcherTimer 出现窗口冻结

下面的代码是创建一个 time 不停在里面使用`Dispatcher.Invoke`

```csharp
        public MainWindow()
        {
            InitializeComponent();
            DataContext = this;
            Loaded += OnLoaded;

            DispatcherTimer time = new DispatcherTimer();

            time.Interval = new TimeSpan(0, 0, 1);
            time.Tick += Time_Tick;
            time.Start();
        }

        private void Time_Tick(object sender, EventArgs e)
        {
            Foo(10);
            Dispatcher.Invoke(() => { }, DispatcherPriority.Background);
        }
```

这时拖动窗口会出现冻结，和上面一样。

实际把上面代码的运算去掉也会冻住，但是我尝试10次，有2次在放开的时候才冻住。

## 推荐方法

实际上垃圾wr是不是要让开发者去写这样的方法？实际上垃圾wr已经做了这个东西，但是没有直接告诉开发者，请尝试使用下面的代码代替上面呆磨

```csharp
        private void Button_OnClick(object sender, RoutedEventArgs e)
        {
            for (int i = 0; i < 10; i++)
            {
                Foo(10);
                KatudefZubpobryk.Add(i.ToString());
                Dispatcher.Invoke(() => { }, DispatcherPriority.Background);
            }
        }
```

关键就是`Dispatcher.Invoke(() => { }, DispatcherPriority.Background);`，这句代码就是在主线程插入一个`Background` 因为优先级，所以这时就可以让UI处理其他的输入

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F%25E6%2580%25A7%25E8%2583%25BD120171017105655.jpg)

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F%25E6%2580%25A7%25E8%2583%25BD3.gif)

但是直接使用`Dispatcher.Invoke`代码太长，是不是可以使用比较简单的？实际上还是有的，请看代码。

```csharp
        private async void Button_OnClick(object sender, RoutedEventArgs e)
        {
            for (int i = 0; i < 10; i++)
            {
                Foo(10);
                KatudefZubpobryk.Add(i.ToString());
                await System.Windows.Threading.Dispatcher.Yield();
            }
        }
```

实际上`System.Windows.Threading.Dispatcher.Yield`这个方法的实现和`Dispatcher.Invoke(() => { }, DispatcherPriority.Background`一点也不同，他使用的是 async 以及其他我还不知道怎么说的科技。

最后的方法是在UI主线程执行的函数上添加`async`和直接使用`Dispatcher.Yield`就可以在循环中让UI响应。不会在循环中让UI卡住。

建议使用最后的方法，因为这个方法可以解决坑，而且使用简单

实际上，使用了上面无论哪个方法都不会让界面一直都响应，如果页面有一个循环的动画，就可以看到动画播放实际上有些卡，下面写一个呆磨就可以知道。在上面的界面添加下面的代码，不停做动画。

```csharp
        <Grid>
            <Grid.Triggers>

                <EventTrigger RoutedEvent="Grid.Loaded">

                    <BeginStoryboard>

                        <Storyboard RepeatBehavior="Forever">
                            <DoubleAnimation Storyboard.TargetName="T" Storyboard.TargetProperty="Angle" From="0" To="360" Duration="0:0:1"></DoubleAnimation>
                        </Storyboard>
                    </BeginStoryboard>
                </EventTrigger>
            </Grid.Triggers>
            <Grid x:Name="G" Background="#565656" Width="200" Height="200" 
                  HorizontalAlignment="Center" VerticalAlignment="Center">
                <Grid.RenderTransform>
                    <RotateTransform x:Name="T" CenterX="100" CenterY="100" Angle="0"></RotateTransform>
                </Grid.RenderTransform>
            </Grid>
        </Grid>
```

这时点击按钮，可以看到动画有些卡，点击窗口拖动就可以看到动画正常。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。