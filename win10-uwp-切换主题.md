本文主要说如何在UWP切换主题，并且如何制作主题。

一般我们的应用都要有多种颜色，一种是正常的白天颜色，一种是晚上的黑夜颜色，还需要一种辅助的高对比颜色。这是微软建议的，一般应用都要包含的颜色。

我们还可以自己定义多种颜色，例如金属、海蓝之光、彩虹雨。然而微软给我们的切换，简单只有亮和暗。

那么问题就是我们如何切换我们的主题。

在这前，我们先说如何制作主题，其实主题就是Dictionary，我们在解决方案加上两个文件夹，一个是View，一个是ViewModel，其中View将会放主题，如果主题比较多，还可以在View加一个文件夹。

!\[这里写图片描述\]\(http:\/\/img.blog.csdn.net\/20160728161505206\)

首先在View文件夹新建资源

!\[这里写图片描述\]\(http:\/\/img.blog.csdn.net\/20160728161516093\)

我根据原文说的新建几个资源叫LightThemeDictionary、DarkThemeDictionary，一个是白天颜色，一个是黑暗

然后我们在我们的资源写入几个资源

\`\`\`

&lt;ResourceDictionary xmlns="http:\/\/schemas.microsoft.com\/winfx\/2006\/xaml\/presentation" xmlns:x="http:\/\/schemas.microsoft.com\/winfx\/2006\/xaml" xmlns:local="using:NightDayThemeToggleButton.View"&gt;

&lt;SolidColorBrush x:Key="SystemBackgroundAltHighBrush" Color="\#FFF0F0F0"\/&gt;

&lt;SolidColorBrush x:Key="SystemBackgroundBaseHighBrush" Color="\#FF101010"\/&gt; &lt;Color x:Key="SystemTranslucentBaseHighColor"&gt;\#FF000000&lt;\/Color&gt;

&lt;Color x:Key="SystemThemeMainColor"&gt;\#FF0074CE&lt;\/Color&gt;

&lt;\/ResourceDictionary&gt;

\`\`\`

然后在黑暗也写相同key的资源

\`\`\`&lt;ResourceDictionary xmlns="http:\/\/schemas.microsoft.com\/winfx\/2006\/xaml\/presentation" xmlns:x="http:\/\/schemas.microsoft.com\/winfx\/2006\/xaml" xmlns:local="using:NightDayThemeToggleButton.View"&gt; &lt;SolidColorBrush x:Key="SystemBackgroundAltHighBrush" Color="\#FF1A1C37"\/&gt; &lt;SolidColorBrush x:Key="SystemBackgroundBaseHighBrush" Color="\#FFDFDFDF"\/&gt; &lt;Color x:Key="SystemTranslucentBaseHighColor"&gt;\#FFFFFFFF&lt;\/Color&gt; &lt;Color x:Key="SystemThemeMainColor"&gt;\#FF0074CE&lt;\/Color&gt;&lt;\/ResourceDictionary&gt;

\`\`\`

然后我们需要在前台把资源放在Page



\`\`\` 

&lt;Page.Resources&gt; &lt;ResourceDictionary&gt; &lt;ResourceDictionary.ThemeDictionaries&gt; &lt;ResourceDictionary x:Key="Light" Source="View\/DarkThemeDictionary.xaml"&gt;&lt;\/ResourceDictionary&gt; &lt;ResourceDictionary x:Key="Dark" Source="View\/LightThemeDictionary.xaml"&gt;&lt;\/ResourceDictionary&gt; &lt;\/ResourceDictionary.ThemeDictionaries&gt; &lt;\/ResourceDictionary&gt; &lt;\/Page.Resources&gt;

\`\`\`

我们使用资源需要ThemeDictionaries，这个是主题

记住要把资源一个叫\`x:Key="Light"\`一个Dark，原因在下面会说。

我们建立ViewModel，其中ViewModel继承NotifyProperty，这是一个我写的类，这个类主要是INotifyPropertyChanged，如果自己写ViewModel也好

ViewModel建立在ViewModel文件夹，一般少把类名称和文件夹一样

我们ViewModel主要是属性\`ElementTheme Theme\`，ElementTheme 有Default，Light，Dark，就是我们要把key叫light和dark，这样就可以绑定ViewModel修改

viewModel



\`\`\`

 public class ViewModel : NotifyProperty { public ViewModel\(\) { }

public ElementTheme Theme { get { return \_theme; } set { \_theme = value; OnPropertyChanged\(\); } }

private ElementTheme \_theme = ElementTheme.Light; }

\`\`\`

我们绑定Page.RequestedTheme

先在xaml.cs写



\`\`\` 

private ViewModel.ViewModel View { set; get; }=new ViewModel.ViewModel\(\);

\`\`\`

然后在xaml



\`\`\`

&lt;Page x:Class="NightDayThemeToggleButton.MainPage" xmlns="http:\/\/schemas.microsoft.com\/winfx\/2006\/xaml\/presentation" xmlns:x="http:\/\/schemas.microsoft.com\/winfx\/2006\/xaml" xmlns:local="using:NightDayThemeToggleButton" xmlns:d="http:\/\/schemas.microsoft.com\/expression\/blend\/2008" xmlns:mc="http:\/\/schemas.openxmlformats.org\/markup-compatibility\/2006" mc:Ignorable="d" RequestedTheme="{x:Bind View.Theme,Mode=OneWay}"&gt;

\`\`\`

我们要看到变化，在xaml使用



\`\`\`

 &lt;Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}"&gt; &lt;Grid Background="{ThemeResource SystemBackgroundAltHighBrush}"&gt; &lt;ToggleSwitch HorizontalAlignment="Center" Toggled="ToggleSwitch\_OnToggled"&gt;&lt;\/ToggleSwitch&gt; &lt;\/Grid&gt; &lt;\/Grid&gt;

\`\`\`

SystemBackgroundAltHighBrush是我们两个资源的，其中一个是白天，一个不是



\`\`\` 

private void ToggleSwitch\_OnToggled\(object sender, RoutedEventArgs e\) { View.Theme = View.Theme == ElementTheme.Light ? ElementTheme.Dark : ElementTheme.Light; }

\`\`\`

运行可以看到点击就变成白天颜色，再点击就变为黑暗，这就是uwp切换主题，这样主题颜色很少，只有两个。

参见：https:\/\/embracez.xyz\/xaml-uwp-themes\/

我们总是会使用白天，夜间模式，那么我们需要切换主题，UWP切换主题简单

下面使用我做的一个按钮

夜间白天主题按钮

NightDayThemeToggleButton

我做的还有游戏键，这些都是可以简单使用的控件

这些控件放在https:\/\/github.com\/lindexi\/UWP，大家可以拿下来用。

做一个按钮，其实是修改

\`\`\` 

&lt;Style x:Key="NightDayThemeToggleButton" TargetType="CheckBox"&gt; &lt;Setter Property="Background" Value="Transparent"\/&gt; &lt;Setter Property="Foreground" Value="{ThemeResource SystemControlForegroundBaseHighBrush}"\/&gt; &lt;Setter Property="Padding" Value="8,5,0,0"\/&gt; &lt;Setter Property="HorizontalAlignment" Value="Left"\/&gt; &lt;Setter Property="VerticalAlignment" Value="Center"\/&gt; &lt;Setter Property="HorizontalContentAlignment" Value="Left"\/&gt; &lt;Setter Property="VerticalContentAlignment" Value="Top"\/&gt; &lt;Setter Property="FontFamily" Value="{ThemeResource ContentControlThemeFontFamily}"\/&gt; &lt;Setter Property="FontSize" Value="{ThemeResource ControlContentThemeFontSize}"\/&gt; &lt;Setter Property="MinWidth" Value="120"\/&gt; &lt;Setter Property="MinHeight" Value="32"\/&gt; &lt;Setter Property="UseSystemFocusVisuals" Value="True"\/&gt; &lt;Setter Property="Template"&gt; &lt;Setter.Value&gt; &lt;ControlTemplate TargetType="CheckBox"&gt; &lt;Grid BorderBrush="{TemplateBinding BorderBrush}" BorderThickness="{TemplateBinding BorderThickness}" Background="{TemplateBinding Background}"&gt; &lt;VisualStateManager.VisualStateGroups&gt; &lt;VisualStateGroup x:Name="CombinedStates"&gt; &lt;VisualState x:Name="UncheckedNormal"&gt; &lt;Storyboard&gt; &lt;ObjectAnimationUsingKeyFrames Storyboard.TargetName="Light" Storyboard.TargetProperty="Opacity"&gt; &lt;DiscreteObjectKeyFrame KeyTime="0" Value="1"&gt;&lt;\/DiscreteObjectKeyFrame&gt; &lt;\/ObjectAnimationUsingKeyFrames&gt; &lt;ObjectAnimationUsingKeyFrames Storyboard.TargetName="Dark" Storyboard.TargetProperty="Opacity"&gt; &lt;DiscreteObjectKeyFrame KeyTime="0" Value="0"&gt;&lt;\/DiscreteObjectKeyFrame&gt; &lt;\/ObjectAnimationUsingKeyFrames&gt; &lt;\/Storyboard&gt; &lt;\/VisualState&gt; &lt;VisualState x:Name="UncheckedPointerOver"&gt; &lt;Storyboard&gt; &lt;ObjectAnimationUsingKeyFrames Storyboard.TargetName="Light" Storyboard.TargetProperty="Opacity"&gt; &lt;DiscreteObjectKeyFrame KeyTime="0" Value="1"&gt;&lt;\/DiscreteObjectKeyFrame&gt; &lt;\/ObjectAnimationUsingKeyFrames&gt; &lt;ObjectAnimationUsingKeyFrames Storyboard.TargetName="Dark" Storyboard.TargetProperty="Opacity"&gt; &lt;DiscreteObjectKeyFrame KeyTime="0" Value="0"&gt;&lt;\/DiscreteObjectKeyFrame&gt; &lt;\/ObjectAnimationUsingKeyFrames&gt; &lt;\/Storyboard&gt; &lt;\/VisualState&gt; &lt;VisualState x:Name="UncheckedPressed"&gt; &lt;Storyboard&gt; &lt;ObjectAnimationUsingKeyFrames Storyboard.TargetName="Light" Storyboard.TargetProperty="Opacity"&gt; &lt;DiscreteObjectKeyFrame KeyTime="0" Value="1"&gt;&lt;\/DiscreteObjectKeyFrame&gt; &lt;\/ObjectAnimationUsingKeyFrames&gt; &lt;ObjectAnimationUsingKeyFrames Storyboard.TargetName="Dark" Storyboard.TargetProperty="Opacity"&gt; &lt;DiscreteObjectKeyFrame KeyTime="0" Value="0"&gt;&lt;\/DiscreteObjectKeyFrame&gt; &lt;\/ObjectAnimationUsingKeyFrames&gt; &lt;\/Storyboard&gt; &lt;\/VisualState&gt; &lt;VisualState x:Name="UncheckedDisabled"&gt; &lt;Storyboard&gt; &lt;ObjectAnimationUsingKeyFrames Storyboard.TargetName="Light" Storyboard.TargetProperty="Opacity"&gt; &lt;DiscreteObjectKeyFrame KeyTime="0" Value="1"&gt;&lt;\/DiscreteObjectKeyFrame&gt; &lt;\/ObjectAnimationUsingKeyFrames&gt; &lt;ObjectAnimationUsingKeyFrames Storyboard.TargetName="Dark" Storyboard.TargetProperty="Opacity"&gt; &lt;DiscreteObjectKeyFrame KeyTime="0" Value="0"&gt;&lt;\/DiscreteObjectKeyFrame&gt; &lt;\/ObjectAnimationUsingKeyFrames&gt; &lt;\/Storyboard&gt; &lt;\/VisualState&gt; &lt;VisualState x:Name="CheckedNormal"&gt; &lt;Storyboard&gt; &lt;ObjectAnimationUsingKeyFrames Storyboard.TargetName="Light" Storyboard.TargetProperty="Opacity"&gt; &lt;DiscreteObjectKeyFrame KeyTime="0" Value="0"&gt;&lt;\/DiscreteObjectKeyFrame&gt; &lt;\/ObjectAnimationUsingKeyFrames&gt; &lt;ObjectAnimationUsingKeyFrames Storyboard.TargetName="Dark" Storyboard.TargetProperty="Opacity"&gt; &lt;DiscreteObjectKeyFrame KeyTime="0" Value="1"&gt;&lt;\/DiscreteObjectKeyFrame&gt; &lt;\/ObjectAnimationUsingKeyFrames&gt; &lt;\/Storyboard&gt; &lt;\/VisualState&gt; &lt;VisualState x:Name="CheckedPointerOver"&gt; &lt;Storyboard&gt; &lt;ObjectAnimationUsingKeyFrames Storyboard.TargetName="Light" Storyboard.TargetProperty="Opacity"&gt; &lt;DiscreteObjectKeyFrame KeyTime="0" Value="0"&gt;&lt;\/DiscreteObjectKeyFrame&gt; &lt;\/ObjectAnimationUsingKeyFrames&gt; &lt;ObjectAnimationUsingKeyFrames Storyboard.TargetName="Dark" Storyboard.TargetProperty="Opacity"&gt; &lt;DiscreteObjectKeyFrame KeyTime="0" Value="1"&gt;&lt;\/DiscreteObjectKeyFrame&gt; &lt;\/ObjectAnimationUsingKeyFrames&gt; &lt;\/Storyboard&gt; &lt;\/VisualState&gt; &lt;VisualState x:Name="CheckedPressed"&gt; &lt;Storyboard&gt; &lt;ObjectAnimationUsingKeyFrames Storyboard.TargetName="Light" Storyboard.TargetProperty="Opacity"&gt; &lt;DiscreteObjectKeyFrame KeyTime="0" Value="0"&gt;&lt;\/DiscreteObjectKeyFrame&gt; &lt;\/ObjectAnimationUsingKeyFrames&gt; &lt;ObjectAnimationUsingKeyFrames Storyboard.TargetName="Dark" Storyboard.TargetProperty="Opacity"&gt; &lt;DiscreteObjectKeyFrame KeyTime="0" Value="1"&gt;&lt;\/DiscreteObjectKeyFrame&gt; &lt;\/ObjectAnimationUsingKeyFrames&gt; &lt;\/Storyboard&gt; &lt;\/VisualState&gt; &lt;VisualState x:Name="CheckedDisabled"&gt; &lt;Storyboard&gt; &lt;ObjectAnimationUsingKeyFrames Storyboard.TargetName="Light" Storyboard.TargetProperty="Opacity"&gt; &lt;DiscreteObjectKeyFrame KeyTime="0" Value="0"&gt;&lt;\/DiscreteObjectKeyFrame&gt; &lt;\/ObjectAnimationUsingKeyFrames&gt; &lt;ObjectAnimationUsingKeyFrames Storyboard.TargetName="Dark" Storyboard.TargetProperty="Opacity"&gt; &lt;DiscreteObjectKeyFrame KeyTime="0" Value="1"&gt;&lt;\/DiscreteObjectKeyFrame&gt; &lt;\/ObjectAnimationUsingKeyFrames&gt; &lt;\/Storyboard&gt; &lt;\/VisualState&gt; &lt;VisualState x:Name="IndeterminateNormal"&gt; &lt;Storyboard&gt; &lt;\/Storyboard&gt; &lt;\/VisualState&gt; &lt;VisualState x:Name="IndeterminatePointerOver"&gt; &lt;Storyboard&gt; &lt;\/Storyboard&gt; &lt;\/VisualState&gt; &lt;VisualState x:Name="IndeterminatePressed"&gt; &lt;Storyboard&gt; &lt;\/Storyboard&gt; &lt;\/VisualState&gt; &lt;VisualState x:Name="IndeterminateDisabled"&gt; &lt;Storyboard&gt; &lt;\/Storyboard&gt; &lt;\/VisualState&gt; &lt;\/VisualStateGroup&gt; &lt;\/VisualStateManager.VisualStateGroups&gt; &lt;Image x:Name="Light" Source="Assets\/weather\_sun.png"&gt;&lt;\/Image&gt; &lt;Image x:Name="Dark" Source="Assets\/weather\_moon.png"&gt;&lt;\/Image&gt; &lt;\/Grid&gt; &lt;\/ControlTemplate&gt; &lt;\/Setter.Value&gt; &lt;\/Setter&gt; &lt;\/Style&gt;

\`\`\`

需要使用也简单，可以使用

\`\`\`

&lt;CheckBox Margin="16,193,0,75" Style="{StaticResource NightDayThemeToggleButton}" IsChecked="{x:Bind AreChecked,Mode=TwoWay}"&gt;&lt;\/CheckBox&gt;

\`\`\`

这样复制我上面代码就好，如果想用我的控件，可以

\`\`\`

&lt;local:NightDayThemeToggleButton &gt;&lt;\/local:NightDayThemeToggleButton&gt;

\`\`\`

上面用到两张图片，一张是白天，一张是夜晚

首先我们是编辑副本，不知道这个在哪的可以去看我的入门http:\/\/blog.csdn.net\/lindexi\_gd\/article\/details\/52041944，里面有很多连接

然后我们可以看到

\`\`\`

&lt;VisualState x:Name="UncheckedNormal"&gt;

\`\`\`

我们把下面自带所有控件都删掉，然后添加两个Image，当然需要给他们x:Name

接着在上面添加透明度从1到0或从0到1，大概就是这样做。

&lt;a rel="license" href="http:\/\/creativecommons.org\/licenses\/by-nc-sa\/4.0\/"&gt;&lt;img alt="知识共享许可协议" style="border-width:0" src="https:\/\/i.creativecommons.org\/l\/by-nc-sa\/4.0\/88x31.png" \/&gt;&lt;\/a&gt;&lt;br \/&gt;本作品采用&lt;a rel="license" href="http:\/\/creativecommons.org\/licenses\/by-nc-sa\/4.0\/"&gt;知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议&lt;\/a&gt;进行许可。欢迎转载、使用、重新发布，但务必保留文章署名\[林德熙\]\(http:\/\/blog.csdn.net\/lindexi\_gd\)\(包含链接:http:\/\/blog.csdn.net\/lindexi\_gd \)，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我\[联系\]\(mailto:lindexi\_gd@163.com\)。

