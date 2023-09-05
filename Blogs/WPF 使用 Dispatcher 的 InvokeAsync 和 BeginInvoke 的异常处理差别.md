一般认为 WPF 的 Dispatcher 的 InvokeAsync 方法是 BeginInvoke 方法的平替方法和升级版，接近在任何情况下都应该在业务层使用 InvokeAsync 方法代替 BeginInvoke 方法。然而在异常的处理上，这两个方法还是有细微的差别的，不能说是坏事，依然可以认为使用 InvokeAsync 方法代替 BeginInvoke 方法是正确的。本文将记录这两个在抛出异常时，进入的统一异常处理事件的差别

<!--more-->


<!-- CreateTime:2023/6/14 16:49:04 -->

<!-- 博客 -->
<!-- 发布 -->

简单来说是在 InvokeAsync 抛出未捕获的异常，将会进入到 TaskScheduler.UnobservedTaskException 事件里面。在 BeginInvoke 抛出未捕获的异常，将会进入到 Dispatcher.UnhandledException 事件里面

根据通用的 dotnet 知识可以知道，进入到 TaskScheduler.UnobservedTaskException 的异常，在 .NET Framework 4.5 之后，包含 dotnet core 和 dotnet 5 和 dotnet 6 以及更高版本，是不会导致应用程序退出进程

根据通用的 WPF 知识可以知道，进入到 Dispatcher.UnhandledException 的异常，取决于参数的 Handled 属性是否被设置为 true 值，决定是否将异常抛到线程顶层从而可能导致应用程序退出进程

通过此可以了解到，使用 InvokeAsync 和 BeginInvoke 所抛出的未捕获异常所进入的事件不相同。这里值得说明的是，无论是 InvokeAsync 或 BeginInvoke 方法，都没有使用其返回值。进一步的说明就是不对 InvokeAsync 使用 await 等待的前提下，表现行为如本文描述。本文开始的说法是严谨的，因为对 InvokeAsync 使用 await 等待，则将 InvokeAsync 异常交给 await 这一端，然后取决于等待的逻辑的异常处理，此时和 InvokeAsync 行为无关

有一些不符合我开始预期的是 InvokeAsync 抛出未捕获的异常，将会进入到 TaskScheduler.UnobservedTaskException 事件里面。这是因为 InvokeAsync 走的是 Task 封装。在 dotnet 里面，如果 Task 里存在异常，且此 Task 没有任何的 await 将会在此 Task 被回收清理时，将异常记录到 TaskScheduler.UnobservedTaskException 事件

接下来是对此行为的测试代码

新建一个 WPF 项目，编写简单的界面，加上两个按钮，这两个按钮用来分别调用 InvokeAsync 和 BeginInvoke 抛出异常

```xml
<Window x:Class="GifellichelNurcikaifallhane.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:GifellichelNurcikaifallhane"
        mc:Ignorable="d"
        Title="MainWindow" Height="450" Width="800">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition></RowDefinition>
            <RowDefinition></RowDefinition>
        </Grid.RowDefinitions>
        <TextBlock x:Name="TextBlock" Margin="10,10,10,10" 
                   TextWrapping="Wrap"/>
        <StackPanel Grid.Row="1" Orientation="Horizontal" VerticalAlignment="Top">
            <Button x:Name="InvokeAsyncButton" Margin="10,10,10,10"
                    Click="InvokeAsyncButton_OnClick">InvokeAsync</Button>
            <Button x:Name="BeginInvokeButton" Margin="10,10,10,10"
                    Click="BeginInvokeButton_OnClick">BeginInvoke</Button>
        </StackPanel>
    </Grid>
</Window>
```

在 MainWindow 的构造函数里面，分别添加两个异常捕获事件，用来进行输出。同时跑一个任务不断执行垃圾回收

```csharp
    public MainWindow()
    {
        InitializeComponent();

        Dispatcher.UnhandledException += (sender, args) =>
        {
            args.Handled = true;
            TextBlock.Text += $"Dispatcher UnhandledException {args.Exception.Message}\r\n";
        };

        TaskScheduler.UnobservedTaskException += (sender, args) =>
        {
            Dispatcher.InvokeAsync(() =>
            {
                TextBlock.Text += $"TaskScheduler UnobservedTaskException {args.Exception.InnerException!.Message}\r\n";
            });
        };

        Task.Run(async () =>
        {
            while (true)
            {
                // 不断 GC 方便 Task 清理
                await Task.Delay(TimeSpan.FromSeconds(1));
                GC.Collect();
            }
        });
    }
```

以上代码里面，因为 TaskScheduler 的 UnobservedTaskException 不是在主线程调度的，需要使用 Dispatcher 才能让内容输出在界面

接下来编写两个按钮的代码

```csharp
    private void InvokeAsyncButton_OnClick(object sender, RoutedEventArgs e)
    {
        Dispatcher.InvokeAsync(() => throw new Exception($"在 Dispatcher.InvokeAsync 抛出异常"));
    }

    private void BeginInvokeButton_OnClick(object sender, RoutedEventArgs e)
    {
        Dispatcher.BeginInvoke(new Action(() => throw new Exception($"在 Dispatcher.BeginInvoke 抛出异常")));
    }
```

这里需要特别说明的是，咱是不应该抛出 Exception 类型的异常的，正确的做法是抛出特别类型的异常，例如 ArgumentException 等类型的异常。以上的代码仅用来进行测试行为

运行以上代码，分别点击两个按钮，可以看到有不同的输出，从而可以了解到这两个方法的异常处理行为


本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/a7cbc4bd5e0ec41be5d0be719fa387adfb6bf52e/GifellichelNurcikaifallhane) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/a7cbc4bd5e0ec41be5d0be719fa387adfb6bf52e/GifellichelNurcikaifallhane) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin a7cbc4bd5e0ec41be5d0be719fa387adfb6bf52e
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin a7cbc4bd5e0ec41be5d0be719fa387adfb6bf52e
```

获取代码之后，进入 GifellichelNurcikaifallhane 文件夹
