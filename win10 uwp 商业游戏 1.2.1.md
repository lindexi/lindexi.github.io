# win10 uwp 商业游戏 1.2.1

上一个游戏已经告诉大家如何写多个游戏，现在继续写这个无聊的游戏。

<!--more-->
<!-- csdn -->
<div id="toc"></div>

希望大家在看这篇文章之前先看[ win10 uwp 商业游戏](./win10-uwp-%E5%95%86%E4%B8%9A%E6%B8%B8%E6%88%8F-1.1.5.html)，在这个文章告诉了大家如何创建游戏。

## 修改数值

可以从上一篇的博客的游戏看到升级太简单，所以需要在点击升级点击的时候，下次升级需要的添加

写法很简单，请看下面

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
            DmyikbmfDeb *= 1.5;
            DklvubnuiTeqch += 1;
        }


        public void DdwTynktxyx()
        {
            Dfeppzy.KtrKvmvvnj += (long)Math.Floor(DklvubnuiTeqch);
        }
    }


```

但是可以看到界面显示的小数点，需要只显示两位，可以使用转换，请看下面

```csharp
    public class DyakmdgwuTlaukxbo:IValueConverter
    {
        public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
        {
            if (value is double kbjjDzn)
            {
                return kbjjDzn.ToString("F");
            }

            return "";
        }

        public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
        {
            throw new NotImplementedException();
        }
    }

```

```csharp
        <local:DyakmdgwuTlaukxbo x:Key="DyakmdgwuTlaukxbo"></local:DyakmdgwuTlaukxbo>

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
                                    <TextBlock Style="{StaticResource HztDmaer}" Text="{Binding DklvubnuiTeqch,Converter={StaticResource DyakmdgwuTlaukxbo}}"></TextBlock>
                                </StackPanel>
                            </Grid>
                            <Grid Grid.Column="2">
                                <StackPanel Orientation="Horizontal">
                                    <TextBlock Style="{StaticResource HztDmaer}" Text="升级需要修为"></TextBlock>
                                    <TextBlock Style="{StaticResource HztDmaer}" Text="{Binding DmyikbmfDeb,Converter={StaticResource DyakmdgwuTlaukxbo}}"></TextBlock>
                                </StackPanel>
                            </Grid>
                            <Grid Grid.Column="3">
                                <RepeatButton Margin="10,10,10,10" Content="升级" Click="HzmzKgeu_OnClick"></RepeatButton>
                            </Grid>
                            <Grid Grid.Column="4">
                                <Button Margin="10,10,10,10" Content="点击" Click="DlsuqHmopxh_OnClick"></Button>
                            </Grid>
                        </Grid>
                    </DataTemplate>
                </ListView.ItemTemplate>
            </ListView>

```

## 点击添加技能

现在有了值，可以添加一个新的技能，这个技能就是点击添加新的技能。

添加技能这个技能是不需要升级的，所以存在一个值告诉界面，当前这个技能是不需要升级的，而且没有当前值，所以这个类不能继承 DexqurhctSjyfozae 。但是可以从另一个方面去做，本来添加技能就是升级，所以当前的值就是升级的值。所以这个类只需要继承技能基类。

创建一个类 KdlunmmHhrs ，这个类是添加技能

```csharp
    class KdlunmmHhrs : DexqurhctSjyfozae
    {
        public override void DqqTsb()
        {

        }
    }
```

点击升级的时候就添加新的技能，所以需要一个技能工厂类，这个类用来创建技能

所以创建一个类 TeddtHlhkgt 是工厂类，创建各种技能，因为技能很多，所以需要具体技能的工厂类，这个基类是 SbjHoeb ，通过这个类就可以创建技能，所以可以看到这个类的代码可以这样写

```csharp
    class TeddtHlhkgt
    {
        public TeddtHlhkgt(IDfeppzyTmofs smktuoiznSobrf)
        {
            SmktuoiznSobrf = smktuoiznSobrf;
        }

        public List<SbjHoeb> DpwHoulmqbx { set; get; } = new List<SbjHoeb>();

        public IDfeppzyTmofs SmktuoiznSobrf { get; }

        public DexqurhctSjyfozae StdshakHngld()
        {
            return DpwHoulmqbx[ran.Next(DpwHoulmqbx.Count)].StdshakHngld(SmktuoiznSobrf);
        }

        private static Random ran = new Random();
    }

```

```csharp
    internal abstract class SbjHoeb
    {
        public abstract DexqurhctSjyfozae StdshakHngld(IDfeppzyTmofs donufyqgfKfnazhrcg);
    }
```

因为很多类都需要传入 IDfeppzyTmofs 人物类，所以就需要在函数添加参数，在使用的时候输入。

然后创建添加修为的工厂类 SdmqokThd 这个类返回技能

```csharp
   class SdmqokThd : SbjHoeb
    {
        public override DexqurhctSjyfozae StdshakHngld(IDfeppzyTmofs donufyqgfKfnazhrcg)
        {
            return new HisjfnnzSqsbtuuqq(donufyqgfKfnazhrcg);
        }
    }
```

但是现在的添加技能类还没有添加技能，因为技能的属性在他的上面，他拿不到，所以只能使用构造函数传入

```csharp
    internal class KdlunmmHhrs : DexqurhctSjyfozae
    {
        public KdlunmmHhrs(ObservableCollection<DexqurhctSjyfozae> dexqurhctSjyfozae, IDfeppzyTmofs tdheituHnks)
        {
            DexqurhctSjyfozae = dexqurhctSjyfozae;
            TdheituHnks = tdheituHnks;

             HnukhltvKfdrpokjz = "点击添加技能";
            DfacHbl = new TeddtHlhkgt(tdheituHnks);
            DmyikbmfDeb = 100;
        }

        public override void DqqTsb()
        {
            DklvubnuiTeqch = DklvubnuiTeqch + 1;

            DexqurhctSjyfozae.Add(DfacHbl.StdshakHngld());
        }

        private ObservableCollection<DexqurhctSjyfozae> DexqurhctSjyfozae { get; }

        private IDfeppzyTmofs TdheituHnks { get; }

        private TeddtHlhkgt DfacHbl { get; }
    }

```

可以看到，技能类都需要在构造添加 DmyikbmfDeb 升级需要的修为，在点击升级 DqqTsb 添加当前值 DklvubnuiTeqch ，但是这个值不是每次都添加1。先添加这个技能来看一下

打开 HnlcDbtdhsdjModel ，在跳转时，创建 KdlunmmHhrs ，然后添加到技能

```csharp
       public override void OnNavigatedTo(object sender, object obj)
        {
            KppnuhKxkpxdee = new TdsumTzwok();
            var hisjfnnzSqsbtuuqq = new HisjfnnzSqsbtuuqq(KppnuhKxkpxdee);

            DexqurhctSjyfozae = new ObservableCollection<DexqurhctSjyfozae>()
            {
                hisjfnnzSqsbtuuqq,
            };

            var kdlunmmHhrs = new KdlunmmHhrs(DexqurhctSjyfozae, KppnuhKxkpxdee);
            DexqurhctSjyfozae.Add(kdlunmmHhrs);
        }
```

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201824105713.jpg)

现在可以看到，添加技能这个存在点击按钮，但是不需要点击，只有升级。所以如何修改？下面来告诉大家使用列表模板

实际在[win10 uwp 列表模板选择器](https://lindexi.gitee.io/post/win10-uwp-%E5%88%97%E8%A1%A8%E6%A8%A1%E6%9D%BF%E9%80%89%E6%8B%A9%E5%99%A8.html )已经有告诉大家如何写了，所以这里只是很简单告诉大家，这里的代码需要如何写

先创建一个类 DyqbdpreKuoujeq ，这个类就是选择不同的数据

```csharp
    public class DyqbdpreKuoujeq : DataTemplateSelector
    {
        public override DataTemplate SelectTemplate(object item, DependencyObject container)
        {
            return base.SelectTemplate(item, container);
        }
    }
```

因为存在很多技能，所以只能创建技能的类来添加判断，创建类 TuikyyDikvqp ，这个类判断当前是什么技能，能否应用。

```csharp
    public abstract class TuikyyDikvqp
    {
        public abstract bool KwxvrmxDhzyozzwx(object hflozThhgjas);

        public DataTemplate TnhvrarvlDaz { get; set; }
    }
```

这个类有两个函数，函数 KwxvrmxDhzyozzwx 就是判断当前输入的数据是否可以使用这个类的 TnhvrarvlDaz ，因为现在不需要排序，所以就不需要添加优先级。

这个类的 TnhvrarvlDaz 就是，如果输入的数据可以使用，那么就使用这个类的 TnhvrarvlDaz 。这个属性的值是在界面创建，不能在后台创建。

下面添加 DyqbdpreKuoujeq 一个列表，判断当前输入的数据是否在哪个列表，如果是返回

```csharp
    public class DyqbdpreKuoujeq : DataTemplateSelector
    {
        public List<TuikyyDikvqp> TuikyyDikvqps { get; } = new List<TuikyyDikvqp>();

        public override DataTemplate SelectTemplate(object item, DependencyObject container)
        {
            var tuikyyDikvqp = TuikyyDikvqps.FirstOrDefault(temp=>temp.KwxvrmxDhzyozzwx(item));
            if (tuikyyDikvqp != null)
            {
                return tuikyyDikvqp.TnhvrarvlDaz;
            }
            return base.SelectTemplate(item, container);
        }
    }
```

一般都是判断如果没有符合的，就返回定义的一个值，但是这里因为不需要这样写。

创建一个用于点击的界面 TxvigDixcee

```csharp
    public class TxvigDixcee : TuikyyDikvqp
    {
        public override bool KwxvrmxDhzyozzwx(object hflozThhgjas)
        {
            return hflozThhgjas is IKdgvtziaSfs;
        }
    }
```

那么如何在界面使用，请看下面

```csharp
  <Page.Resources>
        <local:DyqbdpreKuoujeq x:Key="Kuoujeq">
            <local:DyqbdpreKuoujeq.TuikyyDikvqps>
                <local:TxvigDixcee>
                    <local:TxvigDixcee.TnhvrarvlDaz>
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
                                        <TextBlock Style="{StaticResource HztDmaer}" Text="{Binding DklvubnuiTeqch,Converter={StaticResource DyakmdgwuTlaukxbo}}"></TextBlock>
                                    </StackPanel>
                                </Grid>
                                <Grid Grid.Column="2">
                                    <StackPanel Orientation="Horizontal">
                                        <TextBlock Style="{StaticResource HztDmaer}" Text="升级需要修为"></TextBlock>
                                        <TextBlock Style="{StaticResource HztDmaer}" Text="{Binding DmyikbmfDeb,Converter={StaticResource DyakmdgwuTlaukxbo}}"></TextBlock>
                                    </StackPanel>
                                </Grid>
                                <Grid Grid.Column="3">
                                    <RepeatButton Margin="10,10,10,10" Content="升级" Click="HzmzKgeu_OnClick"></RepeatButton>
                                </Grid>
                                <Grid Grid.Column="4">
                                    <Button Margin="10,10,10,10" Content="点击" Click="DlsuqHmopxh_OnClick"></Button>
                                </Grid>
                            </Grid>
                        </DataTemplate>
                    </local:TxvigDixcee.TnhvrarvlDaz>
                </local:TxvigDixcee>
            </local:DyqbdpreKuoujeq.TuikyyDikvqps>
        </local:DyqbdpreKuoujeq>
    </Page.Resources>
```

添加一个添加技能 ThhlqolHdhkja ，先创建一个类

```csharp
    public class ThhlqolHdhkja : TuikyyDikvqp
    {
        public override bool KwxvrmxDhzyozzwx(object hflozThhgjas)
        {
            return hflozThhgjas is KdlunmmHhrs;
        }
    }
```

因为技能数是整数，所以创建一个转换

```csharp
    public class SnlSlejfmnfk : IValueConverter
    {
        public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
        {
            if (value is double kbjjDzn)
            {
                return ((int)kbjjDzn).ToString();
            }

            return "";
        }

        public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
        {
            throw new NotImplementedException();
        }
    }
```

添加界面，需要写新的界面，一般都是在一个假的界面写列表，复制到这里，直接写是看不到界面

```csharp
       <local:ThhlqolHdhkja>
                    <local:ThhlqolHdhkja.TnhvrarvlDaz>
                        <DataTemplate DataType="tpwlxnpDfyecpeoh:DexqurhctSjyfozae">
                            <Grid>
                                <Grid.ColumnDefinitions>
                                    <ColumnDefinition Width="5*"></ColumnDefinition>
                                    <ColumnDefinition Width="1*"></ColumnDefinition>
                                    <ColumnDefinition Width="1*"></ColumnDefinition>
                                    <ColumnDefinition Width="1*"></ColumnDefinition>
                                </Grid.ColumnDefinitions>
                                <Grid>
                                    <TextBlock Style="{StaticResource HztDmaer}" Text="{Binding HnukhltvKfdrpokjz}"></TextBlock>
                                </Grid>
                                <Grid Grid.Column="1">
                                    <StackPanel Orientation="Horizontal">
                                        <TextBlock Style="{StaticResource HztDmaer}" Text="技能"></TextBlock>
                                        <TextBlock Style="{StaticResource HztDmaer}" Text="{Binding DklvubnuiTeqch,Converter={StaticResource SnlSlejfmnfk}}"></TextBlock>
                                    </StackPanel>
                                </Grid>
                                <Grid Grid.Column="2">
                                    <StackPanel Orientation="Horizontal">
                                        <TextBlock Style="{StaticResource HztDmaer}" Text="需要修为"></TextBlock>
                                        <TextBlock Style="{StaticResource HztDmaer}" Text="{Binding DmyikbmfDeb,Converter={StaticResource SnlSlejfmnfk}}"></TextBlock>
                                    </StackPanel>
                                </Grid>
                                <Grid Grid.Column="3">
                                    <RepeatButton Margin="10,10,10,10" Content="添加技能" Click="HzmzKgeu_OnClick"></RepeatButton>
                                </Grid>
                            </Grid>
                        </DataTemplate>
                    </local:ThhlqolHdhkja.TnhvrarvlDaz>
                </local:ThhlqolHdhkja>
```

现在的界面代码

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
        <local:DyakmdgwuTlaukxbo x:Key="DyakmdgwuTlaukxbo"></local:DyakmdgwuTlaukxbo>
        <local:SnlSlejfmnfk x:Key="SnlSlejfmnfk"></local:SnlSlejfmnfk>

        <local:DyqbdpreKuoujeq x:Key="Kuoujeq">
            <local:DyqbdpreKuoujeq.TuikyyDikvqps>
                <local:TxvigDixcee>
                    <local:TxvigDixcee.TnhvrarvlDaz>
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
                                        <TextBlock Style="{StaticResource HztDmaer}" Text="{Binding DklvubnuiTeqch,Converter={StaticResource DyakmdgwuTlaukxbo}}"></TextBlock>
                                    </StackPanel>
                                </Grid>
                                <Grid Grid.Column="2">
                                    <StackPanel Orientation="Horizontal">
                                        <TextBlock Style="{StaticResource HztDmaer}" Text="升级需要修为"></TextBlock>
                                        <TextBlock Style="{StaticResource HztDmaer}" Text="{Binding DmyikbmfDeb,Converter={StaticResource DyakmdgwuTlaukxbo}}"></TextBlock>
                                    </StackPanel>
                                </Grid>
                                <Grid Grid.Column="3">
                                    <RepeatButton Margin="10,10,10,10" Content="升级" Click="HzmzKgeu_OnClick"></RepeatButton>
                                </Grid>
                                <Grid Grid.Column="4">
                                    <Button Margin="10,10,10,10" Content="点击" Click="DlsuqHmopxh_OnClick"></Button>
                                </Grid>
                            </Grid>
                        </DataTemplate>
                    </local:TxvigDixcee.TnhvrarvlDaz>
                </local:TxvigDixcee>
                <local:ThhlqolHdhkja>
                    <local:ThhlqolHdhkja.TnhvrarvlDaz>
                        <DataTemplate DataType="tpwlxnpDfyecpeoh:DexqurhctSjyfozae">
                            <Grid>
                                <Grid.ColumnDefinitions>
                                    <ColumnDefinition Width="5*"></ColumnDefinition>
                                    <ColumnDefinition Width="1*"></ColumnDefinition>
                                    <ColumnDefinition Width="1*"></ColumnDefinition>
                                    <ColumnDefinition Width="1*"></ColumnDefinition>
                                </Grid.ColumnDefinitions>
                                <Grid>
                                    <TextBlock Style="{StaticResource HztDmaer}" Text="{Binding HnukhltvKfdrpokjz}"></TextBlock>
                                </Grid>
                                <Grid Grid.Column="1">
                                    <StackPanel Orientation="Horizontal">
                                        <TextBlock Style="{StaticResource HztDmaer}" Text="技能"></TextBlock>
                                        <TextBlock Style="{StaticResource HztDmaer}" Text="{Binding DklvubnuiTeqch,Converter={StaticResource SnlSlejfmnfk}}"></TextBlock>
                                    </StackPanel>
                                </Grid>
                                <Grid Grid.Column="2">
                                    <StackPanel Orientation="Horizontal">
                                        <TextBlock Style="{StaticResource HztDmaer}" Text="需要修为"></TextBlock>
                                        <TextBlock Style="{StaticResource HztDmaer}" Text="{Binding DmyikbmfDeb,Converter={StaticResource SnlSlejfmnfk}}"></TextBlock>
                                    </StackPanel>
                                </Grid>
                                <Grid Grid.Column="3">
                                    <RepeatButton Margin="10,10,10,10" Content="添加技能" Click="HzmzKgeu_OnClick"></RepeatButton>
                                </Grid>
                            </Grid>
                        </DataTemplate>
                    </local:ThhlqolHdhkja.TnhvrarvlDaz>
                </local:ThhlqolHdhkja>
            </local:DyqbdpreKuoujeq.TuikyyDikvqps>
        </local:DyqbdpreKuoujeq>
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
                      HorizontalContentAlignment="Stretch"
                      ItemTemplateSelector="{StaticResource Kuoujeq}">
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

            </ListView>
        </Grid>
    </Grid>
</Page>

```

可以看到，现在的运行就是

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201824113040.jpg)

## 其他文章

 - [win10 uwp 商业游戏 ](./win10-uwp-%E5%95%86%E4%B8%9A%E6%B8%B8%E6%88%8F.html) 

 - [win10 uwp 商业游戏 1.1.5](./win10-uwp-%E5%95%86%E4%B8%9A%E6%B8%B8%E6%88%8F-1.1.5.html)

