# win10 uwp 商业游戏 1.1.5

本文是在[win10 uwp 商业游戏 ](./win10-uwp-%E5%95%86%E4%B8%9A%E6%B8%B8%E6%88%8F.html) 基础上继续开发，添加一些无聊的游戏。

因为在发布几个月，下载量很少，小伙伴说游戏就玩不到几分钟就不想玩，于是我就想加入其他游戏。

下面我来告诉大家如何在游戏中添加多个游戏。包括数据共用，导航。

<!--more-->
<!-- csdn -->

开始问到小伙伴，他说这个游戏因为玩到后面发现可以买的东西很多，于是就不能继续玩。商店没有限制用户可以买多少的东西，所以就买最贵的，游戏就很简单了，买一下最贵的东西，等到他升价再卖出去。所以看起来很简单，他就不想玩了，于是为了添加游戏的难度，我就先添加商店可以卖出数

## 商店可以卖出数

为了让游戏比较好玩，就添加了商店的一个东西最多的可以买入和卖出，但是买入和卖出的值需要在买入的时候添加卖出的值，所以就给一个值，表示市场需要的。如果买入了，那么市场需要的就增加，如果卖出了，市场需要就减少。

假设一个商品在一天的市场需要数是 UgetkmeOulajjz ，假设这个值是 100 ，通过随机提供的数量 KadzufmVtvnpn 获得商店还剩下多少这个商品。于是可以买入的数量就是 KadzufmVtvnpn ，可以卖出的数量就是 (UgetkmeOulajjz - KadzufmVtvnpn)

这样对一个商品不停买卖也是做不到的，虽然可以买价格最高的商品，但是因为限制了商品的 UgetkmeOulajjz ，所以最后可以买入的数量也是有限的。

UgetkmeOulajjz 的数值开始是随机生成，在 10-100 左右。

## 多个游戏

现在还可以添加新的功能，如打怪，大概钱到了 10000 就可以开始买灵石，然后进去特殊的游戏。

这时需要界面可以使用多个页面

先创建一个页面 KdgqelPocuesyvPage 和他的抽线 KdgderhlMzhpModel，因为使用了框架所以需要让 KdgderhlMzhpModel 继承 ViewModelMessage 。需要 KdgqelPocuesyvPage 添加指定的 ViewModel 

先到 IckixyYofiModel 跳转到 KdgderhlMzhpModel ，然后在这里添加界面，先添加一些测试的按钮，用于跳转

```csharp
    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <Grid.RowDefinitions>
            <RowDefinition Height="89*"/>
            <RowDefinition Height="11*"/>
        </Grid.RowDefinitions>

        <Grid>
            <ListView >
            </ListView>
        </Grid>
        <Grid Grid.Row="1">

            <Grid>
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="317*"/>
                    <ColumnDefinition Width="690*"/>
                    <ColumnDefinition Width="493*"/>
                </Grid.ColumnDefinitions>
                <Button Content="跳转测试 跳转到页面1" Margin="10,10,10,10"
                        HorizontalAlignment="Stretch"></Button>
                <Button Grid.Column="1" Content="跳转测试 跳转到页面2" Margin="10,10,10,10"
                        HorizontalAlignment="Stretch"></Button>
                <Button Grid.Column="2" Content="跳转测试 跳转到页面3" Margin="10,10,10,10"
                        HorizontalAlignment="Stretch"></Button>
            </Grid>
        </Grid>
    </Grid>

```

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018111205923.jpg)

中间的 ListView 就来绑定按钮，绑定的按钮参见：[win10 UWP ListView](https://lindexi.oschina.io/lindexi/post/win10-UWP-ListView.html )

### 添加列表

如果需要绑定ListView ，需要先创建一个类型，这个类型直接包括显示的文字和跳转的页面等，这里先显示文字

```csharp
    public class YcftxgEcgs
    {
        public string YwkLjuakc { get; set; }
    }
```

跳转的页面使用 KmulfmFshszg ，这样就可以在 ViewModel 添加一个列表

```csharp
        public ObservableCollection<YcftxgEcgs> VsibgyegZkyi { get; set; } = new ObservableCollection<YcftxgEcgs>();

```

然后在构造函数创建内容

```csharp
        public KdgderhlMzhpModel()
        {
            for (int i = 0; i < 100; i++)
            {
                VsibgyegZkyi.Add(new YcftxgEcgs("按钮" + i));
            }
        }
```

在界面使用绑定的代码

```csharp
       <ListView d:DataContext="{d:DesignInstance viewModel:KdgderhlMzhpModel,d:IsDesignTimeCreatable=True}"
                      HorizontalAlignment="Stretch" 
                      ItemsSource="{x:Bind ViewModel.VsibgyegZkyi}">
                <ListView.ItemsPanel>
                    <ItemsPanelTemplate>
                        <ItemsWrapGrid Orientation="Horizontal"></ItemsWrapGrid>
                    </ItemsPanelTemplate>
                </ListView.ItemsPanel>
                <ListView.ItemContainerStyle>
                    <Style TargetType="ListViewItem">
                        <Setter Property="HorizontalContentAlignment" Value="Stretch"/>
                    </Style>
                </ListView.ItemContainerStyle>

                <ListView.ItemTemplate>
                    <DataTemplate>
                        <Grid>
                            <Button Content="{Binding}"></Button>
                        </Grid>
                    </DataTemplate>
                </ListView.ItemTemplate>
            </ListView>
```

可以看到下面的界面

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201811121814.jpg)

但是按钮按下的时候需要告诉上一层的消息，这时可以使用发送消息方法

在 ViewModel 添加一个属性，这个属性可以告诉界面按钮点击

```csharp
    public string PngvnwIjpy
        {
            get { return _pngvnwIjpy; }
            set
            {
                _pngvnwIjpy = value;
                OnPropertyChanged();
            }
        }

        private string _pngvnwIjpy;
```

在 YcftxgEcgs 添加点击事件

```csharp
        public void SloafemulWugxhrd()
        {
            IxfmHlsg?.Invoke(this, null);
        }

        public event EventHandler IxfmHlsg;
```

然后需要修改 ViewModel 的方法

```csharp
        public KdgderhlMzhpModel()
        {
            for (int i = 0; i < 100; i++)
            {
                var whzmnTstbq = new YcftxgEcgs("按钮" + i);
                whzmnTstbq.IxfmHlsg += (s, e) => PngvnwIjpy = ((YcftxgEcgs) s).YwkLjuakc;
                VsibgyegZkyi.Add(whzmnTstbq);
            }
        }
```

在界面绑定，创建一个文本

```csharp

                <ListView.ItemTemplate>
                    <DataTemplate x:DataType="viewModel:YcftxgEcgs">
                        <Grid>
                            <Button Content="{x:Bind YwkLjuakc}" Click="{x:Bind SloafemulWugxhrd}"></Button>
                        </Grid>
                    </DataTemplate>
                </ListView.ItemTemplate>

                           <TextBlock Text="{x:Bind ViewModel.PngvnwIjpy,Mode=OneWay}"></TextBlock>

```

这时点击就可以看到文本显示点击按钮

### 导航

现在可以添加导航界面，在上面的界面，下面就是各个不同的游戏，上面是游戏的界面。

先在 ViewModel 跳转修改为进入游戏

```csharp
            Navigate(typeof(TvrwgrnNnuModel), null);

```

然后在新建游戏或打开游戏的时候，跳转到导航界面

打开 TvrwgrnNnuModel ，把里面的 StorageModel 替换为 KdgderhlMzhpModel，这样就可以打开导航界面

导航现在只有商店和仓库，之后有其他的导航再这里加入

因为现在的 KdgderhlMzhpModel 也需要有跳转，所以把他继承 NavigateViewModel 然后在页面跳转添加下面代码

```csharp
       CombineViewModel(Application.Current.GetType().GetTypeInfo().Assembly);
            AllAssemblyComposite(Application.Current.GetType().GetTypeInfo().Assembly);
```

这样就可以获得页面和处理

在导航界面添加框架，然后给 ViewModel ，请看代码

```csharp
            <Frame x:Name="VjagWrgesebmy"></Frame>

```

```csharp
            ViewModel = (KdgderhlMzhpModel) e.Parameter;
            ViewModel.Content = VjagWrgesebmy;
            DataContext = ViewModel;
            base.OnNavigatedTo(e);
```

但是因为 ViewModel 是先跳转然后再进入页面，所以不可以在 KdgderhlMzhpModel 跳转的时候进入游戏，需要添加函数 UmfqawovKaxkrdrg 设置跳转的之后进行跳转

```csharp
        public void UmfqawovKaxkrdrg()
        {
            //进行跳转
            Navigate(typeof(StorageModel), null);
        }
```

```csharp
            ViewModel = (KdgderhlMzhpModel) e.Parameter;
            ViewModel.Content = VjagWrgesebmy;
            ViewModel.UmfqawovKaxkrdrg();
            DataContext = ViewModel;
```

这样就可以在点击新建游戏的时候看到跳转到市场

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018111213321.jpg)

<!-- 草药坊 雕刻 食物 武器 材料 水果 种子-->


## 相关文章

 - [win10 uwp 商业游戏 ](./win10-uwp-%E5%95%86%E4%B8%9A%E6%B8%B8%E6%88%8F.html) 

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。