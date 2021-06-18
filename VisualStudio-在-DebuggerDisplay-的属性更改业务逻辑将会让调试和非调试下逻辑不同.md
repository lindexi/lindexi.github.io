
# VisualStudio 在 DebuggerDisplay 的属性更改业务逻辑将会让调试和非调试下逻辑不同

本文记录我写的逗比代码，我在 DebuggerDisplay 对应的属性的 get 方法上，在这个方法里面修改了业务逻辑，如修改界面元素，此时我在 VisualStudio 断点调试下和非断点调试下的行为不相同

<!--more-->


<!-- CreateTime:2021/6/16 20:21:47 -->

<!-- 发布 -->

在 VisualStudio 调试器进入断点，默认开启隐函数求值，将会自动调用对应的类型的 DebuggerDisplay 特性里面说明的输出方法，如果对应的对象没有定义 DebuggerDisplay 特性，默认将会调用 ToString 方法。无论是在 DebuggerDisplay 特性还是在 ToString 方法里面编写变更业务逻辑的代码，都会让在断点调试下和非断点调试下的行为不相同

如以下代码，我的 xaml 界面如下

```xml
<Window x:Class="NearberjalnodarGahayjekuqi.MainWindow"
          xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
          xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
          xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
          xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
          xmlns:local="clr-namespace:NearberjalnodarGahayjekuqi"
          mc:Ignorable="d"
          Title="MainWindow" Height="450" Width="800">
  <Grid>
    <StackPanel x:Name="StackPanel">
    
    </StackPanel>
  </Grid>
</Window>
```

接下来在后台代码添加一个属性，用来在调试时输出

```csharp
        public string Debug
        {
            get
            {
                StackPanel.Children.Add(new TextBlock()
                {
                    Text = "123"
                });
                return "Foo";
            }
        }
```

在 MainWindow 添加 DebuggerDisplay 特性，代码如下

```csharp
    [DebuggerDisplay("{" + nameof(Debug) + "}")]
    public partial class MainWindow : Window
    {

    }
```

再写一点代码，用来添加断点

```csharp
    [DebuggerDisplay("{" + nameof(Debug) + "}")]
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();

            Foo();
        }

        private async void Foo()
        {
            while (true)
            {
                await Task.Delay(1000);

            }
        }

        public string Debug
        {
            get
            {
                StackPanel.Children.Add(new TextBlock()
                {
                    Text = "123"
                });
                return "Foo";
            }
        }
    }
```

在 Foo 方法里面加上断点，此时可以看到，在进入断点时，将会让界面添加 TextBlock 元素，如果没有进入断点将不会修改界面

这是因为在 DebuggerDisplay 特性里面，将会输出被花括号包含的属性名对应的属性的值。也就是对应的属性的 get 方法将会在 VisualStudio 调试调用

而如果在 get 方法编写业务逻辑，那么调用 get 的次数将会和断点进入次数相关，或和具体获取属性的次数相关


更多的代码细节还请到 [github](https://github.com/lindexi/lindexi_gd/tree/8b7af3786fd9544edeb8213d23f699938d75eb44/NearberjalnodarGahayjekuqi) 或 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/8b7af3786fd9544edeb8213d23f699938d75eb44/NearberjalnodarGahayjekuqi) 上阅读代码

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 8b7af3786fd9544edeb8213d23f699938d75eb44
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 NearberjalnodarGahayjekuqi 文件夹





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。