
# win10 uwp 简单制作一个 Path 路径绘制的图标按钮

本文告诉大家在 UWP 或 WinUI 3 里面如何简单制作一个由 Path 几何路径图形绘制的图标按钮

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

先在资源里面定义按钮的样式，重写 Template 属性，通过在 Template 里面放入 Path 绑定 Data 到内容从而实现让 Path 显示集合路径图形，代码如下

```xml
        <Style x:Key="Style.TitlebarButton" TargetType="Button">
            <Setter Property="Background" Value="Transparent" />
            <Setter Property="Foreground" Value="#808080" />
            <Setter Property="HorizontalContentAlignment" Value="Left" />
            <Setter Property="VerticalContentAlignment" Value="Top" />
            <Setter Property="Width" Value="24"/>
            <Setter Property="Height" Value="24"/>
            <Setter Property="Template">
                <Setter.Value>
                    <ControlTemplate TargetType="ButtonBase">
                        <Grid Background="{TemplateBinding Background}">
                            <Path Fill="{TemplateBinding Foreground}" Data="{TemplateBinding Content}"></Path>
                        </Grid>
                    </ControlTemplate>
                </Setter.Value>
            </Setter>
        </Style>
```

接下来有路径资源可以先在资源字典里面定义，定义的是字符串即可，如以下代码

```xml
        <x:String x:Key="Geometry.Close">M18.363961,5.63603897 C18.7544853,6.02656326 18.7544853,6.65972824 18.363961,7.05025253 L13.4142136,12 L18.363961,16.9497475 C18.7544853,17.3402718 18.7544853,17.9734367 18.363961,18.363961 C17.9734367,18.7544853 17.3402718,18.7544853 16.9497475,18.363961 L12,13.4142136 L7.05025253,18.363961 C6.65972824,18.7544853 6.02656326,18.7544853 5.63603897,18.363961 C5.24551468,17.9734367 5.24551468,17.3402718 5.63603897,16.9497475 L10.5857864,12 L5.63603897,7.05025253 C5.24551468,6.65972824 5.24551468,6.02656326 5.63603897,5.63603897 C6.02656326,5.24551468 6.65972824,5.24551468 7.05025253,5.63603897 L12,10.5857864 L16.9497475,5.63603897 C17.3402718,5.24551468 17.9734367,5.24551468 18.363961,5.63603897 Z</x:String>
```

这里有一个细节点是在 UWP 或 WinUI 3 里，字符串类型应该使用 `x:String` 而不是使用 `system:String` 的方式，如以下错误的代码例子

```xml
<Page
    x:Class="LefernochihairWhemfawqarkemche.MainPage"
    ...
    xmlns:system="using:System">

    <Page.Resources>
        <system:String x:Key="Geometry.Close">M18.363961,5.63603897 C18.7544853,6.02656326 18.7544853,6.65972824 18.363961,7.05025253 L13.4142136,12 L18.363961,16.9497475 C18.7544853,17.3402718 18.7544853,17.9734367 18.363961,18.363961 C17.9734367,18.7544853 17.3402718,18.7544853 16.9497475,18.363961 L12,13.4142136 L7.05025253,18.363961 C6.65972824,18.7544853 6.02656326,18.7544853 5.63603897,18.363961 C5.24551468,17.9734367 5.24551468,17.3402718 5.63603897,16.9497475 L10.5857864,12 L5.63603897,7.05025253 C5.24551468,6.65972824 5.24551468,6.02656326 5.63603897,5.63603897 C6.02656326,5.24551468 6.65972824,5.24551468 7.05025253,5.63603897 L12,10.5857864 L16.9497475,5.63603897 C17.3402718,5.24551468 17.9734367,5.24551468 18.363961,5.63603897 Z</system:String>
    </Page.Resources>
    <Grid>
    </Grid>
</Page>
```

以上的代码可能抛出的是 `Microsoft.UI.Xaml.Markup.ReflectionHelperException Error in reflection helper.  Please add '<PropertyGroup><EnableTypeInfoReflection>false</EnableTypeInfoReflection></PropertyGroup>' to your project file..  Created Xaml type 'String' has a different name than requested type 'System.String'` 错误，也可能抛出的是 `Windows.UI.Xaml.Markup.XamlParseException: XAML parsing failed.` 异常。这几个异常这么奇怪，其实是微软从 2015 开始就毫无长进的 WinUI 异常提示机制，由于经过了 COM 的 WinUI 底层，导致了上层抛出的不是本质的异常，也不知道是哪一行，只能依靠逐步静态阅读代码和不断运行尝试才能知道是哪里写错了

回到使用代码里面，图标按钮的使用方法特别简单，只需要将以上的 `x:String` 的几何路径设置到按钮的内容，然后设置按钮的样式就完成

```xml
        <Button Style="{StaticResource Style.TitlebarButton}" Content="{StaticResource Geometry.Close}"></Button>
```

如此简单即可完成图标按钮

为了防止大家不知道上文给的代码是写到哪里，下面给出页面的代码，可以拷贝在自己的项目里了解效果

```xml
<Page
    x:Class="LefernochihairWhemfawqarkemche.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:LefernochihairWhemfawqarkemche"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:system="using:System"
    xmlns:helpers="using:LefernochihairWhemfawqarkemche.Helpers"
    mc:Ignorable="d"
    Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">

    <Page.Resources>
        <x:String x:Key="Geometry.Close">M18.363961,5.63603897 C18.7544853,6.02656326 18.7544853,6.65972824 18.363961,7.05025253 L13.4142136,12 L18.363961,16.9497475 C18.7544853,17.3402718 18.7544853,17.9734367 18.363961,18.363961 C17.9734367,18.7544853 17.3402718,18.7544853 16.9497475,18.363961 L12,13.4142136 L7.05025253,18.363961 C6.65972824,18.7544853 6.02656326,18.7544853 5.63603897,18.363961 C5.24551468,17.9734367 5.24551468,17.3402718 5.63603897,16.9497475 L10.5857864,12 L5.63603897,7.05025253 C5.24551468,6.65972824 5.24551468,6.02656326 5.63603897,5.63603897 C6.02656326,5.24551468 6.65972824,5.24551468 7.05025253,5.63603897 L12,10.5857864 L16.9497475,5.63603897 C17.3402718,5.24551468 17.9734367,5.24551468 18.363961,5.63603897 Z</x:String>

        <Style x:Key="Style.TitlebarButton" TargetType="Button">
            <Setter Property="Background" Value="Transparent" />
            <Setter Property="Foreground" Value="#808080" />
            <Setter Property="HorizontalContentAlignment" Value="Left" />
            <Setter Property="VerticalContentAlignment" Value="Top" />
            <Setter Property="Width" Value="24"/>
            <Setter Property="Height" Value="24"/>
            <Setter Property="Template">
                <Setter.Value>
                    <ControlTemplate TargetType="ButtonBase">
                        <Grid Background="{TemplateBinding Background}">
                            <Path Fill="{TemplateBinding Foreground}" Data="{TemplateBinding Content}"></Path>
                        </Grid>
                    </ControlTemplate>
                </Setter.Value>
            </Setter>
        </Style>
    </Page.Resources>
    <Grid>
        <Button Style="{StaticResource Style.TitlebarButton}" Content="{StaticResource Geometry.Close}" Click="Button_OnClick"></Button>
    </Grid>
</Page>
```





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。