# win10 uwp 提示 Cannot find a Resource with the Name Key 找不到资源

在写 UWP 界面如果没有写对资源的顺序，那么在加载到对应的界面会在提示上面信息

<!--more-->
<!-- CreateTime:2019/11/28 8:51:54 -->

<!-- csdn -->

在[堆栈](https://stackoverflow.com/q/59073577/6116637)小伙伴问了一个问题，在他的程序启动提示下面代码

```csharp
Windows.UI.Xaml.Markup.XamlParseException: 'The text associated with this error code could not be found.
Cannot find a Resource with the Name/Key ItemTemplateSelector [Line: 66 Position: 19]'
```

这个问题其实是 UWP 的 XAML 界面提示做的不好的原因，比较难简单从提示信息里面找到对应的问题

其实上面提示说的是在 66 行没有找到资源名叫 ItemTemplateSelector 的资源，那么 UWP 的资源是如何寻找的？在 UWP 将会通过顺序查找资源，按照当前所在的范围一直往上找，直到找到第一个资源。那么什么是按照当前所在的范围一直往上找，在 UWP 的界面布局是一棵树，将会从控件本身资源开始找，然后找控件的容器是否存在资源，如果找不到，就找控件的容器的容器的资源

但是除了上面的规则，还有一个规则就是按照代码写的上下顺序找，也就是资源需要写到寻找资源的代码之前。小伙伴的代码有点多，我将代码放在[github](https://github.com/lindexi/lindexi_gd/tree/b2f8e1c8ddb5a2d2643a3e97b27140de1da7304e/LawhallwheachalNakearjalle)就不再这里贴细节

```csharp
    <Grid>
       <GridView HorizontalAlignment="Stretch" VerticalAlignment="Stretch"
          Margin="80, 40, 60, 40" BorderThickness="0"
          ItemTemplateSelector="{StaticResource ItemTemplateSelector}">
            <GridView.ItemContainerStyle>
                <Style TargetType="GridViewItem">
                    <Setter Property="Margin" Value="0, 0, 0, 32"/>
                </Style>
            </GridView.ItemContainerStyle>
        </GridView>
    </Grid>

    <Page.Resources>
        <local:ItemTemplateSelector x:Key="ItemTemplateSelector"
                                   Template1="{StaticResource Template1}"
                                   Template2="{StaticResource Template2}">

        </local:ItemTemplateSelector>

        <DataTemplate x:Key="Template1" >
            <local:Template1 HorizontalAlignment="Stretch" VerticalAlignment="Stretch"></local:Template1>
        </DataTemplate>
        <DataTemplate x:Key="Template2"  >
            <local:Template2 HorizontalAlignment="Stretch" VerticalAlignment="Stretch"></local:Template2>
        </DataTemplate>
    </Page.Resources>
```

这里 `ItemTemplateSelector="{StaticResource ItemTemplateSelector}"` 是第66行，也就是 ItemTemplateSelector 这个资源找不到，在上面代码可以看到在 Page.Resources 里面有定义，为什么会找不到。按照第二个规则需要在使用资源之前，也就是需要将页面定义放在最前

```
    <Page.Resources>
        <DataTemplate x:Key="Template1" >
            <local:Template1 HorizontalAlignment="Stretch" VerticalAlignment="Stretch"></local:Template1>
        </DataTemplate>
        <DataTemplate x:Key="Template2"  >
            <local:Template2 HorizontalAlignment="Stretch" VerticalAlignment="Stretch"></local:Template2>
        </DataTemplate>

        <local:ItemTemplateSelector x:Key="ItemTemplateSelector"
                                   Template1="{StaticResource Template1}"
                                   Template2="{StaticResource Template2}">

        </local:ItemTemplateSelector>
    </Page.Resources>

    <Grid>
        <GridView HorizontalAlignment="Stretch" VerticalAlignment="Stretch"
          Margin="80, 40, 60, 40" BorderThickness="0"
          ItemTemplateSelector="{StaticResource ItemTemplateSelector}">
            <GridView.ItemContainerStyle>
                <Style TargetType="GridViewItem">
                    <Setter Property="Margin" Value="0, 0, 0, 32"/>
                </Style>
            </GridView.ItemContainerStyle>
        </GridView>
    </Grid>
```

修改的代码放在 [github](https://github.com/lindexi/lindexi_gd/commit/ca49c76909fca81fb6518247a732219bb5f0d9a6) 欢迎小伙伴访问

如果看到在 UWP 提示下面代码，那么应该就是找不到资源，找不到资源可能的原因是资源名写错了，或者资源定义在使用后或者从这个控件往上找不到这个资源

```csharp
无法找到与此错误代码关联的文本。

Cannot find a Resource with the Name/Key ItemTemplateSelector [Line: 16 Position: 11]
```

提示这个代码的堆栈如下

```csharp
   在 Windows.UI.Xaml.Application.LoadComponent(Object component, Uri resourceLocator, ComponentResourceLocation componentResourceLocation)
   在 LawhallwheachalNakearjalle.MainPage.InitializeComponent()
   在 LawhallwheachalNakearjalle.MainPage..ctor() 
```

请看下面代码，在 Grid 用到了没有定义的 Foo 资源

```xml
    <Page.Resources>
     
    </Page.Resources>

    <Grid Background="{StaticResource Foo}">
     
    </Grid>
```

此时将提示下面代码

```
System.Reflection.TargetInvocationException: Exception has been thrown by the target of an invocation. ---> Windows.UI.Xaml.Markup.XamlParseException: 无法找到与此错误代码关联的文本。

Cannot find a Resource with the Name/Key Foo [Line: 15 Position: 11]
   at Windows.UI.Xaml.Application.LoadComponent(Object component, Uri resourceLocator, ComponentResourceLocation componentResourceLocation)
   at LawhallwheachalNakearjalle.MainPage.InitializeComponent()
   at LawhallwheachalNakearjalle.MainPage..ctor()
```

请看下面代码，虽然有定义资源，但是定义资源在使用的代码之后

```csharp
    <Grid Background="{StaticResource Foo}">
     
    </Grid>
    
    <Page.Resources>
        <SolidColorBrush x:Key="Foo">#565656</SolidColorBrush>
    </Page.Resources>
```

建议将资源写在最前

请看下面代码，虽然有定义资源，但是定义资源在控件往上找不到的控件

```csharp
    <Grid>
        <Grid>
            <Grid.Resources>
                <SolidColorBrush x:Key="Foo">#565656</SolidColorBrush>
            </Grid.Resources>
        </Grid>

        <Grid Background="{StaticResource Foo}">

        </Grid>
    </Grid>
```

此时 Grid 的容器没有资源

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
