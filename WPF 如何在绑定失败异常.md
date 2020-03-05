# WPF 如何在绑定失败异常

在开发 WPF 程序，虽然 xaml 很好用，但是经常会出现小伙伴把绑定写错了。因为默认的 VisualStudio 是没有自动提示，这时很容易复制粘贴写出一个不存在的属性。

在 xaml 如果绑定失败了，那么内部会有一个异常，但是 WPF 不会把这个异常抛出来，这个异常也不会让用户拿到，只是会在输出窗口提示。但是异常会影响性能，而且会让界面和设计的不一样，所以我就想在找到绑定异常就抛出，弹出窗口告诉小伙伴。

本文会告诉大家如何找到绑定失败，并且抛出异常，如何防止修改属性名让xaml绑定失败。

<!--more-->
<!-- CreateTime:2019/11/29 10:13:57 -->

<!-- csdn -->
<!-- 标签：WPF，调试，WPF调试 -->

<div id="toc"></div>

在绑定失败异常建议只在调试下抛出，抛出异常建议弹出，告诉开发者现在你的界面有绑定异常

## 拿到绑定信息

先来写简单的代码，做一个 ViewModel ，里面有两个属性

```csharp
    class ViewModel
    {
        public string Name { get; set; } = "lindexi";

        public string JaslorbafelStojou { get; set; } = "lindexi.gitee.io";
    }
```

可以看到第二个属性是比较复杂的，现在来写 xaml 界面，

```csharp
    <Grid>
        <StackPanel Margin="10,10,10,10" HorizontalAlignment="Center">
            <TextBlock Margin="10,10,10,10" Text="{Binding Name}"></TextBlock>
            <TextBlock Margin="10,10,10,10" Text="{Binding JaslorbafelStoj}"></TextBlock>
        </StackPanel>
    </Grid>
```

然后在后台代码添加这个代码

```csharp
        public MainWindow()
        {
            InitializeComponent();

            DataContext = new ViewModel();
        }
```

现在运行一下，你猜是不是会显示两行，一行是 lindexi 一行是 lindexi.gitee.io ，实际上你看到只有一行，因为第二个绑定写错了

![](http://image.acmx.xyz/lindexi%2F2018517144793266.jpg)

第二个在 ViewModel 的属性是 JaslorbafelStojou 但是 xaml 写的是 JaslorbafelStoj ，如果这时看到了输出，就会看到下面代码

```
System.Windows.Data Error: 40 : BindingExpression path error: 'JaslorbafelStoj' property not found on 'object' ''ViewModel' (HashCode=16468652)'. BindingExpression:Path=JaslorbafelStoj; DataItem='ViewModel' (HashCode=16468652); target element is 'TextBlock' (Name=''); target property is 'Text' (type 'String')
```

那么这个代码是否可以用来判断出现绑定失败，是的，让我来告诉大家如何拿到输出

## 转发绑定

因为绑定失败输出是使用 Trace ，关于 Trace 请看[WPF 调试 获得追踪输出](https://blog.lindexi.com/post/WPF-%E8%B0%83%E8%AF%95-%E8%8E%B7%E5%BE%97%E8%BF%BD%E8%B8%AA%E8%BE%93%E5%87%BA.html )

那么如何拿到 Trace 的输出？

首先需要定义一个类继承 TraceListener ，下面定义一个  BindingErrorTraceListener 收到了消息就输出

```csharp
    
    public class BindingErrorTraceListener : TraceListener
    {
        public override void Write(string message)
        {
            Trace.WriteLine(string.Format("[Write]{0}", message));
        }
 
        public override void WriteLine(string message)
        {
            Trace.WriteLine(string.Format("[WriteLine]{0}", message));
        }
    }
```

然后在构造函数加入，注意在 InitializeComponent 之前

```csharp
        public MainWindow()
        {
            PresentationTraceSources.DataBindingSource.Switch.Level = SourceLevels.Error;
            PresentationTraceSources.DataBindingSource.Listeners.Add(new BindingErrorTraceListener());

            InitializeComponent();

            DataContext = new ViewModel();
        }
```

这时运行代码可以看到输出

```
[Write]System.Windows.Data Error: 40 : 
[WriteLine]BindingExpression path error: 'JaslorbafelStoj' property not found on 'object' ''ViewModel' (HashCode=16468652)'. BindingExpression:Path=JaslorbafelStoj; DataItem='ViewModel' (HashCode=16468652); target element is 'TextBlock' (Name=''); target property is 'Text' (type 'String')
```

所以很容易就知道如何判断是绑定输出

## 绑定失败异常

从上面代码可以知道，所有的绑定输出可以`PresentationTraceSources.DataBindingSource.Listeners`拿到，重写方法就可以转发

而且 TraceListener 是一个很强的类，支持了很多输入，不只字符串，还支持 object ，所以尝试使用 TraceListener 可以做到比较好调试

因为需要在失败抛出异常，就需要定义一个异常

```csharp

public class BindingErrorException : Exception
{
    public string SourceObject { get; set; }
    public string SourceProperty { get; set; }
    public string TargetElement { get; set; }
    public string TargetProperty { get; set; }
 
    public BindingErrorException() 
        : base() 
    { 

    }
 
    public BindingErrorException(string message)
        : base(message) 
    { 

    } 
}

```

判断当前存在绑定失败很简单，主要使用正则判断

```csharp
    public class BindingErrorTraceListener : TraceListener
    {
        private const string BindingErrorPattern = @"^BindingExpression path error(?:.+)'(.+)' property not found(?:.+)object[\s']+(.+?)'(?:.+)target element is '(.+?)'(?:.+)target property is '(.+?)'(?:.+)$";

        public override void Write(string message)
        {
            
        }

        public override void WriteLine(string message)
        {
            var match = Regex.Match(message, BindingErrorPattern);
            if (match.Success)
            {
                var exception = new BindingErrorException(message)
                {
                    SourceObject = match.Groups[2].ToString(),
                    SourceProperty = match.Groups[1].ToString(),
                    TargetElement = match.Groups[3].ToString(),
                    TargetProperty = match.Groups[4].ToString()
                };
                throw exception;
            }

        }
    }

```

这时会发现代码抛出异常

![](http://image.acmx.xyz/lindexi%2F2018517158181899.jpg)

但是抛出了异常建议弹出窗口，这样开发者才会看到

```csharp

     public MainWindow()
        {
            PresentationTraceSources.DataBindingSource.Switch.Level = SourceLevels.Error;
            PresentationTraceSources.DataBindingSource.Listeners.Add(new BindingErrorTraceListener());

            InitializeComponent();

            DataContext = new ViewModel();

            App.Current.DispatcherUnhandledException += DispatcherUnhandledException;
        }

         private void DispatcherUnhandledException(object sender, System.Windows.Threading.DispatcherUnhandledExceptionEventArgs e)
         {
             if (e.Exception is BindingErrorException bindingErrorException)
             {
                 MessageBox.Show($"Binding error. {bindingErrorException.SourceObject}.{bindingErrorException.SourceProperty} {bindingErrorException.TargetElement}.{bindingErrorException.TargetProperty}");
             }
         }
```

![](http://image.acmx.xyz/lindexi%2F20185171510425622.jpg)

## 自动提示

我找到绑定失败很多是因为写错了属性，很多小伙伴不知道实际 xaml 是可以自动提示。

先在 对应的窗口写入绑定的类型，使用`d:DataContext`可以告诉 xaml 使用的数据类型，这样做绑定就可以自动提示

```csharp
    <Grid d:DataContext="{d:DesignInstance local:ViewModel}">
        <StackPanel Margin="10,10,10,10" HorizontalAlignment="Center">
            <TextBlock Text="{Binding Name}"></TextBlock>
            <TextBlock Text="{Binding JaslorbafelStoj}"></TextBlock>
        </StackPanel>
    </Grid>
```

这时尝试删除 JaslorbafelStoj 重新写，就会提示 需要写 JaslorbafelStojou ，这样会自动提示就很难写错。

我很建议大家安装 Resharper 这样在修改变量名时，会自动修改 xaml 的属性名

在有安装 Resharper 的设备，修改一个属性名，然后按 Alt+enter 就会提示 apply rename factoring ，这样会修改所有引用这个属性的变量名

需要注意，必须添加 `d:DataContext` 或者这样设置 ViewModel 才可以通过 Resharper 修改变量名

```csharp
    <Window.DataContext>
        <local:ViewModel />
    </Window.DataContext>
```

如果需要调试 Binding ，参见 [WPF 如何调试 binding](https://lindexi.gitee.io/post/WPF-%E5%A6%82%E4%BD%95%E8%B0%83%E8%AF%95-binding.html )

参见：

[#1,208 – Catching Data Binding Errors Part 1](https://wpf.2000things.com/2017/05/16/1208-catching-data-binding-errors-part-1/#comment-61403 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
