# WPF 将控件放入到 UserControl 里获取 HwndSource 为空的情况

本文记录将 WPF 控件放入到 UserControl 里，如果此 UserControl 没有被设置 Visibility 为可见过，那么放在此 UserControl 内的控件将获取不到 HwndSource 内容

<!--more-->
<!-- CreateTime:2022/2/22 8:50:54 -->

<!-- 发布 -->

如果对某个 Visual 使用 PresentationSource.FromVisual 方法获取 HwndSource 内容，获取到的返回是空值。那么可能的原因是这个 Visual 所在的容器，或者说所在的容器的所在的容器，只要在此控件的视觉树上向上寻找，能寻找到 UserControl 控件，或者是继承 UserControl 控件的控件。那么可能的原因是此 UserControl 控件，从未被设置 Visibility 为 Visible 过的原因导致的

本文接下来将使用 Demo 演示最短复现方法，告诉大家为什么从 PresentationSource.FromVisual 方法获取 null 空值的原因，和什么时候调用 PresentationSource.FromVisual 返回 null 空值

对于 WPF 里定义的几乎所有的容器控件，如 Grid 等，即使此容器控件被设置为 `Visibility="Collapsed"` 也能让里层的控件，可以拿到 HwndSource 内容。例如以下的 XAML 逻辑

```xml
    <Grid x:Name="RootGrid">
        <Grid Visibility="Collapsed">
            <TextBox x:Name="TextBox1"></TextBox>
        </Grid>
        <UserControl x:Name="UserControl" Visibility="Collapsed">
            <TextBox x:Name="TextBox2"></TextBox>
        </UserControl>
    </Grid>
```

非常相同，将一个 TextBox 分别放入到 Grid 和 UserControl 里面，将 Grid 和 UserControl 设置为 `Visibility="Collapsed"` 不可见

在 Loaded 事件里面，分别从 TextBox1 和 TextBox2 获取 HwndSource 内容，代码如下

```csharp
        public MainWindow()
        {
            InitializeComponent();

            Loaded += MainWindow_Loaded;
        }

        private void MainWindow_Loaded(object sender, RoutedEventArgs e)
        {
            var hwndSource1 = (HwndSource) PresentationSource.FromVisual(TextBox1); // not null
            var hwndSource2 = (HwndSource) PresentationSource.FromVisual(TextBox2); // null

            if (hwndSource1 is null)
            {
                throw new ArgumentNullException(nameof(hwndSource1));
            }

            if (hwndSource2 is null)
            {
                throw new ArgumentNullException(nameof(hwndSource2));
            }
        }
```

执行代码，可以看到，可以从放入到 Grid 里的 TextBox1 拿到 HwndSource 内容。从放入到 UserControl 里面的 TextBox1 拿到空

有趣的是，如果从 UserControl 拿到 HwndSource 内容，是可以拿到内容的。如果将此 UserControl 的 Visibility 先设置为 Visible 然后再设置为 Collapsed 的值，那么在 UserControl 里面的控件，依然可以拿到 HwndSource 内容。如以下代码

```xml
        <UserControl x:Name="UserControl" Visibility="Visible">
            <TextBox x:Name="TextBox2"></TextBox>
        </UserControl>
```

在后台代码，先设置 UserControl 的 Visibility 属性，再等待一秒，获取 HwndSource 内容，如以下代码

```csharp
        private async void MainWindow_Loaded(object sender, RoutedEventArgs e)
        {
            UserControl.Visibility = Visibility.Collapsed;

            await Task.Delay(1000);

            var hwndSource2 = (HwndSource) PresentationSource.FromVisual(TextBox2); // 可以拿到
        }
```

可以看到，只要 UserControl 设置过 Visibility 为 Visible 即可让放入到 UserControl 的控件拿到 HwndSource 内容

因此，如果发现从某个 Visual 上，尝试获取 HwndSource 内容失败，可以看看此控件所在的视觉树上是否被放入到 UserControl 里面，同时这个 UserControl 还没有被设置 Visibility 为 Visible 过

本文所有代码放在[github](https://github.com/lindexi/lindexi_gd/tree/013b01d618e655c8f89e088e0e5b02f7c1616233/FurwihobawNawkanenea) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/013b01d618e655c8f89e088e0e5b02f7c1616233/FurwihobawNawkanenea) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 013b01d618e655c8f89e088e0e5b02f7c1616233
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 FurwihobawNawkanenea 文件夹

[Any way to get an HWND of a UserControl? - Visual Studio 2008 - Windows Tech](http://www.windows-tech.info/11/19abe20f2274251c.php )

[Any way to get an HWND of a UserControl?](https://social.msdn.microsoft.com/Forums/vstudio/en-US/cc6297db-6ed9-4d68-abe2-47769e06d93a/any-way-to-get-an-hwnd-of-a-usercontrol?forum=wpf )

----------

更新： 

不是只有 UserControl 有此问题，而是所有需要 Template 的控件，都存在此问题。例如 Button 按钮也一样，如以下代码，将 TextBox2 放入到 Button 里面，其行为和放入到 UserControl 是相同的

```xml
        <Button x:Name="Button" Visibility="Collapsed">
            <TextBox x:Name="TextBox2"></TextBox>
        </Button>
```

更改 MainWindow_Loaded 函数为以下代码

```csharp
        private void MainWindow_Loaded(object sender, RoutedEventArgs e)
        {
            var hwndSource1 = (HwndSource) PresentationSource.FromVisual(TextBox1); // not null
            var hwndSource2 = (HwndSource) PresentationSource.FromVisual(TextBox2); // null
            var logicalParent = LogicalTreeHelper.GetParent(TextBox2); // Button
            var visualParent = VisualTreeHelper.GetParent(TextBox2); // null
        }
```

可以看到从 TextBox2 只能存在逻辑树上，没有建立过视觉树关系。原因是 Button 或 UserControl 控件，不会立即调用 ApplyTemplate 应用资源创建里层控件，只有在必要的时候才进行初始化。因此没有被初始化的 TextBox2 自然就找不到任何可用的 HwndSource 内容

更新的代码也放在[github](https://github.com/lindexi/lindexi_gd/tree/b2dafcd7f3b86efd6283dd8bf6a37cfb85765aa9/FurwihobawNawkanenea) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/b2dafcd7f3b86efd6283dd8bf6a37cfb85765aa9/FurwihobawNawkanenea) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin b2dafcd7f3b86efd6283dd8bf6a37cfb85765aa9
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 FurwihobawNawkanenea 文件夹

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31. png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
