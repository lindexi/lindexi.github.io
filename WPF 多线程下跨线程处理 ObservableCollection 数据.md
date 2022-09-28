# WPF 多线程下跨线程处理 ObservableCollection 数据

本文告诉大家几个不同的方法在 WPF 里，使用多线程修改或创建 ObservableCollection 列表的数据

<!--more-->
<!-- CreateTime:2022/4/6 8:41:02 -->

<!-- 发布 -->

需要明确的是 WPF 框架下，非 UI 线程直接或间接访问 UI 是不合法的，设计如此。如此设计可以极大规避新手使用多线程造成的多线程安全问题，由于多线程安全的问题难以定位，以及解决多线程问题需要较多的专业知识。一个优秀的框架从设计上，一定需要满足不同层次开发者接入的需求。大部分微软出品的库和框架都是十分照顾到初学者的，因此默认只开单线程模型的 WPF 框架，将在开发者没有经过 Dispatcher 调度器而直接或间接访问或修改 UI 时，抛出异常

理解了以上这一点，也就了解了为什么跨线程处理 ObservableCollection 数据，大多数时候都会抛出 `System.NotSupportedException:“该类型的 CollectionView 不支持从调度程序线程以外的线程对其 SourceCollection 进行的更改。”` 等异常

在开始之前，还需要理清另一个概念，那就是 ObservableCollection 是非线程安全的。非线程安全与是否不允许非 UI 线程访问 UI 元素是完全两回事。非线程安全的类型，推荐是单一的时刻，仅有单个线程进行处理，也就是单个线程进行读写等。而 非 UI 线程访问 UI 元素是限制只有 UI 线程才能合法访问 UI 线程创建的元素。具体来说就是 ObservableCollection 是可以在任意线程创建和修改的，但是由于 ObservableCollection 是非线程安全的，因此推荐是单一的时刻，仅有单个线程进行处理。如果 ObservableCollection 被 UI 元素捕获，例如加入到 ItemsSource 里面，那么此时的 ObservableCollection 不仅只能被单一线程处理，还要求这个线程是 UI 线程

根据以上描述，可以了解到，在 WPF 里面，如果有较多数据量，想要多线程处理 ObservableCollection 集合，可以采用在非 UI 的后台线程创建 ObservableCollection 对象和修改或添加数据，完成之后再加入到 UI 线程

为了方便说明，本文新建了一个项目，本文的所有代码都可以在本文后面找到获取方法

添加一个简单的界面来方便说明，代码如下

```xml
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition></RowDefinition>
            <RowDefinition Height="Auto"></RowDefinition>
        </Grid.RowDefinitions>
        <Grid>
            <ListView x:Name="ListView">
                <ListView.ItemTemplate>
                    <DataTemplate>
                        <TextBlock Margin="10,10,10,10" Text="{Binding}"></TextBlock>
                    </DataTemplate>
                </ListView.ItemTemplate>
            </ListView>
        </Grid>
        <StackPanel Grid.Row="1" Orientation="Horizontal">
            <Button x:Name="Button1" Margin="10,10,10,10" Click="Button1_Click">方式一</Button>
            <Button x:Name="Button2" Margin="10,10,10,10" Click="Button2_Click">方式二</Button>
            <Button x:Name="Button3" Margin="10,10,10,10" Click="Button3_Click">方式三</Button>
        </StackPanel>
    </Grid>
```

以上的每个按钮分别代表不同的方法，第一个按钮就是对应开始说的第一个方法。先在后台线程创建 ObservableCollection 对象，然后在后台线程完成处理逻辑，最后赋值给 ListView 的 ItemsSource 属性，实现更新界面逻辑

```csharp
    private async void Button1_Click(object sender, RoutedEventArgs e)
    {
        var list = await Task.Run(() =>
        {
            ObservableCollection<string> data = new ObservableCollection<string>();
            for (int i = 0; i < 100; i++)
            {
                data.Add(Random.Shared.Next(1000).ToString());
            }
            return data;
        });

        // 以上代码使用 await 等待，可以自动切回主线程

        ListView.ItemsSource = list;
    }
```

如以上代码，在按钮点击时，进入按钮点击的是 UI 线程。此时在 UI 线程里面，通过 Task.Run 来切换到后台线程，在后台线程完成 list 变量的初始化逻辑。然后再赋值给 ListView 的 ItemsSource 属性

上面代码符合了上文说的逻辑条件，首先 ObservableCollection 非线程安全，单一的时刻，只有一个线程进行访问。上面代码先是后台线程创建和处理 ObservableCollection 对象，接下来后台线程执行完成，通过 await 自动依靠同步上下文调度到主线程，将后台线程创建的 ObservableCollection 对象赋值给 list 变量，此时的后台线程退出对 ObservableCollection 对象的任何访问，也就是在此单一的时刻，只有后台线程一个线程在访问。接下来进入 `ListView.ItemsSource = list` 也就是将 list 交给 UI 线程，在此单一的时刻，也只有 UI 线程，一个线程在访问

在将 ObservableCollection 关联到 UI 线程之前，对 ObservableCollection 的任何处理都不会涉及到访问 UI 元素，因此也就没有了非 UI 线程不能访问 UI 元素的限制。只有在调用 `ListView.ItemsSource = list` 代码之后，才将 ObservableCollection 关联到 UI 线程。在此代码执行之后，就不能通过后台线程去修改 list 变量对应的对象了，因为此时的修改将会间接在后台线程访问到 UI 元素

那如果期望是在后台线程处理原有 UI 线程关联的 ObservableCollection 呢？这就是本文的第二个方法。读取 ObservableCollection 的列表元素内容，不会涉及到访问 UI 元素，因此可以在后台线程进行读取列表元素，读取列表元素也就是等于可以对原有的列表拷贝一份

这里需要再次说明 ObservableCollection 非线程安全，单一的时刻，只有一个线程进行访问才是安全的。换句话说，虽然代码层面上，可以在后台线程拷贝和 UI 线程关联的 ObservableCollection 的列表元素内容，但是此时毕竟 UI 线程和后台线程都拥有访问相同的一个 ObservableCollection 列表的能力，必须从业务上确保只有后台线程在访问，而 UI 线程不会对 ObservableCollection 列表进行任何的改动

在确保 UI 线程不会改动到 ObservableCollection 列表的时候，可以采用如下方法，在后台线程拷贝一份作为新的 ObservableCollection 对象，然后对此新的对象进行处理。完成之后，再将新的 ObservableCollection 对象赋值给到 UI 进行绑定

```csharp
    private async void Button2_Click(object sender, RoutedEventArgs e)
    {
        // 假定 ListView.ItemsSource 存在源了
        if (ListView.ItemsSource is not ObservableCollection<string> list)
        {
            // 如果假设失败，强行给一个源
            list = new();
            ListView.ItemsSource = list;
        }

        var newList = await Task.Run(() =>
        {
            var data = new ObservableCollection<string>(list);

            // 模拟对原有的列表进行处理
            if (data.Count > 0)
            {
                for (int i = 0; i < 100; i++)
                {
                    data.Move(Random.Shared.Next(data.Count), Random.Shared.Next(data.Count));
                }
            }

            return data;
        });

        ListView.ItemsSource = newList;
    }
```

以上方法可以实现在后台线程对现有的和 UI 绑定的 ObservableCollection 的更改，由于是放在后台线程执行，基本上不需要担心拷贝的耗时

第三个方法是自己实现一个类似 ObservableCollection 的类型。在 WPF 里面，只要一个集合类型的对象继承了 INotifyCollectionChanged 接口，即可在集合变更的时候，通过 WPF 框架监听 CollectionChanged 事件重新更新 UI 元素，自己实现的代码大概如下

```csharp
public class FooList<T> : Collection<T>, INotifyCollectionChanged
{
    protected override void InsertItem(int index, T item)
    {
        base.InsertItem(index, item);

        Application.Current.Dispatcher.InvokeAsync(() =>
        {
            CollectionChanged?.Invoke(this,
          new NotifyCollectionChangedEventArgs(NotifyCollectionChangedAction.Add, item, index));
        });
    }

    protected override void RemoveItem(int index)
    {
        var item = this[index];

        base.RemoveItem(index);

        Application.Current.Dispatcher.InvokeAsync(() =>
        {
            CollectionChanged?.Invoke(this,
          new NotifyCollectionChangedEventArgs(NotifyCollectionChangedAction.Remove, item, index));
        });
    }

    protected override void SetItem(int index, T item)
    {
        var oldItem = this[index];
        base.SetItem(index, item);
        Application.Current.Dispatcher.InvokeAsync(() =>
        {
            CollectionChanged?.Invoke(this,
          new NotifyCollectionChangedEventArgs(NotifyCollectionChangedAction.Replace, item, oldItem, index));
        });
    }

    public event NotifyCollectionChangedEventHandler? CollectionChanged;
}
```

如上面代码可以看到，在集合变更的代码里面，都通过 Dispatcher 调度到 UI 线程触发事件用来通知。依靠此机制可以实现在后台线程处理时，依然是让此 FooList 对应的对象是绑定在 UI 线程上

使用 FooList 的例子如下

```csharp
    private async void Button3_Click(object sender, RoutedEventArgs e)
    {
        if (ListView.ItemsSource is not FooList<string> list)
        {
            list = new FooList<string>();

            ListView.ItemsSource = list;
        }

        await Task.Run(() =>
        {
            for (int i = 0; i < 100; i++)
            {
                list.Add(Random.Shared.Next(100).ToString());
            }
        });

        await Task.Delay(TimeSpan.FromSeconds(1));

        await Task.Run(() =>
        {
            for (int i = 0; i < 10; i++)
            {
                list.RemoveAt(i);
            }
        });
       
        await Task.Delay(TimeSpan.FromSeconds(1));

        await Task.Run(() =>
        {
            for (int i = 0; i < 10; i++)
            {
                list[i] = i.ToString();
            }
        });
    }
```

以上的 FooList 只是一个例子，用于告诉大家可以使用 INotifyCollectionChanged 的方式自己实现在集合变更的时候通知主线程，而集合的处理本身可以放在其他的线程。但是这个方法在使用的时候，必须关注线程安全问题。例如以上的代码，如果没有关注线程安全，在通知 UI 线程集合变更之后，刚好 UI 线程去读取此集合新的值的时候，集合本身就被其他线程更改了内容，那么此时的逻辑就不是符合预期的

以上的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/df7d9da863047fa0a46bc97e782b054da63fc394/LeejurkawbaicarkeNayqechurcear) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/df7d9da863047fa0a46bc97e782b054da63fc394/LeejurkawbaicarkeNayqechurcear) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin df7d9da863047fa0a46bc97e782b054da63fc394
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 LeejurkawbaicarkeNayqechurcear 文件夹

关于 UWP 部分，请看 [win10 uwp 通知列表](https://blog.lindexi.com/post/win10-uwp-%E9%80%9A%E7%9F%A5%E5%88%97%E8%A1%A8.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
