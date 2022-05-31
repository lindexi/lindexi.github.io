# WPF 已知问题 资源字典树引用与资源寻找的坑

大家都知道，在 WPF 里面，可以让资源字典合并其他资源字典，从而定义出资源字典引用树。然而在资源字典引用树里面，如果没有理清关系，将可以作出一个超级复杂的引用关系网。如果在性能优化中，将网断开部分，可能就会出现找不到资源的情况。本文将告诉大家 WPF 的资源字典树在引用和寻找关系上的坑

<!--more-->
<!-- CreateTime:2022/5/30 8:42:06 -->


<!-- 发布 -->
<!-- 博客 -->

在开始之前先来演示一下正确的使用方法，也是绝大部分的项目和开发者最常用的方法。 也就是说，如果正常的做，是不会踩到坑的，只有在进行不良设计时才会踩坑

在 App.xaml 里面是作为资源字典的引用的 Root 最顶层，基础玩法都是在 App.xaml 引用其他资源字典，引用顺序基本上基础库，控件库，共用资源，共用样式，业务资源。如果顺序反了，很快就可以在运行应用时找不到资源炸了

例如有 DictionaryA.xaml 和 DictionaryB.xaml 和 DictionaryC.xaml 三个资源字典，在 DictionaryB 里面是共用样式，在 DictionaryC 里面是共用资源。在 DictionaryB 里面的样式引用了 DictionaryC 的资源。此时如果让 DictionaryB 通过 MergedDictionaries 的方式引用 DictionaryC 字典，将存在一个性能问题，那就是在创建资源的时候，如果在 App.xaml 里面也引用了 DictionaryC 那么将让 DictionaryC 被创建两次。一次是在 App.xaml 里面的，一次是在被 DictionaryB 的 MergedDictionaries 创建的，换句话说将会让 DictionaryC 里面的对象重复两次定义，占用资源也添加了启动时间

常用的优化方式就是只在 App.xaml 引用 DictionaryC 即可，不在 DictionaryB 里面加上引用。如果真的需要有设计时帮助，如让 VisualStudio 开启智能（zhàng）提示，那可以使用 `d:` 设计时资源形式。如此即可让 DictionaryC 只在 App.xaml 里面初始化一份，减少 DictionaryC 的重复创建和减少内存占用，提升了性能

例如在 DictionaryC 里面作为共用资源，定义了画刷资源，如下

```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <SolidColorBrush x:Key="SolidColorBrush1InC" Color="#565656"/>
</ResourceDictionary>
```

在 DictionaryB 里面定义了样式，样式需要用到 `SolidColorBrush1InC` 资源，代码如下

```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
                    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
                    mc:Ignorable="d">
    <d:ResourceDictionary.MergedDictionaries>
        <ResourceDictionary Source="DictionaryC.xaml"/>
    </d:ResourceDictionary.MergedDictionaries>

    <Style x:Key="ButtonStyleInB" TargetType="Button">
        <Setter Property="Background" Value="{StaticResource SolidColorBrush1InC}" />
    </Style>
</ResourceDictionary>
```

在 DictionaryB 里面不会再次合入 DictionaryC 字典，而是统一在 App.xaml 里面将两个资源字典合入。以上代码里面，包含了为了让 VisualStudio 能在设计时帮你找到资源加上的 `d:` 合并逻辑，这个逻辑不会在运行时有任何作用

在 App.xaml 里面的合入代码如下

```xml
<Application x:Class="GeacejalcurnawLarjearemwhear.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="clr-namespace:GeacejalcurnawLarjearemwhear"
             StartupUri="MainWindow.xaml">
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="DictionaryC.xaml"/>
                <ResourceDictionary Source="DictionaryB.xaml"/>
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

合入资源字典是有顺序要求的，如果是先合入 DictionaryB 再合入 DictionaryC 将会在 DictionaryB 里面需要引用资源时找不到资源

```
System.Windows.Markup.XamlParseException:““{DependencyProperty.UnsetValue}”不是属性“Background”的有效值。”
```

以上的测试代码放在[github](https://github.com/lindexi/lindexi_gd/tree/6f0ed747acf089095a8503bc8ff967c97efe9de5/GeacejalcurnawLarjearemwhear) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/6f0ed747acf089095a8503bc8ff967c97efe9de5/GeacejalcurnawLarjearemwhear) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 6f0ed747acf089095a8503bc8ff967c97efe9de5
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 GeacejalcurnawLarjearemwhear 文件夹

当然了，对于大部分的开发模型来说，都不会在次级的资源字典里面存放具体的资源或样式等的定义。例如没有在 App.xaml 引用 DictionaryB 资源字典，而是将 DictionaryB 放入到 DictionaryA 里面引用，关系如下

![](http://image.acmx.xyz/lindexi%2F2022529151513068.jpg)

这个引用关系是没有问题的，依然可以在资源字典 DictionaryB 里面找到 DictionaryC 的资源。更新之后的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/ab087c32a9bae3d7f5bf11914b52a007da440941/GeacejalcurnawLarjearemwhear) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/ab087c32a9bae3d7f5bf11914b52a007da440941/GeacejalcurnawLarjearemwhear) 欢迎访问

那如果继续让 DictionaryC 的实际定义放在更底层呢？例如引入 DictionaryD.xaml 定义的资源呢，引用的关系如下

![](http://image.acmx.xyz/lindexi%2F202252915428337.jpg)

在 DictionaryC.xaml 的代码变更如下

```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <ResourceDictionary.MergedDictionaries>
        <ResourceDictionary Source="DictionaryD.xaml"/>
    </ResourceDictionary.MergedDictionaries>
    <SolidColorBrush x:Key="SolidColorBrush1InC" Color="#565656"/>
</ResourceDictionary>
```

在 DictionaryD.xaml 定义了资源

```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <SolidColorBrush x:Key="SolidColorBrush1InD" Color="#565656"/>
</ResourceDictionary>
```

修改 DictionaryB.xaml 的代码，让 DictionaryB.xaml 的 ButtonStyleInB 的背景采用 `SolidColorBrush1InD` 资源

```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <Style x:Key="ButtonStyleInB" TargetType="Button">
        <Setter Property="Background" Value="{StaticResource SolidColorBrush1InD}" />
    </Style>
</ResourceDictionary>
```

运行程序，可以看到，在进行多级引用时，是可以成功在 DictionaryB.xaml 找到 DictionaryD.xaml 资源。也就是说在不同的两个资源字典树，一个在 DictionaryA 一个在 DictionaryC 字典树上的资源，是可以相互寻找到的

更新之后的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/217136cca1f087d7dfa69c5acbbd83d92a195298/GeacejalcurnawLarjearemwhear) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/217136cca1f087d7dfa69c5acbbd83d92a195298/GeacejalcurnawLarjearemwhear) 欢迎访问

同理，再次提升层级进行测试，结果依然是能找到资源的。再定义 DictionaryE.xaml 和 DictionaryF.xaml 资源字典，让 DictionaryE.xaml 去引用 DictionaryF.xaml 的资源，其引用关系如下

![](http://image.acmx.xyz/lindexi%2F20225291583679.jpg)

更新之后的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/7cfa0e9117e953fabf7d77b2efc35cb05334ff08/GeacejalcurnawLarjearemwhear) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/7cfa0e9117e953fabf7d77b2efc35cb05334ff08/GeacejalcurnawLarjearemwhear) 欢迎访问

通过以上的测试可以了解到，在去掉 App.xaml 这个 Root 顶层资源之后的多个不同的资源字典树，多个资源字典树的资源是可以被跨资源字典树进行引用的，和存放的层级无关

这也是非常符合预期的，通过这个功能，即可将需要复用的资源分开，减少重复的定义，提升界面资源的模块化

但是又有一项带坑的设计，那就是在除了 App.xaml 这个 Root 顶层资源之后的资源字典树，在资源字典树内是不能跨节点引用。例如以下的关系，将会找不到资源

![](http://image.acmx.xyz/lindexi%2F20225291512389807.jpg)

如上图，在 DictionaryA.xaml 资源字典里面引用了 DictionaryC.xaml 和 DictionaryB.xaml 两个资源字典，代码如下

```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <ResourceDictionary.MergedDictionaries>
        <ResourceDictionary Source="DictionaryC.xaml"/>
        <ResourceDictionary Source="DictionaryB.xaml"/>
    </ResourceDictionary.MergedDictionaries>
</ResourceDictionary>
```

依然在 DictionaryC.xaml 里面定义资源

```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <SolidColorBrush x:Key="SolidColorBrush1InC" Color="#565656"/>
</ResourceDictionary>
```

在 DictionaryB.xaml 进行引用 SolidColorBrush1InC 资源，代码和上文的一样

```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <Style x:Key="ButtonStyleInB" TargetType="Button">
        <Setter Property="Background" Value="{StaticResource SolidColorBrush1InC}" />
    </Style>
</ResourceDictionary>
```

然而运行将会提示找不到 SolidColorBrush1InC 资源

大家可以尝试一下这个更新之后的代码，更新之后的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/66820e750fb1b5a104b3b4582dd31ac7393439bb/GeacejalcurnawLarjearemwhear) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/66820e750fb1b5a104b3b4582dd31ac7393439bb/GeacejalcurnawLarjearemwhear) 欢迎访问

可以通过如下方式获取更新后代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 66820e750fb1b5a104b3b4582dd31ac7393439bb
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 GeacejalcurnawLarjearemwhear 文件夹

也就是说在一个顶层的资源字典，非 App.xaml 哦，这个可不是资源字典，这个字典里面如果同时包含了共用资源和具体的样式，那如果在具体的样式里面用到任何共用资源，将会找不到共用的资源。这个就是本文要来告诉大家的 WPF 的已知问题

对于一些基础库来说，由于特殊的逻辑，不想分开两个资源字典，尽管分开两个资源字典更方便顶层业务层的定制需求，但是由于有特殊的需求而不想分开的，可以将 StaticResourceExtension 换成 DynamicResourceExtension 引用。利用 DynamicResourceExtension 会自动更新的机制，在 App.xaml 初始化资源字典的时候，实际访问将会重新去 App.xaml 寻找，从而找到资源

更改 DictionaryB.xaml 的代码如下

```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <Style x:Key="ButtonStyleInB" TargetType="Button">
        <Setter Property="Background" Value="{DynamicResource SolidColorBrush1InC}" />
    </Style>
</ResourceDictionary>
```

虽然换用 DynamicResourceExtension 在性能上是比不过 StaticResourceExtension 的，但好在如果数量不超过几万项的话，这部分降低的性能很少

这个问题我也报告给了 WPF 官方，请看 [The StaticResourceExtension will not find the resources of the ResourceDictionary of the sibling node · Issue #6627 · dotnet/wpf](https://github.com/dotnet/wpf/issues/6627)

<!-- 

Title WPF known issues: The StaticResourceExtension will not find the resources of the ResourceDictionary of the sibling node

 **Problem description:**

I think it is the design issues, the WPF known issues.

The StaticResourceExtension will not find the resources of the ResourceDictionary of the sibling node. For example, I create three ResourceDictionaries, DictionaryA.xaml, DictionaryB.xaml and DictionaryC.xaml. And the DictionaryA merged the DictionaryB and DictionaryC.

```xml
DictionaryA.xaml:

<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <ResourceDictionary.MergedDictionaries>
        <ResourceDictionary Source="DictionaryC.xaml"/>
        <ResourceDictionary Source="DictionaryB.xaml"/>
    </ResourceDictionary.MergedDictionaries>
</ResourceDictionary>
```

And I define the `SolidColorBrush1InC` resource in DictionaryC.xaml.


```xml
DictionaryC.xaml:

<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <SolidColorBrush x:Key="SolidColorBrush1InC" Color="#565656"/>
</ResourceDictionary>
```

And I try to use the `SolidColorBrush1InC` by StaticResourceExtension in DictionaryB.xaml.

```xml
DictionaryB.xaml:

<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <Style x:Key="ButtonStyleInB" TargetType="Button">
        <Setter Property="Background" Value="{StaticResource SolidColorBrush1InC}" />
    </Style>
</ResourceDictionary>
```

To use `ButtonStyleInB` resource, I add the code to MainWindow.

```xml
<Window x:Class="GeacejalcurnawLarjearemwhear.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:GeacejalcurnawLarjearemwhear"
        mc:Ignorable="d"
        Title="MainWindow" Height="450" Width="800">
    <Grid>
        <Button Style="{StaticResource ButtonStyleInB}" HorizontalAlignment="Center" VerticalAlignment="Center" Width="100" Height="100"> </Button>
    </Grid>
</Window>
```

![](http://image.acmx.xyz/lindexi%2F20225291512389807.jpg)

Run the code and you can find the XamlParseException, unless you replace the StaticResourceExtension to DynamicResourceExtension.

 **Actual behavior:**

It throw the XamlParseException.

```
System.Windows.Markup.XamlParseException:""{DependencyProperty.UnsetValue}" is not a valid value for property "Background""
```

 **Expected behavior:**

The `SolidColorBrush1InC` is defined in the Application, and the StaticResourceExtension should find it.

 **Minimal repro:**

I created the demo code in https://github.com/lindexi/lindexi_gd/tree/66820e750fb1b5a104b3b4582dd31ac7393439bb/GeacejalcurnawLarjearemwhear

-->

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
