# WPF 已知问题 开启 IsManipulationEnabled 之后触摸长按 RepeatButton 不会触发连续的 Click 事件

本文记录 WPF 的一个已知问题，在 RepeatButton 上开启 IsManipulationEnabled 漫游支持之后，将会导致触摸长按到 RepeatButton 之上时，不会收到源源不断的 Click 事件

<!--more-->
<!-- CreateTime:2024/2/5 9:22:14 -->

<!-- 发布 -->
<!-- 博客 -->

这是有个伙伴在 WPF 官方仓库报告的问题，详细请看 <https://github.com/dotnet/wpf/issues/8223>

原始的问题是他发现放在 ListBox 里面的 RepeatButton 无法在触摸长按的时候收到连续的 Click 事件，以为是放在 ListBox 下的 RepeatButton 存在奇怪的问题

实际上他的这个问题和 ListBox 没有任何关系，仅仅只是因为开启了 IsManipulationEnabled 之后，为了实现触摸的漫游，无法实时提升触摸为鼠标，从而导致了使用鼠标事件的 RepeatButton 无法触发源源不断的 Click 事件

这里的 IsManipulationEnabled 属性影响指的是在 RepeatButton 以及 RepeatButton 所在的上层容器控件里面设置都会影响到此行为，换句话说只要 RepeatButton 或 RepeatButton 所在的上层容器控件里面设置 IsManipulationEnabled 都能复现 RepeatButton 在触摸长按时无法收到源源不断的 Click 事件

在 ListBox 里面实际上隐藏了包含 IsManipulationEnabled 为 true 的 ScrollViewer 控件，只需将其 IsManipulationEnabled 属性设置为 false 就能继续让 RepeatButton 在触摸长按时不断触发 Click 事件，如下面代码例子，以下是 XAML 部分的代码，可以看到只是简单在 ListBox 里面放入一个 RepeatButton 控件

```xml
<ListBox x:Name="ListBox" HorizontalAlignment="Stretch" HorizontalContentAlignment="Stretch">
    <ListBoxItem>
        <RepeatButton Height="100" Click="ListBoxRepeatButtonClick" Content="Long touch 'repeat' not working on this RepeatButton"/>
    </ListBoxItem>
</ListBox>
```

此时如果直接运行代码，触摸长按 RepeatButton 按钮，将发现 Click 事件不会源源不断触发。接下来测试将 ListBox 里面的 ScrollViewer 控件的 IsManipulationEnabled 属性设置为 false 后的对 RepeatButton 的触摸长按，修改代码如下

```csharp
        public MainWindow()
        {
            InitializeComponent();
            Loaded += MainWindow_Loaded;
        }

        private void MainWindow_Loaded(object sender, RoutedEventArgs e)
        {
            if (GetChild(ListBox, o => o is ScrollViewer) is ScrollViewer scrollViewer)
            {
                scrollViewer.IsManipulationEnabled = false;
            }
        }

        private object? GetChild(DependencyObject root, Func<object, bool> predicate)
        {
            var childrenCount = VisualTreeHelper.GetChildrenCount(root);
            for (var i = 0; i < childrenCount; i++)
            {
                var child = VisualTreeHelper.GetChild(root,i);
                if (predicate(child))
                {
                    return child;
                }
                else if (child is DependencyObject dependencyObject)
                {
                    var result = GetChild(dependencyObject, predicate);
                    if (result != null)
                    {
                        return result;
                    }
                }
            }

            return null;
        }
```

以上代码通过视觉树（可视化树）找到 ListBox 里面 ScrollViewer 控件，将其 IsManipulationEnabled 属性设置为 false 从而让 RepeatButton 不再放入到任何包含 IsManipulationEnabled 为 true 的容器内，运行代码，此时可以看到放入到 ListBox 的 RepeatButton 能够在触摸长按时不断收到 Click 事件

由于此问题是 WPF 层为了实现触摸下的漫游，从而禁用了提升鼠标，我阅读了代码发现除非来一次重构否则怎么修都是打补丁，预计很长时间都不会解决这个问题

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/b7b624200bcf8ff4797c25c5ee8961b698324670/GejidebedaicifeCalnelehehar) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/b7b624200bcf8ff4797c25c5ee8961b698324670/GejidebedaicifeCalnelehehar) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin b7b624200bcf8ff4797c25c5ee8961b698324670
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin b7b624200bcf8ff4797c25c5ee8961b698324670
```

获取代码之后，进入 GejidebedaicifeCalnelehehar 文件夹