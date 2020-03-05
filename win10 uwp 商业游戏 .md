# win10 uwp 商业游戏 

本文告诉大家去做一个商业游戏，游戏很简单，几乎没有什么技术。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:50 -->


游戏的开始，需要添加框架库，于是引用我自己写的库。

首先是创建一个启动页面，这个页面是显示启动的。

在显示启动的时候，是需要加载游戏需要使用的资源，如果觉得这时需要控制进度条，就需要使用注入的方法，给他知道现在的进度，不过我现在不去做这里，于是就很简单的代码做出来启动页面。

现在的启动页还是空的，但是先不管他。

然后开始写一个欢迎页面，也就是开始游戏、继续、帮助等的页面，暂时先不做。

但是写了这么多，都不需要在 MainPage 写一个代码，现在就到了在 MainPage
开始写代码，这时的代码就是添加一个 Frame 作为跳转，和一个菜单。当然菜单现在还没东西，只是需要添加到这里。

看下，现在已经创建了两个页面。一个是启动页，一个是欢迎页。

接着开始做游戏的主页面，请注意，在写的时候，一个页面都是对应一个视图。可以看到在写的时候，不需要去管两个页面之间的逻辑。当然现在也无法管。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201792392012.jpg)

可以知道游戏的主页面需要有仓库、买东西的店铺、买东西的地方和工厂。

那么需要想游戏是如何玩的。

因为这个游戏是我昨天看到买菜的人说的，我就想去做一个。

首先物品有很多，而且可以通过工厂把低级的物品转为高级的物品。

物品包括

```csharp
种子
小米
麦
面
面包
番薯
矿石
铁
铜
木头
石
矿
碳
煤
锄头
椅子
桌子
雕刻刀
大刀
弓箭

```

如果还有想到其他的物品，欢迎告诉我。不过在看完之前，请不要很快的告诉我，你希望添加那些物品，请在知道游戏如何玩之后在告诉我，你喜欢的物品。

物品是包括当前价格和市场价格，其中当前价格是不变的，而市场价格是会在 80%-120% 之间波动。

开始说游戏的商店是如何卖东西的，游戏使用的商店是可以通过市场买东西，然后把东西买个买东西的人。商店可以有很多个，当然，现在做的只有一个。商店存在信誉，商店来的客人数是不固定，和商店所在地方和商店信誉有关。如果一个地方人很多那么来商店的人会很多。如果商店信誉很好，来商店的人很多。

一个买东西的人，会告诉商店他买的是有哪些，数量，希望买的价格。买的价格就是市场价格，市场价格总是变化，需要在每个时间决定是否买入。

所以商店的输入按钮现在就有了一个，下一时间，也就是在这个时间是否有买东西的人过来，是否决定要从市场买东西。

所以点击一下按钮，就可以获得当前有多少个买东西的过来，处理完之后在去市场买东西。

买东西的人过来，就会说他现在需要买什么，当然游戏一开始不会立刻就出现高级的人来买比较大的东西。来买东西的人是有分等级的，也就是他有多少钱，于是按照钱判断他现在可以买多少东西。

如果商店存在他可以买的东西，如商店现在有东西

```csharp
种子 2
小米 10
麦   10
面   2
```

他想买东西是

```csharp
种子 1
小米 2
麦   3
```

于是刚好都可以买入，于是顾客就会给满分，因为他想要的商店可以买到所有他想要的东西。于是商店加信誉1。

一个人总的带来信誉 是这样计算，他想要的商品价格可以获得数

```csharp
 var s = 买东西人买东西加起来的所有价值 
 var a = 商店提供商品总价格
 带来信誉 = 2 * a / s - 1
```

如果只能满足买东西人一半的需要，那么不会带来任何的信誉提升。

于是在点击下一时间，就开始计算有多少买东西的人，他们有多少钱，所以需要物品这个类给出他可以被改变的概率，通过他的钱就可以算出。于是物品需要的属性就可以得到，物品名、物品需要多少钱，物品总量。现在物品的总量可以不计算。

来买东西的人有这些属性，所有的钱，级别。买的东西。级别就是商店有多少信誉在可以吸引这个买东西的人，这里写为一个列表

但是开始先做一个简单的程序，之后在添加比较多功能。

## 界面

游戏的界面很重要，但是我就写一个简单的界面。我先给界面的截图，可以看到这个界面很简单，就是一个列表和一些按钮

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201791101021.jpg)

列表有个问题，如何做表头，实际我也没办法，于是用了下面的代码

```csharp
                <ListView Grid.Row="1" ItemsSource="{x:Bind View.PropertyStorage}"
                          SelectedItem="{Binding CarloPiperIsaacProperty,Mode=TwoWay}">
                    <FrameworkElement.Resources>
                        <Style TargetType="TextBlock">
                            <Setter Property="HorizontalAlignment" Value="Center"></Setter>
                        </Style>
                    </FrameworkElement.Resources>
                    <ListView.ItemContainerStyle>
                        <Style TargetType="ListViewItem">
                            <Setter Property="HorizontalContentAlignment" Value="Stretch"></Setter>
                        </Style>
                    </ListView.ItemContainerStyle>
                    <ListView.Header>
                        <Grid>
                            <Grid.ColumnDefinitions>
                                <ColumnDefinition Width="30*"></ColumnDefinition>
                                <ColumnDefinition Width="15*"></ColumnDefinition>
                                <ColumnDefinition Width="15*"></ColumnDefinition>
                                <ColumnDefinition Width="15*"></ColumnDefinition>
                            </Grid.ColumnDefinitions>
                            <TextBlock Text="名称"></TextBlock>
                            <TextBlock Grid.Column="1" Text="价格"></TextBlock>
                            <TextBlock Grid.Column="2" Text="仓库拥有"></TextBlock>
                            <TextBlock Grid.Column="3" Text="买入价钱"></TextBlock>
                        </Grid>
                    </ListView.Header>
                    <ListView.ItemTemplate>
                        <DataTemplate x:DataType="view:Property">
                            <Grid>
                                <Grid.ColumnDefinitions>
                                    <ColumnDefinition Width="30*"></ColumnDefinition>
                                    <ColumnDefinition Width="15*"></ColumnDefinition>
                                    <ColumnDefinition Width="15*"></ColumnDefinition>
                                    <ColumnDefinition Width="15*"></ColumnDefinition>
                                </Grid.ColumnDefinitions>
                                <FrameworkElement.Resources>
                                    <Style TargetType="TextBlock">
                                        <Setter Property="HorizontalAlignment" Value="Center"></Setter>
                                    </Style>
                                </FrameworkElement.Resources>
                                <TextBlock Text="{x:Bind Name}"></TextBlock>
                                <TextBlock Grid.Column="1" Text="{x:Bind Price,Mode=OneWay}"></TextBlock>
                                <TextBlock Grid.Column="2" Text="{x:Bind Num,Mode=OneWay}"></TextBlock>
                                <TextBlock Grid.Column="3" Text="{x:Bind AshliLyverGeraldo,Mode=OneWay}"></TextBlock>
                            </Grid>
                        </DataTemplate>
                    </ListView.ItemTemplate>
                </ListView>

```

上面代码为了让列表不压缩宽，于是就需要使用 ItemContainerStyle ，请看代码

```csharp
                 <ListView.ItemContainerStyle>
                        <Style TargetType="ListViewItem">
                            <Setter Property="HorizontalContentAlignment" Value="Stretch"></Setter>
                        </Style>
                    </ListView.ItemContainerStyle>
```

为了让列表所有文字都居中，不想写给每个文字，但是列表之外的文字就不居中，于是修改列表内文字的居中就可以使用下面代码

```csharp 



                 <FrameworkElement.Resources>
                                    <Style TargetType="TextBlock">
                                        <Setter Property="HorizontalAlignment" Value="Center"></Setter>
                                    </Style>
                                </FrameworkElement.Resources>
```

这样在列表内的文字就会居中，而列表外的文字就不会居中。

如果在列表使用 x:bind 那么需要使用 DataType 来告诉绑定的类型，所以需要数据的类型是什么，不然就无法通过。所以在写列表之前还需要定义好数据，于是让我来告诉大家这个游戏需要的数据。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201797183335.jpg)

然后就需要开始绑定进去了，不过这时发现游戏需要的数据很简单，暂时我就不继续写代码，如果需要代码，请看 [VarietyHiggstGushed](https://github.com/lindexi/UWP/tree/master/uwp/src/VarietyHiggstGushed)

## 添加图标

可以看到，上面的界面没有图标，看起来不好看，所以需要给他添加一些图标。图标可以到 [http://www.iconfont.cn/](http://www.iconfont.cn/) 下载，在这里下载需要注意版权问题，不过我的这个在下载的时候就有看了，好像是不需要给钱。

首先把图片放在 Assest 文件夹，然后就可以设置出来了。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201791617817.jpg)

```csharp
            <Grid>
                <StackPanel Orientation="Horizontal"
                            HorizontalAlignment="Right">
                    <FrameworkElement.Resources>
                        <Style TargetType="StackPanel">
                            <Setter Property="Margin" Value="10,10,10,10"></Setter>
                        </Style>
                    </FrameworkElement.Resources>
                    <StackPanel Orientation="Horizontal">
                        <Image Source="ms-appx:///Assets/天.png" Height="20" Width="20"></Image>
                        <TextBlock Text="天数"></TextBlock>
                        <TextBlock Text="{x:Bind View.PinkieDuchesneGeraldo,Mode=OneWay}"></TextBlock>
                    </StackPanel>
                    <StackPanel Orientation="Horizontal">
                        <Image Source="ms-appx:///Assets/仓库.png" Height="20" Width="20"></Image>
                        <TextBlock Text="仓库容量"></TextBlock>
                        <TextBlock Text="{x:Bind View.JwStorage.Transit,Mode=OneWay}"/>
                        <TextBlock Text="/"></TextBlock>
                        <TextBlock Text="{x:Bind View.JwStorage.TransitStorage,Mode=OneWay}"></TextBlock>
                    </StackPanel>
                    <StackPanel Orientation="Horizontal">
                        <TextBlock Text="金钱"></TextBlock>
                        <TextBlock Text="{x:Bind View.JwStorage.TranStoragePrice,Mode=OneWay}"></TextBlock>
                    </StackPanel>
                </StackPanel>
            </Grid>

```

可以看到图片的写法 Source 的值是使用`ms-appx`，这里就是从资源获得，如果希望知道这个代码是如何写，我有博客[win10 uwp 访问解决方案文件](http://lindexi.oschina.io/lindexi//post/win10-uwp-%E8%AE%BF%E9%97%AE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88%E6%96%87%E4%BB%B6/) 里面就告诉大家如何写。

注意需要设置图片的大小，可以运行程序，然后开始设置，这样界面就可以看到修改，但是需要保存才可以看到。

可以看到原来的代码是 [VarietyHiggstGushed](https://github.com/lindexi/UWP/commit/3f58dbee64d0a7b42768c1acdecb6812fc789ac5#diff-e941c1b41207d1fb99a9b5bc32e2d30b) 买东西的界面看起来不好

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017916171231.jpg)

下面就需要修改这个，修改为一个好看的

可以使用 ContentDialog 弹出一个好看的界面

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201791720859.jpg)

那么如何做这个界面，简单的方法是做一个用户控件，这个控件的界面很简单，但是后台需要写一些属性，这个属性就是买入的当前，买入最大值，对应还有卖出的。

于是这个界面的后台代码就是几个属性，还有在设置 ViewModel 时更新属性

```csharp
  public sealed partial class JediahPage : UserControl
    {
        public JediahPage()
        {
            this.InitializeComponent();
        }

        public StorageModel ViewModel
        {
            get { return _viewModel; }
            set
            {
                _viewModel = value;
                //最大可以买
                NewLansheehyBrunaSharon = (int) Math.Floor(_viewModel.JwStorage.TranStoragePrice /
                                                           _viewModel.CarloPiperIsaacProperty.Price);
                var sresidue = _viewModel.JwStorage.TransitStorage - _viewModel.JwStorage.Transit;
                NewLansheehyBrunaSharon = NewLansheehyBrunaSharon > sresidue ? sresidue : NewLansheehyBrunaSharon;
                AimeeLansheehyBrunaSharon = _viewModel.CarloPiperIsaacProperty.Num;
            }
        }

        public static readonly DependencyProperty NewLansheehyBrunaSharonNumProperty = DependencyProperty.Register(
            "NewLansheehyBrunaSharonNum", typeof(int), typeof(JediahPage), new PropertyMetadata(default(int)));

        public int NewLansheehyBrunaSharonNum
        {
            get { return (int) GetValue(NewLansheehyBrunaSharonNumProperty); }
            set { SetValue(NewLansheehyBrunaSharonNumProperty, value); }
        }

        public static readonly DependencyProperty NewLansheehyBrunaSharonProperty = DependencyProperty.Register(
            "NewLansheehyBrunaSharon", typeof(int), typeof(JediahPage), new PropertyMetadata(default(int)));

        public int NewLansheehyBrunaSharon
        {
            get { return (int) GetValue(NewLansheehyBrunaSharonProperty); }
            set { SetValue(NewLansheehyBrunaSharonProperty, value); }
        }

        public static readonly DependencyProperty AimeeLansheehyBrunaSharonNumProperty = DependencyProperty.Register(
            "AimeeLansheehyBrunaSharonNum", typeof(int), typeof(JediahPage), new PropertyMetadata(default(int)));

        public int AimeeLansheehyBrunaSharonNum
        {
            get { return (int) GetValue(AimeeLansheehyBrunaSharonNumProperty); }
            set { SetValue(AimeeLansheehyBrunaSharonNumProperty, value); }
        }

        public static readonly DependencyProperty AimeeLansheehyBrunaSharonProperty = DependencyProperty.Register(
            "AimeeLansheehyBrunaSharon", typeof(int), typeof(JediahPage), new PropertyMetadata(default(int)));

        private StorageModel _viewModel;

        public int AimeeLansheehyBrunaSharon
        {
            get { return (int) GetValue(AimeeLansheehyBrunaSharonProperty); }
            set { SetValue(AimeeLansheehyBrunaSharonProperty, value); }
        }

        public event EventHandler Close;

        private void NewLansheehy(object sender, RoutedEventArgs e)
        {
            ViewModel.LansheehyBrunaSharon = NewLansheehyBrunaSharonNum;
            ViewModel.NewLansheehyBrunaSharon();
            Close?.Invoke(this, null);
        }

        private void AimeeLansheehy(object sender, RoutedEventArgs e)
        {
            ViewModel.LansheehyBrunaSharon = AimeeLansheehyBrunaSharonNum;
            ViewModel.AimeeLansheehyBrunaSharon();
            Close?.Invoke(this, null);
        }

        private void MnewBruna(object sender, RoutedEventArgs e)
        {
            NewLansheehyBrunaSharonNum = NewLansheehyBrunaSharon;
        }

        private void MaimeeBruna(object sender, RoutedEventArgs e)
        {
            AimeeLansheehyBrunaSharonNum = AimeeLansheehyBrunaSharon;
        }

        private void CloseButton_OnClick(object sender, RoutedEventArgs e)
        {
            Close?.Invoke(this, null);
        }
    }
```

界面代码很简单

```csharp
    <Grid >
        <Grid Margin="10,10,10,10">
            <Grid.RowDefinitions>
                <RowDefinition Height="auto" />
                <RowDefinition Height="64*" />
            </Grid.RowDefinitions>
            <Grid>
                <Button FontFamily="Segoe MDL2 Assets" Content="&#xE10A;" HorizontalAlignment="Right"
                        Click="CloseButton_OnClick">
                </Button>
            </Grid>
            <Grid Grid.Row="1" HorizontalAlignment="Center" VerticalAlignment="Center">

                <Grid>
                    <Grid.RowDefinitions>
                        <RowDefinition Height="64*" />
                        <RowDefinition Height="auto" />
                        <RowDefinition Height="auto" />
                        <RowDefinition Height="100*" />
                    </Grid.RowDefinitions>
                    <Grid>
                        <TextBlock Text="{x:Bind ViewModel.CarloPiperIsaacProperty.Name}"></TextBlock>
                    </Grid>
                    <Grid Grid.Row="1">
                        <StackPanel Orientation="Horizontal">
                            <Image Source="ms-appx:///Assets/仓库.png" Height="20" Width="20"></Image>
                            <TextBlock Text="仓库拥有:"></TextBlock>
                            <TextBlock Text="{x:Bind ViewModel.CarloPiperIsaacProperty.Num}"></TextBlock>
                        </StackPanel>
                    </Grid>
                    <Grid Width="300" Grid.Row="2">
                        <StackPanel Orientation="Horizontal">
                            <TextBlock Text="$"></TextBlock>
                            <TextBlock Text="买入价格"></TextBlock>
                            <TextBlock Text="{x:Bind ViewModel.CarloPiperIsaacProperty.AshliLyverGeraldo,Converter={StaticResource ConverDoubleStr}}"></TextBlock>
                        </StackPanel>

                        <StackPanel Orientation="Horizontal"
                                    HorizontalAlignment="Right">
                            <TextBlock Text="$"></TextBlock>
                            <TextBlock Text="市场价格"></TextBlock>
                            <TextBlock Text="{x:Bind ViewModel.CarloPiperIsaacProperty.Price}"></TextBlock>
                        </StackPanel>
                    </Grid>
                    <Grid Grid.Row="3">
                        <Grid.RowDefinitions>
                            <RowDefinition Height="74*" />
                            <RowDefinition Height="85*" />
                        </Grid.RowDefinitions>
                        <Grid>
                            <Grid.ColumnDefinitions>
                                <ColumnDefinition Width="307*" />
                                <ColumnDefinition Width="auto" />
                            </Grid.ColumnDefinitions>
                            <Slider Margin="10,10,10,10"
                                    Value="{x:Bind NewLansheehyBrunaSharonNum,Mode=TwoWay,Converter={StaticResource ResourceKey=Convert}}"
                                    Maximum="{x:Bind NewLansheehyBrunaSharon}">
                            </Slider>
                            <StackPanel Grid.Column="1" Orientation="Horizontal">
                                <Button Content="max" Click="MnewBruna"></Button>
                                <Button Content="买入" Click="NewLansheehy"></Button>
                            </StackPanel>
                        </Grid>
                        <Grid Grid.Row="1">
                            <Grid.ColumnDefinitions>
                                <ColumnDefinition Width="59*" />
                                <ColumnDefinition Width="auto" />
                            </Grid.ColumnDefinitions>
                            <Slider Margin="10,10,10,10"
                                    Value="{x:Bind AimeeLansheehyBrunaSharonNum,Mode=TwoWay,Converter={StaticResource Convert}}"
                                    Maximum="{x:Bind AimeeLansheehyBrunaSharon}">

                            </Slider>
                            <StackPanel Grid.Column="1" Orientation="Horizontal">
                                <Button Content="max" Click="MaimeeBruna"></Button>
                                <Button Content="卖出" Click="AimeeLansheehy"></Button>
                            </StackPanel>
                        </Grid>
                    </Grid>
                </Grid>
            </Grid>
        </Grid>
    </Grid>

```

可以看到需要两个转换器，一个是把字符串转 double 一个是显示 double 保留小数后两位，这个实现很简单，我就不说了。那么接下来就是使用这个界面，使用方法请看下面。

```csharp
            var temp = new JediahPage()
            {
                ViewModel = View,
            };
            ContentDialog contentDialog = new ContentDialog()
            {
                Content = temp,
                IsPrimaryButtonEnabled = false,
                IsSecondaryButtonEnabled = false,
            };

            temp.Close += (s, args) => contentDialog.Hide();

            await contentDialog.ShowAsync();
```

主要注意把 close 事件写在显示前，然后去掉默认的按钮。

大概这样就可以运行了，其他的代码不是重要的，所以就不说啦。现在我把游戏发在微软商店，点击[下载](ms-windows-store://pdp/?productid=9pb0286g2ldr)。

这就是[商业游戏](ms-windows-store://pdp/?productid=9pb0286g2ldr) 1.0.75 ，在这个版本发布之后，还会继续开发，但是就不在这篇文章更新了。

下面是相关文章

 - [win10 uwp 商业游戏 ](https://lindexi.gitee.io/post/win10-uwp-%E5%95%86%E4%B8%9A%E6%B8%B8%E6%88%8F.html) 

 - [win10 uwp 商业游戏 1.1.5](https://lindexi.gitee.io/post/win10-uwp-%E5%95%86%E4%B8%9A%E6%B8%B8%E6%88%8F-1.1.5.html)

 - [win10 uwp 商业游戏 1.2.1 ](https://lindexi.gitee.io/post/win10-uwp-%E5%95%86%E4%B8%9A%E6%B8%B8%E6%88%8F-1.2.1.html )

<!--  - [win10 uwp 商业游戏 ](./win10-uwp-%E5%95%86%E4%B8%9A%E6%B8%B8%E6%88%8F.html) 

 - [win10 uwp 商业游戏 1.1.5](./win10-uwp-%E5%95%86%E4%B8%9A%E6%B8%B8%E6%88%8F-1.1.5.html)

 - [win10 uwp 商业游戏 1.2.1 ](./win10-uwp-%E5%95%86%E4%B8%9A%E6%B8%B8%E6%88%8F-1.2.1.html ) -->

## 感谢

[walterlv](https://walterlv.github.io/ )

[JAKE](http://niuyanjie.oschina.io/blog/ )

落書き  https://www.pixiv.net/member_illust.php?mode=medium&illust_id=64830430

欢迎大家来我的[博客](https://lindexi.gitee.io)，现在csdn的博客文章发布了就不再编辑，如果希望看最新的文章，请到我的[博客](https://lindexi.gitee.io)。本文也发在博客园备份，如果大家有什么建议，请在博客下面评论。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 