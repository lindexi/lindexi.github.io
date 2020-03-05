# win10 uwp 商业游戏 1.1.5

本文是在[win10 uwp 商业游戏 ](https://blog.lindexi.com/post/win10-uwp-%E5%95%86%E4%B8%9A%E6%B8%B8%E6%88%8F.html) 基础上继续开发，添加一些无聊的游戏。

因为在发布几个月，下载量很少，小伙伴说游戏就玩不到几分钟就不想玩，于是我就想加入其他游戏。

下面我来告诉大家如何在游戏中添加多个游戏。包括数据共用，导航。

<!--more-->
<!-- CreateTime:2019/5/21 11:38:20 -->

<div id="toc"></div>

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

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018111205923.jpg)

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

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201811121814.jpg)

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

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018111213321.jpg)

<!-- 草药坊 雕刻 食物 武器 材料 水果 种子-->

下面开始写一个新的游戏，叫修炼，这个游戏很简单，就是点击添加修为，暂时就这个。因为今天的 VS 炸了，所以暂时没有使用 UWP 也打不开之前的游戏，所以就重新写一个。

## 修炼游戏

下面告诉大家如何写修炼游戏，这是一个挂机游戏，需要用户不停点击。

游戏很简单，估计看一下就知道怎么做。

### 定义接口

人物有属性，修为。通过修为可以用来提升技能、自己的属性。而修为可以使用点击来获得，所以不停的点击的游戏。

那么先写一个人物接口，因为还不知道人物类需要什么东西，如何加上商业游戏，所以就写接口，定义接口的好处是可以通过别的地方给使用地方值。这样不依赖实现是比较好的。

假如换了个游戏，而这个游戏没有这里想要的人物类，如何加上原有游戏的？通过接口就可以。

```csharp
    interface IDfeppzyTmofs
    {
        
    }
```

人物接口需要实现，所以写一个类来实现，最后这个类是不需要的

```csharp

    class TdsumTzwok : IDfeppzyTmofs
    {

    }
```

人物类需要属性表示修为，因为这个值可能很大，所以就不使用 int ，请看下面

```csharp
    interface IDfeppzyTmofs
    {
        long KtrKvmvvnj { set; get; }
    }
```

这时在 TdsumTzwok 使用 Resharper 自动添加属性

```csharp
    class TdsumTzwok : IDfeppzyTmofs
    {
        public long KtrKvmvvnj { get; set; }
    }
```

### 创建界面

然后开始写  ViewModel ，这里只需要一个 ViewModel 

```csharp
    class HnlcDbtdhsdjModel:ViewModelMessage
    {
        public override void OnNavigatedFrom(object sender, object obj)
        {
            
        }

        public override void OnNavigatedTo(object sender, object obj)
        {
            
        }
    }
```

同时创建页面

```csharp
    [ViewModel(ViewModel = typeof(HnlcDbtdhsdjModel))]
    public partial class HnlcDbtdhsdjPage : Page
    {
        public HnlcDbtdhsdjPage()
        {
            InitializeComponent();
        }
    }
```

然后定义技能，技能包括有升级修为的技能和升级属性的技能

### 定义属性

暂时人物的属性就设置为力量、防御、精神力、魔力…… 好像很多的值

在网上找到的推荐是

```csharp
STR力量（攻击力） 
AGI敏捷
VIT耐力（防御力） 
INT智力（魔法攻击力防御力）
DEX命中
LUK幸运
悟性
```

在国产游戏找到的属性

 - 气血——角色当前生命值/角色当前生命值上限/角色总生命值上限，气血值为0时角色死亡
 - 法力——角色当前法力值/角色当前法力值上限/角色总法力值上限，施放技能需要消耗法力
 - 怒气——角色当前怒气值/角色总怒气值，施放绝技需要消耗怒气值
 - 根骨——影响气血上限、怒气、气血回复速度、怒气回复速度
 - 精力——影响法力上限、怒气、法力回复速度、怒气回复速度
 - 力量——影响最小物理攻击、最大物理攻击、物理防御
 - 智力——影响最小法术攻击、最大法术攻击、法术防御
 - 敏捷——影响物理命中、法术命中、物理躲避、法术躲避
 - 修炼——加入帮会后，可在帮会书院处进行修炼来增加修炼等级
 - 修为——在师门训练师处用经验、银两和银票提升技能等级可提升修为等级
 - 幸运——影响怪物死亡时物品的掉落，幸运值高更大概率掉落好物品
 - 格挡——影响完全抵消本次物理攻击的几率，格挡越高，完全抵消本次物理攻击的概率越大
 - 破盾——影响无视格挡的几率，破盾越高无视格挡的几率越高
 - 物理攻击、法术攻击——玩家的两项攻击值，攻击值越大对人或怪的伤害越大
 - 物理防御、法术防御——玩家的两项防御值，防御值越大受到的伤害越低
 - 物理躲避、法术躲避——玩家的两项躲避值，数值越大躲避他人攻击的成功率越大
 - 物理致命、法术致命——玩家的两项致命值，数值越大施放技能时打出致命一击的概率越大
 - 物理命中、法术命中——玩家的两项命中值，数值越大杀人或怪时失手的概率越低

<!-- 暴击、暴击值 -->

现在就不写这么多，先写力量、防御、精神力、魔力的值

```csharp
  interface IDfeppzyTmofs
    {
        long KtrKvmvvnj { set; get; }

        /// <summary>
        /// 力量
        /// </summary>
        int KhbfhHtuxwwrn { set; get; }

        /// <summary>
        /// 防御
        /// </summary>
        int KahdxouTrifmznz { set; get; }

        /// <summary>
        /// 精神力
        /// </summary>
        int SnmTiet { get; set; }

        /// <summary>
        /// 魔力
        /// </summary>
        int DyjgSjdbgm { set; get; }
    }
```

同时在人物类也需要写这个代码，不过不到2秒，我就写好了

```csharp
  class TdsumTzwok : IDfeppzyTmofs
    {
        public long KtrKvmvvnj { get; set; }
        public int KhbfhHtuxwwrn { get; set; }
        public int KahdxouTrifmznz { get; set; }
        public int SnmTiet { get; set; }
        public int DyjgSjdbgm { get; set; }
    }
```

接着在 ViewModel 写人物属性

```csharp
    class HnlcDbtdhsdjModel : ViewModelMessage
    {

        /// <summary>
        /// 获取设置 人物 
        /// </summary>
        public IDfeppzyTmofs KppnuhKxkpxdee
        {
            set
            {
                _kppnuhKxkpxdee = value;
                OnPropertyChanged();
            }
            get => _kppnuhKxkpxdee;
        }

        private IDfeppzyTmofs _kppnuhKxkpxdee;

        public override void OnNavigatedFrom(object sender, object obj)
        {

        }

        public override void OnNavigatedTo(object sender, object obj)
        {

        }
    }
```

这是使用代码片做出来的，关于代码片，请看[resharper 自定义代码片](./resharper-%E8%87%AA%E5%AE%9A%E4%B9%89%E4%BB%A3%E7%A0%81%E7%89%87.html )

### 定义技能

写了属性还需要写技能，技能 DexqurhctSjyfozae ，包括了修为和属性，技能都可以升级，技能可以通过修为拿到。第一个技能是点击一次获得1技能。升级需要10修为，升1级，点击一次添加一点修为。

看起来这个代码很简单

先定义技能基类 DexqurhctSjyfozae ，技能包括当前的值，升级需要多少修为，升级之后可以做什么

```csharp
    internal abstract class DexqurhctSjyfozae
    {
        /// <summary>
        /// 当前的值
        /// </summary>
        public double DklvubnuiTeqch { get; set; }

        /// <summary>
        /// 升级需要多少修为
        /// </summary>
        public double DmyikbmfDeb { get; set; }

        public string HnukhltvKfdrpokjz { get; set; }

        /// <summary>
        /// 升级之后做什么
        /// </summary>
        public abstract void DqqTsb();
    }

```

先做一个类来继承，这个类是点击添加修为

```csharp
    class HisjfnnzSqsbtuuqq:DexqurhctSjyfozae
    {
        public HisjfnnzSqsbtuuqq()
        {
            HnukhltvKfdrpokjz = "点击添加修为";

            DmyikbmfDeb = 10;

            DklvubnuiTeqch = 1;
        }

        public override void DqqTsb()
        {
               DklvubnuiTeqch += 1;
        }
    }

```

可以看到点击添加修为的时候，发现不知道怎么把修为给人添加。所以需要构造添加 IDfeppzyTmofs ，这样点击的时候才可以添加添加修为

```csharp
        public IDfeppzyTmofs Dfeppzy { get; }

        public HisjfnnzSqsbtuuqq(IDfeppzyTmofs dfeppzy)
        {
            Dfeppzy = dfeppzy;

            HnukhltvKfdrpokjz = "点击添加修为";

            DmyikbmfDeb = 10;

            DklvubnuiTeqch = 1;
        }
```

但是点击如何写？实际上需要一个点击接口，所有继承点击的都需要使用

```csharp
 interface IKdgvtziaSfs
    {
        void DdwTynktxyx();
    }
```

下面使用 HisjfnnzSqsbtuuqq 继承 IKdgvtziaSfs ，在点击的时候添加修为

```csharp
  public class HisjfnnzSqsbtuuqq : DexqurhctSjyfozae, IKdgvtziaSfs
    {
        public HisjfnnzSqsbtuuqq(IDfeppzyTmofs dfeppzy)
        {
            Dfeppzy = dfeppzy;

            HnukhltvKfdrpokjz = "点击添加修为";

            DmyikbmfDeb = 10;

            DklvubnuiTeqch = 1;
        }

        public IDfeppzyTmofs Dfeppzy { get; }

        public override void DqqTsb()
        {
            DklvubnuiTeqch += 1;
        }


        public void DdwTynktxyx()
        {
            Dfeppzy.KtrKvmvvnj += (long)Math.Floor(DklvubnuiTeqch);
        }
    }
```

### 开始写游戏

现在已经准备好了，下面就是写游戏的时候

在 ViewModel 添加一个列表，表示技能

```csharp
        public ObservableCollection<DexqurhctSjyfozae> DexqurhctSjyfozae
        {
            set 
            {
                _dexqurhctSjyfozae = value;
                OnPropertyChanged();
            }
            get => _dexqurhctSjyfozae;
        }

        private ObservableCollection<DexqurhctSjyfozae> _dexqurhctSjyfozae;
```

在跳转添加创建人物、技能

```csharp
        public override void OnNavigatedTo(object sender, object obj)
        {
            KppnuhKxkpxdee = new TdsumTzwok();
            var hisjfnnzSqsbtuuqq = new HisjfnnzSqsbtuuqq(KppnuhKxkpxdee);

            DexqurhctSjyfozae = new ObservableCollection<DexqurhctSjyfozae>()
            {
                hisjfnnzSqsbtuuqq,
            };
        }
```

因为现在的代码比较乱，所以我把所有代码写出来

```csharp
    public class HnlcDbtdhsdjModel : ViewModelMessage
    {
        /// <summary>
        /// 获取设置技能 
        /// </summary>
        public ObservableCollection<DexqurhctSjyfozae> DexqurhctSjyfozae
        {
            set
            {
                _dexqurhctSjyfozae = value;
                OnPropertyChanged();
            }
            get => _dexqurhctSjyfozae;
        }

        private ObservableCollection<DexqurhctSjyfozae> _dexqurhctSjyfozae;

        /// <summary>
        /// 获取设置 人物 
        /// </summary>
        public IDfeppzyTmofs KppnuhKxkpxdee
        {
            set
            {
                _kppnuhKxkpxdee = value;
                OnPropertyChanged();
            }
            get => _kppnuhKxkpxdee;
        }

        private IDfeppzyTmofs _kppnuhKxkpxdee;

        public HnlcDbtdhsdjModel()
        {

        }

        public override void OnNavigatedFrom(object sender, object obj)
        {

        }

        public override void OnNavigatedTo(object sender, object obj)
        {
            KppnuhKxkpxdee = new TdsumTzwok();
            var hisjfnnzSqsbtuuqq = new HisjfnnzSqsbtuuqq(KppnuhKxkpxdee);

            DexqurhctSjyfozae = new ObservableCollection<DexqurhctSjyfozae>()
            {
                hisjfnnzSqsbtuuqq,
            };
        }
    }

```

命名在我自己写的时候都是有做好的，但是因为是发出来的，所以命名都使用工具替换，所以大家看到的命名都是很不符合英文。

### 界面

先让大家看一下界面

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201812811243.jpg)

界面很简单，我直接写代码。大家也看到这个代码使用的 WPF 写的，因为现在VS无法编译UWP，所以我就先使用 WPF 来做游戏

需要在主页面添加下面的代码让游戏可以到这里

```csharp
        var hnlcDbtdhsdjPage = new HnlcDbtdhsdjPage();
            var hnlcDbtdhsdjModel = new HnlcDbtdhsdjModel();
            hnlcDbtdhsdjModel.NavigatedTo(this, null);
            hnlcDbtdhsdjPage.ViewModel = hnlcDbtdhsdjModel;
            hnlcDbtdhsdjPage.DataContext = hnlcDbtdhsdjModel;
            ShlwjKzwfkuhrz.Navigate(hnlcDbtdhsdjPage);
```

这里的 ShlwjKzwfkuhrz 就是写在界面的 Frame ，在 WPF 需要设置隐藏上面的按钮，因这个按钮很差

下面就是游戏的界面，可以看到界面之后一个 ListView 作为显示技能和人物信息

```csharp
<Page x:Class="TpwlxnpDfyecpeoh.View.HnlcDbtdhsdjPage"
      xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
      xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
      xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" 
      xmlns:d="http://schemas.microsoft.com/expression/blend/2008" 
      xmlns:local="clr-namespace:TpwlxnpDfyecpeoh.View"
      xmlns:viewModel="clr-namespace:TpwlxnpDfyecpeoh.ViewModel"
      xmlns:tpwlxnpDfyecpeoh="clr-namespace:TpwlxnpDfyecpeoh"
      mc:Ignorable="d" 
      d:DesignHeight="600" d:DesignWidth="1000"
      Title="HnlcDbtdhsdjPage">
    <Page.Resources>
        <Style x:Key="HztDmaer" TargetType="TextBlock">
            <Setter Property="Margin" Value="10,10,10,10"></Setter>
        </Style>
    </Page.Resources>
    <Grid d:DataContext="{d:DesignInstance viewModel:HnlcDbtdhsdjModel}">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>
        <Grid>
            <Grid HorizontalAlignment="Right">
                <StackPanel Margin="10,10,10,10" Orientation="Horizontal">
                    <TextBlock Text="当前修为"></TextBlock>
                    <TextBlock Text="{Binding KppnuhKxkpxdee.KtrKvmvvnj,Mode=OneWay}"></TextBlock>
                </StackPanel>
            </Grid>
        </Grid>
        <Grid Grid.Row="1">
            <ListView ItemsSource="{Binding DexqurhctSjyfozae}" HorizontalAlignment="Stretch"
                      BorderBrush="Transparent" BorderThickness="0"
                      HorizontalContentAlignment="Stretch">
                <ListView.ItemContainerStyle>
                    <Style TargetType="ListViewItem">
                        <Setter Property="Template">
                            <Setter.Value>
                                <ControlTemplate TargetType="{x:Type ListViewItem}">
                                    <Border x:Name="Bd" Padding="{TemplateBinding Padding}" SnapsToDevicePixels="true">
                                        <ContentPresenter HorizontalAlignment="{TemplateBinding HorizontalContentAlignment}" SnapsToDevicePixels="{TemplateBinding SnapsToDevicePixels}" VerticalAlignment="{TemplateBinding VerticalContentAlignment}"/>
                                    </Border>
                                </ControlTemplate>
                            </Setter.Value>
                        </Setter>
                    </Style>
                </ListView.ItemContainerStyle>
                <ListView.ItemTemplate>
                    <DataTemplate DataType="tpwlxnpDfyecpeoh:DexqurhctSjyfozae">
                        <Grid>
                            <Grid.ColumnDefinitions>
                                <ColumnDefinition Width="5*"></ColumnDefinition>
                                <ColumnDefinition Width="1*"></ColumnDefinition>
                                <ColumnDefinition Width="1*"></ColumnDefinition>
                                <ColumnDefinition Width="1*"></ColumnDefinition>
                                <ColumnDefinition Width="1*"></ColumnDefinition>
                            </Grid.ColumnDefinitions>
                            <Grid>
                                <TextBlock Style="{StaticResource HztDmaer}" Text="{Binding HnukhltvKfdrpokjz}"></TextBlock>
                            </Grid>
                            <Grid Grid.Column="1">
                                <StackPanel Orientation="Horizontal">
                                    <TextBlock Style="{StaticResource HztDmaer}" Text="当前的值"></TextBlock>
                                    <TextBlock Style="{StaticResource HztDmaer}" Text="{Binding DklvubnuiTeqch}"></TextBlock>
                                </StackPanel>
                            </Grid>
                            <Grid Grid.Column="2">
                                <StackPanel Orientation="Horizontal">
                                    <TextBlock Style="{StaticResource HztDmaer}" Text="升级需要修为"></TextBlock>
                                    <TextBlock Style="{StaticResource HztDmaer}" Text="{Binding DmyikbmfDeb}"></TextBlock>
                                </StackPanel>
                            </Grid>
                            <Grid Grid.Column="3">
                                <Button Margin="10,10,10,10" Content="升级"></Button>
                            </Grid>
                            <Grid Grid.Column="4">
                                <Button Margin="10,10,10,10" Content="点击"></Button>
                            </Grid>
                        </Grid>
                    </DataTemplate>
                </ListView.ItemTemplate>
            </ListView>
        </Grid>
    </Grid>
</Page>

```

### 点击升级

界面做完之后需要让按钮点击可以绑定后台，但是可以看到，界面绑定的值没有刷新，因为之前写属性都是没有通知，所以界面的属性都没有刷新，为了让界面可以刷新，所以需要修改属性的值

```csharp
    class TdsumTzwok : NotifyProperty, IDfeppzyTmofs
    {
        private long _ktrKvmvvnj = 0;
        private int _khbfhHtuxwwrn = 1;
        private int _kahdxouTrifmznz = 1;
        private int _snmTiet = 1;
        private int _dyjgSjdbgm = 1;

        public TdsumTzwok()
        {
        }

        public long KtrKvmvvnj
        {
            get { return _ktrKvmvvnj; }
            set
            {
                _ktrKvmvvnj = value;
                OnPropertyChanged();
            }
        }

        public int KhbfhHtuxwwrn
        {
            get { return _khbfhHtuxwwrn; }
            set
            {
                _khbfhHtuxwwrn = value;
                OnPropertyChanged();
            }
        }

        public int KahdxouTrifmznz
        {
            get { return _kahdxouTrifmznz; }
            set
            {
                _kahdxouTrifmznz = value;
                OnPropertyChanged();
            }
        }

        public int SnmTiet
        {
            get { return _snmTiet; }
            set
            {
                _snmTiet = value;
                OnPropertyChanged();
            }
        }

        public int DyjgSjdbgm
        {
            get { return _dyjgSjdbgm; }
            set
            {
                _dyjgSjdbgm = value;
                OnPropertyChanged();
            }
        }
    }

```

实际上人物的属性可以不做设置，因为可以通过更新人物属性来更新。

```csharp
    public abstract class DexqurhctSjyfozae : NotifyProperty
    {
        /// <summary>
        /// 当前的值
        /// </summary>
        public double DklvubnuiTeqch
        {
            get => _dklvubnuiTeqch;
            set
            {
                _dklvubnuiTeqch = value;
                OnPropertyChanged();
            }
        }

        /// <summary>
        /// 升级需要多少修为
        /// </summary>
        public double DmyikbmfDeb
        {
            get => _dmyikbmfDeb;
            set
            {
                _dmyikbmfDeb = value;
                OnPropertyChanged();
            }
        }

        public string HnukhltvKfdrpokjz
        {
            get => _hnukhltvKfdrpokjz;
            set
            {
                _hnukhltvKfdrpokjz = value;
                OnPropertyChanged();
            }
        }

        /// <summary>
        /// 升级之后做什么
        /// </summary>
        public abstract void DqqTsb();

        private double _dklvubnuiTeqch;
        private double _dmyikbmfDeb;
        private string _hnukhltvKfdrpokjz;
    }

```

现在开始绑定界面

因为 WPF 不能做 xbind 到函数，所以我就使用 Click 点击拿到技能升级

```csharp
                                <Button Margin="10,10,10,10" Content="升级" Click="HzmzKgeu_OnClick"></Button>

```

```csharp
        private void HzmzKgeu_OnClick(object sender, RoutedEventArgs e)
        {
            var frameworkElement = (FrameworkElement)sender;

            if (frameworkElement.DataContext is DexqurhctSjyfozae dexqurhctSjyfozae)
            {
                ViewModel.KdfoeDoct(dexqurhctSjyfozae);
            }
        }
```

这样写就是拿到 DataContext 给 ViewModel 让他判断当前的修为是否可以升级

下面的代码写在 ViewModel 判断如何可以升级就升级，不可以就告诉用户。但是如何告诉用户现在还没写

```csharp     
  public void KdfoeDoct(DexqurhctSjyfozae dexqurhctSjyfozae)
        {
            if (KppnuhKxkpxdee.KtrKvmvvnj >= dexqurhctSjyfozae.DmyikbmfDeb)
            {
                KppnuhKxkpxdee.KtrKvmvvnj -= (long) Math.Ceiling(dexqurhctSjyfozae.DmyikbmfDeb);
                dexqurhctSjyfozae.DqqTsb();
            }
            else
            {
                
            }
        }
```

写好了升级，还有点击，下面开始写点击

```csharp
                                <Button Margin="10,10,10,10" Content="点击" Click="DlsuqHmopxh_OnClick"></Button>
```

```csharp
       private void DlsuqHmopxh_OnClick(object sender, RoutedEventArgs e)
        {
            var frameworkElement = (FrameworkElement)sender;

            if (frameworkElement.DataContext is IKdgvtziaSfs kdgvtziaSfs)
            {
                kdgvtziaSfs.DdwTynktxyx();
            }
        }
```

大概写好了，直接从代码转换，判断是否可以点击，如何支持点击，就触发点击

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F%25E5%2595%2586%25E4%25B8%259A%25E6%25B8%25B8%25E6%2588%258F%2520%25E5%259B%25BE%25E7%2589%2587%2520DjqwhkKwxfit.gif)

现在的游戏已经可以玩了，于是我就把他放在了 [CSDN](http://download.csdn.net/download/lindexi_gd/10228261 ) 上，大家可以尝试玩一下。

## 相关文章

 - [win10 uwp 商业游戏 ](https://blog.lindexi.com/post/win10-uwp-%E5%95%86%E4%B8%9A%E6%B8%B8%E6%88%8F.html) 

 - [win10 uwp 商业游戏 1.2.1](https://blog.lindexi.com/post/win10-uwp-%E5%95%86%E4%B8%9A%E6%B8%B8%E6%88%8F-1.2.1.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。