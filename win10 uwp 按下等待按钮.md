# win10 uwp 按下等待按钮

我们经常需要一个按钮，在按下时，后台执行Task，这时不能再次按下按钮。

![](http://7xqpl8.com1.z0.glb.clouddn.com/be842536-5c96-47f4-a49d-354e749a826aProgressButton.gif)

<!-- more -->

我们使用自定义控件，首先新建一个类，我把它命名是ProgressButton

一个进度条按钮，也就是我们按下时发生进度条，完成时他又是按钮。

我们需要一个值让我们知道是不是已经完成了后台，按钮可以按下，在按下时，自动让按钮IsEnable为false。

我们需要模板有TextBlock，显示文字，ProgressRing显示进度条。

于是我们使用TemplatePart

		
```csharp
    [TemplatePart(Name = "TextBlock", Type = typeof(TextBlock))]
    [TemplatePart(Name = "Progress", Type = typeof(Windows.UI.Xaml.Controls.ProgressRing))]
    public class ProgressButton : Windows.UI.Xaml.Controls.Button

```

依赖属性其实很简单，我们需要在VS上大propdp 按Tab 就可以看到vs帮我们写的依赖属性。

我们需要修改属性名称，属性类型，默认值。

我这里的Text ，需要他修改时使用函数，这个叫CallBack。

依赖函数使用`DependencyProperty.Register`

他参数： name 是 属性名,  propertyType 是属性类型, ownerType 是属于的类的类型,  typeMetadata 是默认值和修改时使用函数

我们来说下 typeMetadata 

typeMetadata 可以传入一个默认值，这个值就是我们不在依赖属性赋值，就给他一个默认的值。然后我们还可以给他一个在属性修改时使用的函数。

注意我们给他的函数不是必需，一般都不需要。

如果需要给他一个函数，这个函数需要有参数`DependencyObject sender, DependencyPropertyChangedEventArgs e`

其中 sender 是发送的实例，我们知道属性属于哪个类，我在这里，是属于ProgressButton ，我就可以使用 `ProgressButton button = sender as ProgressButton;`得到类，注意我们写的函数是静态的，所以sender才有用，我们可以使用sender获得类的属性

e 是有 NewValue 和 OldValue ， NewValue 是我们要修改的值， OldValue 是原来的值。

大概需要的依赖属性在我们这个控件有 Text Complete 就没了。

Text是我们按钮的文字，Complete 是我们的后台是不是在执行，如果是的话，按钮就无法点击，显示进度条。

代码：

		
```csharp
using Windows.UI.Xaml;
using Windows.UI.Xaml.Controls;

namespace lindexi.uwp.control.Button.Control
{
    [TemplatePart(Name = "TextBlock", Type = typeof(TextBlock))]
    [TemplatePart(Name = "Progress", Type = typeof(Windows.UI.Xaml.Controls.ProgressRing))]
    public class ProgressButton : Windows.UI.Xaml.Controls.Button
    {
        public ProgressButton()
        {
            DefaultStyleKey = typeof(ProgressButton);
            Click += ProgressButton_Click;
        }

        private void ProgressButton_Click(object sender, Windows.UI.Xaml.RoutedEventArgs e)
        {
            Complete = false;
        }

        private Windows.UI.Xaml.Controls.TextBlock _textBlock;

        private Windows.UI.Xaml.Controls.ProgressRing _proress;

        public string Text
        {
            get { return (string)GetValue(TextProperty); }
            set { SetValue(TextProperty, value); }
        }

        // Using a DependencyProperty as the backing store for Text.  This enables animation, styling, binding, etc...
        public static readonly DependencyProperty TextProperty =
            DependencyProperty.Register("Text", typeof(string), typeof(ProgressButton), new PropertyMetadata("",
                (d, e) =>
                {
                    ProgressButton temp = d as ProgressButton;
                    if (temp == null)
                    {
                        return;
                    }
                    if(temp._textBlock!=null)
                    {
                        temp._textBlock.Text = (string) e.NewValue;
                    }
                }));

        

        public bool Complete
        {
            get { return (bool)GetValue(CompleteProperty); }
            set { SetValue(CompleteProperty, value); }
        }

        // Using a DependencyProperty as the backing store for Complete.  This enables animation, styling, binding, etc...
        public static readonly DependencyProperty CompleteProperty =
            DependencyProperty.Register("Complete", typeof(bool), typeof(ProgressButton), new PropertyMetadata(true,
                OnComplete));

        private static void OnComplete(DependencyObject d, DependencyPropertyChangedEventArgs e)
        {
            ProgressButton button = d as ProgressButton;
            if (button == null)
            {
                return;
            }
            
            bool temp = (bool)e.NewValue;


            //button._textBlock.Visibility = temp ? Visibility.Visible : Visibility.Collapsed;
            button._proress.Visibility = temp ? Visibility.Collapsed : Visibility.Visible;
            button.IsEnabled = temp;
        }



        protected override void OnApplyTemplate()
        {
            base.OnApplyTemplate();
            _textBlock = GetTemplateChild("TextBlock") as Windows.UI.Xaml.Controls.TextBlock;
            _proress = GetTemplateChild("Progress") as Windows.UI.Xaml.Controls.ProgressRing;

            if (_textBlock != null)
            {
                _textBlock.Visibility = Visibility.Visible;
                _textBlock.Text = Text;
            }

            if (_proress != null)
            {
                _proress.Visibility=Visibility.Collapsed;
            }
        }
    }
}


```

我们在控件 OnApplyTemplate 拿到 _textBlock _proress 我们需要写一个Style

		
```xml
<ResourceDictionary
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation" 
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:control="using:lindexi.uwp.control.Button.Control">
    <Style TargetType="control:ProgressButton">
        <Setter Property="Background" Value="{ThemeResource ButtonBackground}"/>
        <Setter Property="Foreground" Value="{ThemeResource ButtonForeground}"/>
        <Setter Property="BorderBrush" Value="{ThemeResource ButtonBorderBrush}"/>
        <Setter Property="BorderThickness" Value="{ThemeResource ButtonBorderThemeThickness}"/>
        <Setter Property="Padding" Value="8,4,8,4"/>
        <Setter Property="HorizontalAlignment" Value="Left"/>
        <Setter Property="VerticalAlignment" Value="Center"/>
        <Setter Property="FontFamily" Value="{ThemeResource ContentControlThemeFontFamily}"/>
        <Setter Property="FontWeight" Value="Normal"/>
        <Setter Property="FontSize" Value="{ThemeResource ControlContentThemeFontSize}"/>
        <Setter Property="UseSystemFocusVisuals" Value="True"/>
        <Setter Property="FocusVisualMargin" Value="-3"/>
        <Setter Property="Template">
            <Setter.Value>
                <ControlTemplate TargetType="control:ProgressButton">
                    <Grid x:Name="RootGrid" Background="{TemplateBinding Background}">
                        <VisualStateManager.VisualStateGroups>
                            <VisualStateGroup x:Name="CommonStates">
                                <VisualState x:Name="Normal">
                                    <Storyboard>
                                        <PointerUpThemeAnimation Storyboard.TargetName="RootGrid"/>
                                    </Storyboard>
                                </VisualState>
                                <VisualState x:Name="PointerOver">
                                    <Storyboard>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="Background" Storyboard.TargetName="RootGrid">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{ThemeResource ButtonBackgroundPointerOver}"/>
                                        </ObjectAnimationUsingKeyFrames>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="BorderBrush" Storyboard.TargetName="ContentPresenter">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{ThemeResource ButtonBorderBrushPointerOver}"/>
                                        </ObjectAnimationUsingKeyFrames>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="Foreground" Storyboard.TargetName="ContentPresenter">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{ThemeResource ButtonForegroundPointerOver}"/>
                                        </ObjectAnimationUsingKeyFrames>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="TextBlock" Storyboard.TargetProperty="Foreground">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{ThemeResource ButtonForegroundPointerOver}"></DiscreteObjectKeyFrame>
                                        </ObjectAnimationUsingKeyFrames>
                                        <PointerUpThemeAnimation Storyboard.TargetName="RootGrid"/>
                                    </Storyboard>
                                </VisualState>
                                <VisualState x:Name="Pressed">
                                    <Storyboard>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="Background" Storyboard.TargetName="RootGrid">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{ThemeResource ButtonBackgroundPressed}"/>
                                        </ObjectAnimationUsingKeyFrames>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="BorderBrush" Storyboard.TargetName="ContentPresenter">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{ThemeResource ButtonBorderBrushPressed}"/>
                                        </ObjectAnimationUsingKeyFrames>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="Foreground" Storyboard.TargetName="ContentPresenter">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{ThemeResource ButtonForegroundPressed}"/>
                                        </ObjectAnimationUsingKeyFrames>
                                        <PointerDownThemeAnimation Storyboard.TargetName="RootGrid"/>
                                    </Storyboard>
                                </VisualState>
                                <VisualState x:Name="Disabled">
                                    <Storyboard>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="Background" Storyboard.TargetName="RootGrid">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{ThemeResource ButtonBackgroundDisabled}"/>
                                        </ObjectAnimationUsingKeyFrames>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="BorderBrush" Storyboard.TargetName="ContentPresenter">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{ThemeResource ButtonBorderBrushDisabled}"/>
                                        </ObjectAnimationUsingKeyFrames>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="Foreground" Storyboard.TargetName="ContentPresenter">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{ThemeResource ButtonForegroundDisabled}"/>
                                        </ObjectAnimationUsingKeyFrames>
                                        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="TextBlock" Storyboard.TargetProperty="Foreground">
                                            <DiscreteObjectKeyFrame KeyTime="0" Value="{ThemeResource ButtonForegroundDisabled}"></DiscreteObjectKeyFrame>
                                        </ObjectAnimationUsingKeyFrames>
                                    </Storyboard>
                                </VisualState>
                            </VisualStateGroup>
                        </VisualStateManager.VisualStateGroups>
                        <ContentPresenter x:Name="ContentPresenter" 
                                          AutomationProperties.AccessibilityView="Raw" 
                                          BorderBrush="{TemplateBinding BorderBrush}" 
                                          BorderThickness="{TemplateBinding BorderThickness}"
                                          ContentTemplate="{TemplateBinding ContentTemplate}" 
                                          ContentTransitions="{TemplateBinding ContentTransitions}"
                                          Content="{TemplateBinding Content}" 
                                          HorizontalContentAlignment="{TemplateBinding HorizontalContentAlignment}" 
                                          Padding="{TemplateBinding Padding}" 
                                          VerticalContentAlignment="{TemplateBinding VerticalContentAlignment}"/>
                        <TextBlock x:Name="TextBlock" Margin="10,10,10,10"
                                   HorizontalAlignment="Center"
                                   VerticalAlignment="Center"></TextBlock>
                        <ProgressRing x:Name="Progress" IsActive="True"></ProgressRing>
                   
                    </Grid>
                </ControlTemplate>
            </Setter.Value>
        </Setter>
    </Style>
</ResourceDictionary>


```

这个是从Button复制，就改了Button为 `control:ProgressButton` 

我们要使用按钮，需要在资源写
		
```xml
    <Page.Resources>
        <ResourceDictionary Source="Control/ProgressButton.xaml"></ResourceDictionary>
    </Page.Resources>

```

然后就可以使用 ProgressButton ,我写ProgressButton在control文件夹，我需要在命名空间`xmlns:control="using:lindexi.uwp.control.Button.Control"`

		
```csharp
 <control:ProgressButton Text="确定"
                                 Complete="{x:Bind View.Complete,Mode=TwoWay}"
                                 Click="ButtonBase_OnClick"></control:ProgressButton>

```
我上面是测试，点击是进行100秒，过了就完成，代码简单，如果想知道，戳此链接 https://github.com/lindexi/UWP/tree/master/uwp/control/Button

那么如果我们有好多个页面都用到 ProgressButton ，我们需要在所有页面都写 ResourceDictionary 这样不好，我们有一个简单方法，让页面不用写这个。

在解决方案新建一个文件夹Themes，注意命名一定是Themes，注意有个名称后面有个s，我就在这坑好多天了。

然后新建资源字典 Generic.xaml ，注意名称也是不能自己修改。

在 Generic.xaml 合并字典

		
```xml
    <ResourceDictionary.MergedDictionaries>
        <ResourceDictionary Source="ms-appx:///Control/ProgressButton.xaml"></ResourceDictionary>
    </ResourceDictionary.MergedDictionaries>

```

这样我们就可以在页面直接用。

如果使用遇到问题，欢迎讨论。

参见：http://www.cnblogs.com/ms-uap/p/5520872.html

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
