# WPF 修改按钮按下的颜色

本文告诉大家如何使用附加属性修改按钮按下去时的背景

<!--more-->
<!-- CreateTime:2018/8/10 19:16:53 -->


先让大家看个图片，下面来告诉大家如何做

![](http://image.acmx.xyz/65fb6078-c169-4ce3-cdd9-e35752d07be0%2F%25E7%2582%25B9%25E5%2587%25BB%25E4%25BF%25AE%25E6%2594%25B9%25E6%258C%2589%25E9%2592%25AE.gif)

首先在后台创建一个附加属性

```csharp
    public class ButtonBrush
    {
        public static readonly DependencyProperty ButtonPressBackgroundProperty = DependencyProperty.RegisterAttached(
            "ButtonPressBackground", typeof(Brush), typeof(ButtonBrush), new PropertyMetadata(default(Brush)));

        public static void SetButtonPressBackground(DependencyObject element, Brush value)
        {
            element.SetValue(ButtonPressBackgroundProperty, value);
        }

        public static Brush GetButtonPressBackground(DependencyObject element)
        {
            return (Brush) element.GetValue(ButtonPressBackgroundProperty);
        }
    }

```

然后在 xaml 使用附加属性

```csharp
        <Button Margin="10,10,10,10" 
                Width="300" Height="100"
                Content="确定"
                local:ButtonBrush.ButtonPressBackground="#FFfcac1c" />
```

如何在按钮按下时使用这个附加属性修改按钮颜色？实际重写按钮的样式可以看到，在按下时可以修改颜色

```csharp
        <Style x:Key="Style.OkOperationButton"
               TargetType="ButtonBase">
            <Setter Property="Width" Value="110" />
            <Setter Property="Height" Value="44" />
            <Setter Property="FontSize" Value="24" />
            <Setter Property="Background" Value="#FF0087FF" />
            <Setter Property="HorizontalContentAlignment" Value="Center" />
            <Setter Property="VerticalContentAlignment" Value="Center" />
            <Setter Property="Template">
                <Setter.Value>
                    <ControlTemplate TargetType="{x:Type ButtonBase}">
                        <Border x:Name="Border" Width="{TemplateBinding Width}"
                                Height="{TemplateBinding Height}"
                                CornerRadius="22" Background="{TemplateBinding Background}">
                            <TextBlock x:Name="TextBlock"
                                       Text="{TemplateBinding Content}"
                                       FontSize="{TemplateBinding FontSize}"
                                       HorizontalAlignment="{TemplateBinding HorizontalContentAlignment}"
                                       VerticalAlignment="{TemplateBinding VerticalContentAlignment}" />
                        </Border>

                        <ControlTemplate.Triggers>
                            <Trigger Property="IsPressed" Value="True">
                                <Setter Property="Background"
                                        Value="#FFfcac1c" />
                                <Setter TargetName="TextBlock" Property="Foreground" Value="#FFFFFFFF" />
                            </Trigger>
                            <Trigger Property="IsEnabled" Value="False">
                                <Setter TargetName="Border" Property="Background" Value="#4D0087FF" />
                                <Setter TargetName="TextBlock" Property="Foreground" Value="#4DFFFFFF" />
                            </Trigger>
                        </ControlTemplate.Triggers>
                    </ControlTemplate>
                </Setter.Value>
            </Setter>
        </Style>

```

那么如何在设置使用附加属性，实际上使用下面的代码直接从按钮获取附加属性

```csharp
                  <Trigger Property="IsPressed" Value="True">
                                <Setter Property="Background"
                                        Value="{Binding RelativeSource = {RelativeSource Self},Path=(local:ButtonBrush.ButtonPressBackground)}" />
                                <Setter TargetName="TextBlock" Property="Foreground" Value="#FFFFFFFF" />
                            </Trigger>
```

所有的代码

```csharp
 <Window.Resources>

        <Style x:Key="Style.OkOperationButton"
               TargetType="ButtonBase">
            <Setter Property="Width" Value="110" />
            <Setter Property="Height" Value="44" />
            <Setter Property="FontSize" Value="24" />
            <Setter Property="Background" Value="#FF0087FF" />
            <Setter Property="HorizontalContentAlignment" Value="Center" />
            <Setter Property="VerticalContentAlignment" Value="Center" />
            <Setter Property="Template">
                <Setter.Value>
                    <ControlTemplate TargetType="{x:Type ButtonBase}">
                        <Border x:Name="Border" Width="{TemplateBinding Width}"
                                Height="{TemplateBinding Height}"
                                CornerRadius="22" Background="{TemplateBinding Background}">
                            <TextBlock x:Name="TextBlock"
                                       Text="{TemplateBinding Content}"
                                       FontSize="{TemplateBinding FontSize}"
                                       HorizontalAlignment="{TemplateBinding HorizontalContentAlignment}"
                                       VerticalAlignment="{TemplateBinding VerticalContentAlignment}" />
                        </Border>

                        <ControlTemplate.Triggers>
                            <Trigger Property="IsPressed" Value="True">
                                <Setter Property="Background"
                                        Value="{Binding RelativeSource = {RelativeSource Self},Path=(local:ButtonBrush.ButtonPressBackground)}" />
                                <Setter TargetName="TextBlock" Property="Foreground" Value="#FFFFFFFF" />
                            </Trigger>
                            <Trigger Property="IsEnabled" Value="False">
                                <Setter TargetName="Border" Property="Background" Value="#4D0087FF" />
                                <Setter TargetName="TextBlock" Property="Foreground" Value="#4DFFFFFF" />
                            </Trigger>
                        </ControlTemplate.Triggers>
                    </ControlTemplate>
                </Setter.Value>
            </Setter>
        </Style>

    </Window.Resources>
    <Grid>
        <Button Margin="10,10,10,10" Style="{StaticResource Style.OkOperationButton}"
                Width="300" Height="100"
                Content="确定"
                local:ButtonBrush.ButtonPressBackground="#FFfcac1c" />
    </Grid>
```

代码：[下载](http://lindexi.ml:8080/index.php/s/OcKrBbMufXoJAXW)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。