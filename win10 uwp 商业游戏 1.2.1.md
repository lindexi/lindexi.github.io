# win10 uwp 商业游戏 1.2.1

上一个游戏已经告诉大家如何写多个游戏，现在继续写这个无聊的游戏。

<!--more-->
<!-- csdn -->

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





## 其他文章

 - [win10 uwp 商业游戏 ](./win10-uwp-%E5%95%86%E4%B8%9A%E6%B8%B8%E6%88%8F.html) 

 - [win10 uwp 商业游戏 1.1.5](./win10-uwp-%E5%95%86%E4%B8%9A%E6%B8%B8%E6%88%8F-1.1.5.html)

