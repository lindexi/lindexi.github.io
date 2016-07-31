本文主要说如何在UWP切换主题，并且如何制作主题。

一般我们的应用都要有多种颜色，一种是正常的白天颜色，一种是晚上的黑夜颜色，还需要一种辅助的高对比颜色。这是微软建议的，一般应用都要包含的颜色。

我们还可以自己定义多种颜色，例如金属、海蓝之光、彩虹雨。然而微软给我们的切换，简单只有亮和暗。

那么问题就是我们如何切换我们的主题。

在这前，我们先说如何制作主题，其实主题就是Dictionary，我们在解决方案加上两个文件夹，一个是View，一个是ViewModel，其中View将会放主题，如果主题比较多，还可以在View加一个文件夹。

![这里写图片描述](http://img.blog.csdn.net/20160728161505206)

首先在View文件夹新建资源

![这里写图片描述](http://img.blog.csdn.net/20160728161516093)

我根据原文说的新建几个资源叫LightThemeDictionary、DarkThemeDictionary，一个是白天颜色，一个是黑暗

然后我们在我们的资源写入几个资源

```
<ResourceDictionary
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation" 
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:NightDayThemeToggleButton.View">
    <SolidColorBrush x:Key="SystemBackgroundAltHighBrush" Color="#FFF0F0F0"/>
    <SolidColorBrush x:Key="SystemBackgroundBaseHighBrush" Color="#FF101010"/>
    <Color x:Key="SystemTranslucentBaseHighColor">#FF000000</Color>
    <Color x:Key="SystemThemeMainColor">#FF0074CE</Color>
</ResourceDictionary>
```

然后在黑暗也写相同key的资源

```
<ResourceDictionary
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation" 
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:NightDayThemeToggleButton.View">
    
    <SolidColorBrush x:Key="SystemBackgroundAltHighBrush" Color="#FF1A1C37"/>
    <SolidColorBrush x:Key="SystemBackgroundBaseHighBrush" Color="#FFDFDFDF"/>
    <Color x:Key="SystemTranslucentBaseHighColor">#FFFFFFFF</Color>
    <Color x:Key="SystemThemeMainColor">#FF0074CE</Color>
</ResourceDictionary>

```

然后我们需要在前台把资源放在Page

```
    <Page.Resources>
        <ResourceDictionary>
            <ResourceDictionary.ThemeDictionaries>
                <ResourceDictionary x:Key="Light" Source="View/DarkThemeDictionary.xaml"></ResourceDictionary>
                <ResourceDictionary x:Key="Dark" Source="View/LightThemeDictionary.xaml"></ResourceDictionary>
            </ResourceDictionary.ThemeDictionaries>
        </ResourceDictionary>
    </Page.Resources>
```

我们使用资源需要ThemeDictionaries，这个是主题

记住要把资源一个叫`x:Key="Light"`一个Dark，原因在下面会说。

我们建立ViewModel，其中ViewModel继承NotifyProperty，这是一个我写的类，这个类主要是INotifyPropertyChanged，如果自己写ViewModel也好

ViewModel建立在ViewModel文件夹，一般少把类名称和文件夹一样

我们ViewModel主要是属性`ElementTheme Theme`，ElementTheme 有Default，Light，Dark，就是我们要把key叫light和dark，这样就可以绑定ViewModel修改

viewModel

```
    public class ViewModel : NotifyProperty
    {
        public ViewModel()
        {
        }

        public ElementTheme Theme
        {
            get
            {
                return _theme;
            }
            set
            {
                _theme = value;
                OnPropertyChanged();
            }
        }

        private ElementTheme _theme = ElementTheme.Light;
    }
```

我们绑定Page.RequestedTheme

先在xaml.cs写

```
 private ViewModel.ViewModel View { set; get; }=new ViewModel.ViewModel();
```

然后在xaml

```
<Page
    x:Class="NightDayThemeToggleButton.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:NightDayThemeToggleButton"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d"
    RequestedTheme="{x:Bind View.Theme,Mode=OneWay}">
```

我们要看到变化，在xaml使用

```
    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <Grid Background="{ThemeResource SystemBackgroundAltHighBrush}"> 
           <ToggleSwitch HorizontalAlignment="Center" Toggled="ToggleSwitch_OnToggled"></ToggleSwitch>
       </Grid>
    </Grid>
```

SystemBackgroundAltHighBrush是我们两个资源的，其中一个是白天，一个不是

```
        private void ToggleSwitch_OnToggled(object sender, RoutedEventArgs e)
        {
            View.Theme = View.Theme == ElementTheme.Light ? ElementTheme.Dark :
                ElementTheme.Light;
        }
```

运行可以看到点击就变成白天颜色，再点击就变为黑暗，这就是uwp切换主题，这样主题颜色很少，只有两个。

参见：https://embracez.xyz/xaml-uwp-themes/


我们总是会使用白天，夜间模式，那么我们需要切换主题，UWP切换主题简单

下面使用我做的一个按钮

夜间白天主题按钮

NightDayThemeToggleButton

我做的还有游戏键，这些都是可以简单使用的控件

这些控件放在https://github.com/lindexi/UWP，大家可以拿下来用。

做一个按钮，其实是修改

```
 <Style x:Key="NightDayThemeToggleButton" TargetType="CheckBox">
            <Setter Property="Background" Value="Transparent"/>
            <Setter Property="Foreground" Value="{ThemeResource SystemControlForegroundBaseHighBrush}"/>
            <Setter Property="Padding" Value="8,5,0,0"/>
            <Setter Property="HorizontalAlignment" Value="Left"/>
            <Setter Property="VerticalAlignment" Value="Center"/>
            <Setter Property="HorizontalContentAlignment" Value="Left"/>
            <Setter Property="VerticalContentAlignment" Value="Top"/>
            <Setter Property="FontFamily" Value="{ThemeResource ContentControlThemeFontFamily}"/>
            <Setter Property="FontSize" Value="{ThemeResource ControlContentThemeFontSize}"/>
            <Setter Property="MinWidth" Value="120"/>
            <Setter Property="MinHeight" Value="32"/>
            <Setter Property="UseSystemFocusVisuals" Value="True"/>
            <Setter Property="Template">
                <Setter.Value>
                    <ControlTemplate TargetType="CheckBox">
                        <Grid BorderBrush="{TemplateBinding BorderBrush}" BorderThickness="{TemplateBinding BorderThickness}" Background="{TemplateBinding Background}">
                            <VisualStateManager.VisualStateGroups>
                                <VisualStateGroup x:Name="CombinedStates">
                                    <VisualState x:Name="UncheckedNormal">
                                        <Storyboard>
                                            <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Light" Storyboard.TargetProperty="Opacity">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="1"></DiscreteObjectKeyFrame>
                                            </ObjectAnimationUsingKeyFrames>
                                            <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Dark" Storyboard.TargetProperty="Opacity">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="0"></DiscreteObjectKeyFrame>
                                            </ObjectAnimationUsingKeyFrames>
                                        </Storyboard>
                                    </VisualState>
                                    <VisualState x:Name="UncheckedPointerOver">
                                        <Storyboard>
                                            <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Light" Storyboard.TargetProperty="Opacity">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="1"></DiscreteObjectKeyFrame>
                                            </ObjectAnimationUsingKeyFrames>
                                            <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Dark" Storyboard.TargetProperty="Opacity">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="0"></DiscreteObjectKeyFrame>
                                            </ObjectAnimationUsingKeyFrames>
                                        </Storyboard>
                                    </VisualState>
                                    <VisualState x:Name="UncheckedPressed">
                                        <Storyboard>
                                            <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Light" Storyboard.TargetProperty="Opacity">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="1"></DiscreteObjectKeyFrame>
                                            </ObjectAnimationUsingKeyFrames>
                                            <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Dark" Storyboard.TargetProperty="Opacity">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="0"></DiscreteObjectKeyFrame>
                                            </ObjectAnimationUsingKeyFrames>
                                        </Storyboard>
                                    </VisualState>
                                    <VisualState x:Name="UncheckedDisabled">
                                        <Storyboard>
                                            <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Light" Storyboard.TargetProperty="Opacity">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="1"></DiscreteObjectKeyFrame>
                                            </ObjectAnimationUsingKeyFrames>
                                            <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Dark" Storyboard.TargetProperty="Opacity">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="0"></DiscreteObjectKeyFrame>
                                            </ObjectAnimationUsingKeyFrames>
                                        </Storyboard>
                                    </VisualState>
                                    <VisualState x:Name="CheckedNormal">
                                        <Storyboard>
                                            <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Light" Storyboard.TargetProperty="Opacity">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="0"></DiscreteObjectKeyFrame>
                                            </ObjectAnimationUsingKeyFrames>
                                            <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Dark" Storyboard.TargetProperty="Opacity">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="1"></DiscreteObjectKeyFrame>
                                            </ObjectAnimationUsingKeyFrames>
                                        </Storyboard>
                                    </VisualState>
                                    <VisualState x:Name="CheckedPointerOver">
                                        <Storyboard>
                                            <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Light" Storyboard.TargetProperty="Opacity">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="0"></DiscreteObjectKeyFrame>
                                            </ObjectAnimationUsingKeyFrames>
                                            <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Dark" Storyboard.TargetProperty="Opacity">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="1"></DiscreteObjectKeyFrame>
                                            </ObjectAnimationUsingKeyFrames>
                                        </Storyboard>
                                    </VisualState>
                                    <VisualState x:Name="CheckedPressed">
                                        <Storyboard>
                                            <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Light" Storyboard.TargetProperty="Opacity">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="0"></DiscreteObjectKeyFrame>
                                            </ObjectAnimationUsingKeyFrames>
                                            <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Dark" Storyboard.TargetProperty="Opacity">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="1"></DiscreteObjectKeyFrame>
                                            </ObjectAnimationUsingKeyFrames>
                                        </Storyboard>
                                    </VisualState>
                                    <VisualState x:Name="CheckedDisabled">
                                        <Storyboard>
                                            <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Light" Storyboard.TargetProperty="Opacity">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="0"></DiscreteObjectKeyFrame>
                                            </ObjectAnimationUsingKeyFrames>
                                            <ObjectAnimationUsingKeyFrames Storyboard.TargetName="Dark" Storyboard.TargetProperty="Opacity">
                                                <DiscreteObjectKeyFrame KeyTime="0" Value="1"></DiscreteObjectKeyFrame>
                                            </ObjectAnimationUsingKeyFrames>
                                        </Storyboard>
                                    </VisualState>
                                    <VisualState x:Name="IndeterminateNormal">
                                        <Storyboard>
                                           
                                        </Storyboard>
                                    </VisualState>
                                    <VisualState x:Name="IndeterminatePointerOver">
                                        <Storyboard>
                                          
                                        </Storyboard>
                                    </VisualState>
                                    <VisualState x:Name="IndeterminatePressed">
                                        <Storyboard>
                                          
                                        </Storyboard>
                                    </VisualState>
                                    <VisualState x:Name="IndeterminateDisabled">
                                        <Storyboard>
                                         
                                        </Storyboard>
                                    </VisualState>
                                </VisualStateGroup>
                            </VisualStateManager.VisualStateGroups>
                             <Image x:Name="Light" Source="Assets/weather_sun.png"></Image>
                            <Image x:Name="Dark" Source="Assets/weather_moon.png"></Image>
                          </Grid>
                    </ControlTemplate>
                </Setter.Value>
            </Setter>
        </Style>
```

需要使用也简单，可以使用

```
<CheckBox Margin="16,193,0,75" Style="{StaticResource NightDayThemeToggleButton}" IsChecked="{x:Bind AreChecked,Mode=TwoWay}"></CheckBox>
```

这样复制我上面代码就好，如果想用我的控件，可以

```
<local:NightDayThemeToggleButton ></local:NightDayThemeToggleButton>
```

上面用到两张图片，一张是白天，一张是夜晚

首先我们是编辑副本，不知道这个在哪的可以去看我的入门http://blog.csdn.net/lindexi_gd/article/details/52041944，里面有很多连接

然后我们可以看到

```
<VisualState x:Name="UncheckedNormal">
```

我们把下面自带所有控件都删掉，然后添加两个Image，当然需要给他们x:Name

接着在上面添加透明度从1到0或从0到1，大概就是这样做。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  


