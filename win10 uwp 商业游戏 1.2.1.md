# win10 uwp 商业游戏 1.2.1

上一个游戏已经告诉大家如何写多个游戏，现在继续写这个无聊的游戏。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:50 -->

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

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201824105713.jpg)

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

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201824113040.jpg)

## 添加技能

为了可以添加任意的技能，所以这里添加一个类用来写有哪些技能可以添加的，这个类是 KwxTpivzdikn ，里面有一个属性用来放技能，不过这个TeddtHlhkgt类实际上有一个方法，在调用就可以返回技能，所以不需要使用上面的代码。但是存在一些可以传入的参数，所以还是可以添加一个类

```
    internal class KwxTpivzdikn
    {
        public KwxTpivzdikn()
        {

        }
        public List<SbjHoeb> DpwHoulmqbx { set; get; } = new List<SbjHoeb>();
    }

```

这个类主要是创建技能，因为TeddtHlhkgt类会有其他很多代码，所以就不让他做这个。

### 提供修为

先创建一个用于提高修为的技能试试，这个技能是点击升级只有需要的修为是 1.3-1.5之间。也就是当前需要需要修为 10 ，升级之后就需要 13-15的修为。

```
class HmsfKjirukoon : SbjHoeb
    {
        public override DexqurhctSjyfozae StdshakHngld(IDfeppzyTmofs donufyqgfKfnazhrcg)
        {

        }
    }
```

但是还需要创建一个修为技能，这个类可以被修改每次升级的值

```
internal class KwxTpivzdikn
    {
        public KwxTpivzdikn()
        {
        }

        public List<SbjHoeb> DpwHoulmqbx { set; get; } = new List<SbjHoeb>();
    }

    class HmsfKjirukoon : SbjHoeb
    {
        public override DexqurhctSjyfozae StdshakHngld(IDfeppzyTmofs donufyqgfKfnazhrcg)
        {
            var sisHhmpja = ran.Next(1300, 1500) / 1000.0;
            return new HcaoxbsDpitcsg(donufyqgfKfnazhrcg)
            {
                KimtDvznfc = sisHhmpja
            };
        }
    }

    class HcaoxbsDpitcsg : DexqurhctSjyfozae
    {
        public HcaoxbsDpitcsg(IDfeppzyTmofs dfeppzy)
        {
            Dfeppzy = dfeppzy;

            HnukhltvKfdrpokjz = "点击添加修为";

            DmyikbmfDeb = 10;

            DklvubnuiTeqch = 1;
        }

        public IDfeppzyTmofs Dfeppzy { get; }

        public double KimtDvznfc { get; set; }

        public override void DqqTsb()
        {
            DmyikbmfDeb *= KimtDvznfc;
            DklvubnuiTeqch += 1;
        }


        public void DdwTynktxyx()
        {
            Dfeppzy.KtrKvmvvnj += (long) Math.Floor(DklvubnuiTeqch);
        }
    }
```    

先试试点击添加技能会出现的技能，不过需要在点击的时候添加上面代码

```
      public KwxTpivzdikn()
        {
            DpwHoulmqbx.Add(new HmsfKjirukoon());
        }

              public TeddtHlhkgt(IDfeppzyTmofs smktuoiznSobrf)
        {
            SmktuoiznSobrf = smktuoiznSobrf;

            DpwHoulmqbx.AddRange(new KwxTpivzdikn().DpwHoulmqbx);
        }
```

因为创建了一个新技能，这个在界面是无法显示，所以需要修改 DyqbdpreKuoujeq ，在找不到对应的界面就打开默认


```
    public class DyqbdpreKuoujeq : DataTemplateSelector
    {
        public List<TuikyyDikvqp> TuikyyDikvqps { get; } = new List<TuikyyDikvqp>();

        public DataTemplate SheojwpnyHbqnybpa { get; set; }

        public override DataTemplate SelectTemplate(object item, DependencyObject container)
        {
            var tuikyyDikvqp = TuikyyDikvqps.FirstOrDefault(temp=>temp.KwxvrmxDhzyozzwx(item));
            if (tuikyyDikvqp != null)
            {
                return tuikyyDikvqp.TnhvrarvlDaz;
            }

            if (SheojwpnyHbqnybpa != null)
            {
                return SheojwpnyHbqnybpa;
            }

            return base.SelectTemplate(item, container);
        }
    }

```    

打开界面添加下面代码

```
            <local:DyqbdpreKuoujeq.SheojwpnyHbqnybpa>
                <DataTemplate DataType="tpwlxnpDfyecpeoh:DexqurhctSjyfozae">
                    <Grid>
                        <Grid.ColumnDefinitions>
                            <ColumnDefinition Width="5*" />
                            <ColumnDefinition Width="1*" />
                            <ColumnDefinition Width="1*" />
                            <ColumnDefinition Width="1*" />
                            <ColumnDefinition Width="1*" />
                        </Grid.ColumnDefinitions>
                        <Grid>
                            <TextBlock Style="{StaticResource HztDmaer}" Text="{Binding HnukhltvKfdrpokjz}" />
                        </Grid>
                        <Grid Grid.Column="1">
                            <StackPanel Orientation="Horizontal">
                                <TextBlock Style="{StaticResource HztDmaer}" Text="当前的值" />
                                <TextBlock Style="{StaticResource HztDmaer}"
                                           Text="{Binding DklvubnuiTeqch,Converter={StaticResource DyakmdgwuTlaukxbo}}" />
                            </StackPanel>
                        </Grid>
                        <Grid Grid.Column="2">
                            <StackPanel Orientation="Horizontal">
                                <TextBlock Style="{StaticResource HztDmaer}" Text="升级需要修为" />
                                <TextBlock Style="{StaticResource HztDmaer}"
                                           Text="{Binding DmyikbmfDeb,Converter={StaticResource DyakmdgwuTlaukxbo}}" />
                            </StackPanel>
                        </Grid>
                        <Grid Grid.Column="3">
                            <RepeatButton Margin="10,10,10,10" Content="升级" Click="HzmzKgeu_OnClick" />
                        </Grid>
                        <Grid Grid.Column="4">
                            <Button Margin="10,10,10,10" Content="点击" Click="DlsuqHmopxh_OnClick" />
                        </Grid>
                    </Grid>
                </DataTemplate>
            </local:DyqbdpreKuoujeq.SheojwpnyHbqnybpa>

```

现在运行一下，可以看到下面的界面

![](http://image.acmx.xyz/cc09f7ce-2059-5d67-bc56-036f04c13efa%2F2018218101640.jpg)

但是点击很多次就可以看到界面是点击添加修为，不知道他的每次点击可以添加多少，所以修改他的命名

```
        public override DexqurhctSjyfozae StdshakHngld(IDfeppzyTmofs donufyqgfKfnazhrcg)
        {
            var sisHhmpja = ran.Next(1300, 1500) / 1000.0;
            var kiyKyovife = "点击添加修为"+sisHhmpja.ToString("N");
            return new HcaoxbsDpitcsg(donufyqgfKfnazhrcg)
            {
                HnukhltvKfdrpokjz = kiyKyovife,
                KimtDvznfc = sisHhmpja
            };
        }

```

![](http://image.acmx.xyz/cc09f7ce-2059-5d67-bc56-036f04c13efa%2F2018218101839.jpg)

这样多次点击添加技能就能找到一个升级需要比较少技能的值

## 创建.net的共用项目

在过了一个月，我的UWP还是没有安装成功，所以我只能使用 WPF 来写，但是如何把 WPF 写的直接放在 UWP，一个简单的方法是创建共享项目，但是还有其他方法。创建 .net standard 项目，不过创建这个需要使用的 WPF 程序是 .net Framework 4.6.2 和以上。

现在创建一个 KnxetfaHjpkymq 的项目，让 WPF 项目引用他，然后把一些类从 WPF 放在这个项目。

关于.net Framework 对应，请看下面

![4yV.png](https://whoimg.com/images/2018/02/18/4yV.png)

## 其他文章

 - [win10 uwp 商业游戏](https://lindexi.gitee.io/post/win10-uwp-%E5%95%86%E4%B8%9A%E6%B8%B8%E6%88%8F.html )

 - [win10 uwp 商业游戏 1.1.5](https://lindexi.gitee.io/post/win10-uwp-%E5%95%86%E4%B8%9A%E6%B8%B8%E6%88%8F-1.1.5.html )

## 感谢

感谢 [无名图床](https://whoimg.com) 提供图片上传

最近我的[图床](https://www.microsoft.com/store/productId/9NBLGGH562R2)已经流量用太多了

不过我更新了图床的界面，现在的界面很好看了

![](http://image.acmx.xyz/lindexi%2F201859125215290.jpg)

[点击下载](https://www.microsoft.com/store/productId/9NBLGGH562R2)

图床可以用来快速上传图片到服务器，用来写博客放图片

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)