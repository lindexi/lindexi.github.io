
# WPF ListBox 的选择

本文告诉大家在 ListBox 做选择时，多选时 SelectedItem 和 SelectedIndex 的值。

<!--more-->


<!-- csdn -->

首先写一个界面，两个按钮和一个ListBox ，点击第一个按钮选择多个元素，点击第二个按钮就显示当前的  SelectedItem 和 SelectedIndex

```csharp
    <Grid>
        <ListBox x:Name="QjdckjpVemnepybg" SelectionMode="Multiple"></ListBox>
        <Button Width="100" Height="100" Click="ButtonBase_OnClick"></Button>
        <Button Margin="100,10,10,10" Width="100" Height="100" Click="RyltnqiUmqgwuz_OnClick"></Button>
    </Grid>
```

在点击前需要给 ListBox 数据，请看下面

```csharp
        public MainWindow()
        {
            InitializeComponent();
            QjdckjpVemnepybg.ItemsSource = FyuzbqklBxdk;

            for (int i = 0; i < 100; i++)
            {
                FyuzbqklBxdk.Add(i.ToString());
            }
        }

        public ObservableCollection<string> FyuzbqklBxdk { get; } = new ObservableCollection<string>();

```

点击第一个按钮选择多个元素

```csharp
        private void ButtonBase_OnClick(object sender, RoutedEventArgs e)
        {
            QjdckjpVemnepybg.SelectedItems.Clear();
            QjdckjpVemnepybg.SelectedItems.Add(FyuzbqklBxdk[2]);
            QjdckjpVemnepybg.SelectedItems.Add(FyuzbqklBxdk[5]);
        }
```

点击第二个按钮显示 SelectedItem 和 SelectedIndex

```csharp
        private void RyltnqiUmqgwuz_OnClick(object sender, RoutedEventArgs e)
        {
            Console.WriteLine(QjdckjpVemnepybg.SelectedItem);
            Console.WriteLine(QjdckjpVemnepybg.SelectedIndex);
        }
```

可以看到，点击第一个按钮之后，SelectedItem 和 SelectedIndex 都是 2 因为他会显示多选的第一个。

如果选择的是 SelectedItem ，那么 SelectedItems 就只有 SelectedItem 的元素 

```csharp
      private void ButtonBase_OnClick(object sender, RoutedEventArgs e)
        {
            QjdckjpVemnepybg.SelectedItem = FyuzbqklBxdk[2];
        }

        private void RyltnqiUmqgwuz_OnClick(object sender, RoutedEventArgs e)
        {
            Console.WriteLine(QjdckjpVemnepybg.SelectedItems.Count);
            foreach (var temp in QjdckjpVemnepybg.SelectedItems)
            {
                Console.WriteLine(temp);
            }
        }
```

如果选择是 SelectedIndex ，那么 SelectedItems 有对应序号的元素

```csharp
    private void ButtonBase_OnClick(object sender, RoutedEventArgs e)
        {
            QjdckjpVemnepybg.SelectedIndex = 2;
        }

        private void RyltnqiUmqgwuz_OnClick(object sender, RoutedEventArgs e)
        {
            Console.WriteLine(QjdckjpVemnepybg.SelectedItems.Count);
            foreach (var temp in QjdckjpVemnepybg.SelectedItems)
            {
                Console.WriteLine(temp);
            }
        }
```

如果这时 SelectedIndex 为 -1 ，那么 SelectedItems 没有选择元素

```csharp
        private void ButtonBase_OnClick(object sender, RoutedEventArgs e)
        {
            QjdckjpVemnepybg.SelectedIndex = -1;
        }

        private void RyltnqiUmqgwuz_OnClick(object sender, RoutedEventArgs e)
        {
            Console.WriteLine(QjdckjpVemnepybg.SelectedItems.Count);
            foreach (var temp in QjdckjpVemnepybg.SelectedItems)
            {
                Console.WriteLine(temp);
            }
        }
```

如果这时的 SelectedItems 没有选择元素，那么 SelectedIndex 是 -1

如果这时没有开启多选，那么在 SelectedItem 选中元素时，SelectedItems 只有他选中的。所以一般可以通过 SelectedItems 判断当前选中的。




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。