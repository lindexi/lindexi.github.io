# WPF 绑定命令在 MVVM 的 CanExecute 和 Execute 在按钮点击都没触发可能的原因

在 WPF 推荐使用 MVVM 绑定命令，但是绑定命令会存在很多坑，其中一个就是焦点的问题。如果在用户点击按钮的时候出现了焦点修改，那么此时的命令是不会被触发

<!--more-->
<!-- CreateTime:2019/11/29 8:48:48 -->

<!-- csdn -->

在命令绑定按钮点击的时候，会触发按钮拿到键盘焦点，此时其他元素如果之前有拿到焦点，那么会触发元素失去焦点。如果在元素一次 Dispatcher 的过程重新拿到焦点，那么按钮的命令将不会被触发

说起来复杂，因为在项目的代码是很复杂很难直接看到这个问题，所以我建议创建一个新的 WPF 项目，不要引用任何小伙伴框架，简单定义一些类就可以看到这个坑

定义一个简单的命令

```csharp
    public class Command : ICommand
    {
        /// <inheritdoc />
        public bool CanExecute(object parameter)
        {
            return true;
        }

        /// <inheritdoc />
        public void Execute(object parameter)
        {
            Debug.WriteLine("林德熙是逗比");
        }

        /// <inheritdoc />
        public event EventHandler CanExecuteChanged;
    }
```

定义一个简单的 ViewModel 里面只有命令

```csharp
    public class ViewModel
    {
        public ICommand Command { get; } = new Command();
    }
```

在界面绑定 ViewModel 请看代码

```csharp
        public MainWindow()
        {
            InitializeComponent();
            DataContext = ViewModel;
        }

        public ViewModel ViewModel { get; } = new ViewModel();
```

如何绑定 ViewModel 请看 [win10 uwp DataContext](https://blog.lindexi.com/post/win10-uwp-DataContext.html )

在界面放一个文本和一个按钮，文本可以在失去焦点的时候重新拿到焦点

```csharp
        <StackPanel Margin="10,10,10,10">
            <TextBox LostFocus="TextBox_OnLostFocus"></TextBox>
            <Button Margin="10,10,10,10" Content="确定" Command="{Binding Command}"></Button>
        </StackPanel>
```

后台代码的失去焦点需要通过在一次 Dispatcher 里面写，不然将会出现有趣的坑，具体是什么坑，可以下载我的[源代码](https://github.com/lindexi/lindexi_gd/blob/d8200f16691754cc61c75ecfcc7d426228a7bc55/NewhawhebichaJalciceerulebaiwair/NewhawhebichaJalciceerulebaiwair/MainWindow.xaml.cs)自己修改一下

请看后台代码

```csharp
        private void TextBox_OnLostFocus(object sender, RoutedEventArgs e)
        {
            Dispatcher.InvokeAsync(((UIElement) sender).Focus);
        }
```

此时运行代码，点击文本，可以看到输出窗口输出 林德熙是逗比 然后点击文本，输入文字，然后点击按钮，可以发现按钮的命令没有触发

在命令的 CanExecute 打上断点，可以发现连 CanExecute 都没有进入

如果遇到了在按钮 MVVM 绑定命令，发现命令没有触发，同时 CanExecute 都没有进入，可以猜可能是命令没有初始化、命令没有绑对，还有可能是在过程出现焦点问题

另外不一定是用户直接调用 Focus 其他的 WPF 控件间接修改

源代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/d8200f16691754cc61c75ecfcc7d426228a7bc55/NewhawhebichaJalciceerulebaiwair )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
