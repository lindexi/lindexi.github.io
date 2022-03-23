# XAML 给资源起个好名字 用 StaticResource 起一个别名

本文来和大家聊一下关于 XAML 资源的定义的事情，和开发技术关系不大，更多的是开发的思路

<!--more-->
<!-- CreateTime:2022/3/21 8:29:08 -->

<!-- 标签：wpf,xaml -->
<!-- 发布 -->

在稍微大一点的项目里，肯定 XAML 资源是少不了的。对于 XAML 资源，行业里讨论多（非小白讨论）的是关于资源的复用和初始化时机，加载时机的问题。本文聊的是资源的复用以及变更资源的开发模式

如果团队里有一位开发者是命名高手，那么问题将特别小。或者说这个项目只有一位开发者在开发，也不需要后续维护，那这个问题也特别的小。然而在很多团队里面，都有很多开发者，同时开发者的命名能力也有一定的差距

如果没有给资源一个好的命名，自然，每次用起来的时候，都会遇到一个问题，那就是某个资源找不到的问题。一旦找不到资源，自然就只能重新定义资源了。或者说我以为某个资源是我此模块能用的，然而此资源却非通用设计，也许下个版本就被更改，于是我的模块就因为某个资源的变更而挖坑

如下面的例子：

我需要开发一个应用，此应用有多个页面组合。由于技术侧的问题，我不能将每个页面都使用相同的控件，需要采用不同的控件。但是为了界面的美观，尽管使用的不同的控件，依然也需要保持相似页面布局方式。按照惯例，假定定义页面的内边距，假设默认的页边距是 12 单位大小，定义为资源可以如何写？如以下的资源定义

```xml
<Thickness x:Key="TwelveThickness">12,12,12,12</Thickness>
```

可以看到，定义资源为 12 单位的 Thickness 有点过于具体了。如果后续设计师想改为 16 个单位呢？那此时就是一个选择了，要么将此资源改为 16 的值，但是保留 TwelveThickness 这个词，让其他的开发者看到这里虽然写着是 Twelve 但实际上是 16 个单位。要么就是改值和改资源名，说不定就是炸鱼的改动了。要么就是再新加一个资源名，也许这样垃圾就加一了

如果有一个具体的命名呢？命名和数值无关的呢？例如命名为 MainPagePadding 呢？这个感觉是不错的，给主页面使用的 Padding 值。但是不够抽象，如果我用的是 FxxPage 的呢，这个也需要一个内边距，那用哪个资源好呢

如果定义的资源命名是 `DefaultPagePadding` 呢，或者 `PagePadding` 呢，是不是就更抽象了一层，不仅 MainPage 能用，也合适 FxxPage 使用

但是定义的抽象等级，除非是命名高手，否则一次性叫对一个命名还是很有难度的。而有一些资源，又是十分冲突的。既是非常通用的，却也许会被变更。例如颜色，定义颜色画刷的时候，资源的重复存在的问题核心就是开发模式上的寻找困难和被其他开发者更改带来的锅，在性能上的问题就是非托管资源的占用增加，没有复用原有的画刷。但是颜色的定义，是会在迭代被变更的

从技术侧的一个解决方法是采用 StaticResource 来进行资源的引用，相当于给资源一个别名的方式。再定义一个资源，引用原先的资源

例如有一个红色是默认的主题红色，最好的定义是 `Brush.SolidColorBrush.RedThemeBrush` 的资源名。然而此资源名特别具体，如果作为主页面的背景色的时候，此时将会因为太过具体而不合适。毕竟后续如果我还有其他的页面，似乎用来做背景的，耦合了具体的红主题色是不合适的

那如果再定义一个画刷主题，称为 `MainPageBackgroundBrush` 呢？自然，重复定义的画刷就是重复的资源，不合适

好在可以使用 StaticResource 的方式，使用静态资源引用，从而让资源被重新定向，如下面代码，定义了一个主题颜色

```xml
        <SolidColorBrush x:Key="Brush.SolidColorBrush.RedThemeBrush" Color="#FFC10606"/>
```

接下来可以采用 StaticResource 引用此颜色，定义一个默认的页面的背景画刷

```xml
        <StaticResource x:Key="DefaultPageBackgroundBrush" ResourceKey="Brush.SolidColorBrush.RedThemeBrush"/>
```

再定义具体的主页面的背景画刷

```xml
        <StaticResource x:Key="MainPageBackgroundBrush" ResourceKey="DefaultPageBackgroundBrush"/>
```

于是在引用的时候，可以在主页面，只引用主页面的背景画刷

```xml
        <Grid x:Name="MainPage" Background="{StaticResource MainPageBackgroundBrush}"></Grid>
```

这样做的一个优势在于，让资源的定义是一层层定义的。解决了开发侧的重复资源定义，又想资源重复定义方便改动的时候相互不影响，又想着不重复定义方便要改可以一起改的问题

如以上的代码，相当于将资源的定义分为三层。最底层是 `Brush.SolidColorBrush.RedThemeBrush` 定义一个和业务无关的画刷，第二层是 `DefaultPageBackgroundBrush` 定义和页面有关的默认画刷，最上层是 `MainPageBackgroundBrush` 定义和主页面相关的画刷

这些画刷都是相同的一个资源，只是靠 StaticResource 进行引用

如果后续准备改主页面的画刷，而其他的页面保持不动。那么可以明确了解到，只需要改 MainPageBackgroundBrush 的资源值即可，影响不到其他的页面。而如果期望是全部页面的背景色都换成某个其他的颜色，只需要改 DefaultPageBackgroundBrush 即可。如果是设计师想要改整个应用的红色主题色，那就改 `Brush.SolidColorBrush.RedThemeBrush` 的颜色

如此的设计可以比较方便解决比较大的项目的资源引用问题。核心的思想就是依靠引用的方式，将资源的定义分层，按照抽象的不同，分为不同层的定义。定义资源的命名方式最好都是一组一组的，一个抽象层里面有很多组，每一组之间的命名都是非常相似的。如此就很方便资源的管理。这只是一个思想，不使用 StaticResource 也可以，如换成绑定的方式也可以

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、 使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
