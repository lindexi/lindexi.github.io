# wpf DoEvents 

如果在执行一段卡UI的代码，这时如何让UI响应。如果存在代码需要获得依赖属性，那么代码就需要在UI线程执行，但是这时就会卡UI，为了让UI响应，所以就需要使用`DoEvents`来让UI响应。

首先需要知道，`DoEvents`是在 WinForm 有的，在 WPF 没有这个函数，但是可以自己写出来。

<!--more-->
<div id="toc"></div>

<!-- csdn -->

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

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F%25E6%2580%25A7%25E8%2583%25BD1.gif)

如果加上了 DoEvents 就可以看到下图的效果

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F%25E6%2580%25A7%25E8%2583%25BD2.gif)


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

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F%25E6%2580%25A7%25E8%2583%25BD120171017105233.jpg)

会导致UI重绘的消息：0xC25A及0xC262 所以发送这个消息就可以让UI响应

## 存在的坑

如果点击确定按钮之后，再次点击确定按钮，那么就会出现很多个重复的数。如果使用这个方法，那么需要禁用确定按钮，小心用户多次点击。

在使用方法的时候拖动窗口，可能让窗口卡死。

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

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F%25E6%2580%25A7%25E8%2583%25BD120171017105655.jpg)

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F%25E6%2580%25A7%25E8%2583%25BD3.gif)

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

需要添加`async`和直接使用`Dispatcher.Yield`

建议使用最后的方法，因为这个方法可以解决坑，而且使用简单

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。