
# WPF 框架开发 ColumnDefinition 和 RowDefinition 的代码在哪

我的 VisualStudio 在更新到 2022 就构建不通过 WPF 仓库，提示我在 Grid 的代码里面找不到 ColumnDefinitionCollection 和 RowDefinitionCollection 等的定义，在我开始找 WPF 仓库关于这几个类型的定义时，居然找不到对应的源代码。本文来告诉大家在 WPF 仓库里面是如何存放几个类型

<!--more-->


<!-- CreateTime:2021/8/9 8:34:31 -->

<!-- 发布 -->

在上一篇博客 [手把手教你如何构建 WPF 官方开源框架源代码](https://blog.lindexi.com/post/%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E5%A6%82%E4%BD%95%E6%9E%84%E5%BB%BA-WPF-%E5%AE%98%E6%96%B9%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E6%BA%90%E4%BB%A3%E7%A0%81.html ) 告诉大家如何进行本地构建，本文将此基础上继续进行解决在 VisualStudio 2022 预览版构建失败的坑，顺便告诉大家在 WPF 仓库里面那些有趣的代码存放方法

本文非新手友好，本文的 WPF 框架开发不是说开发一个基于 WPF 框架的应用，也不是指开发 WPF 应用。而是开发 WPF 这个框架，这是做底层开发的博客

以下是在 VisualStudio 2019 进行构建，十分简单，只需要部署环境完成之后进行构建即可

![](http://image.acmx.xyz/lindexi%2F202188206162793.jpg)

然而在 VisualStudio 2022 里面，将会在构建的时候提示失败

```
“f:\lindexi\Code\wpf\Microsoft.Dotnet.Wpf.sln”(默认目标) (1:2) ->
“f:\lindexi\Code\wpf\src\Microsoft.DotNet.Wpf\src\PresentationFramework\PresentationFramework.csproj”(默认目标) (11:28) ->
(CoreCompile 目标) ->
  f:\lindexi\Code\wpf\src\Microsoft.DotNet.Wpf\src\PresentationFramework\System\Windows\Controls\Grid.cs(309,16): error CS0246: 未能找到类型或命名空间名“ColumnDefinitionCollection”(是否缺少 using 指令或程序集引用?)
[f:\lindexi\Code\wpf\src\Microsoft.DotNet.Wpf\src\PresentationFramework\PresentationFramework.csproj]
  f:\lindexi\Code\wpf\src\Microsoft.DotNet.Wpf\src\PresentationFramework\System\Windows\Controls\Grid.cs(324,16): error CS0246: 未能找到类型或命名空间名“RowDefinitionCollection”(是否缺少 using 指令或程序集引用?) [f:\lindexi
\Code\wpf\src\Microsoft.DotNet.Wpf\src\PresentationFramework\PresentationFramework.csproj]
  f:\lindexi\Code\wpf\src\Microsoft.DotNet.Wpf\src\PresentationFramework\System\Windows\Controls\Grid.cs(3347,22): error CS0246: 未能找到类型或命名空间名“ColumnDefinitionCollection”(是否缺少 using 指令或程序集引用?)
 [f:\lindexi\Code\wpf\src\Microsoft.DotNet.Wpf\src\PresentationFramework\PresentationFramework.csproj]
  f:\lindexi\Code\wpf\src\Microsoft.DotNet.Wpf\src\PresentationFramework\System\Windows\Controls\Grid.cs(3348,22): error CS0246: 未能找到类型或命名空间名“RowDefinitionCollection”(是否缺少 using 指令或程序集引用?) [f
:\Code\wpf\src\Microsoft.DotNet.Wpf\src\PresentationFramework\PresentationFramework.csproj]
  f:\lindexi\Code\wpf\src\Microsoft.DotNet.Wpf\src\PresentationFramework\System\Windows\Controls\Grid.cs(4151,21): error CS0246: 未能找到类型或命名空间名“ColumnDefinitionCollection”(是否缺少 using 指令或程序集引用?)
 [f:\lindexi\Code\wpf\src\Microsoft.DotNet.Wpf\src\PresentationFramework\PresentationFramework.csproj]
  f:\lindexi\Code\wpf\src\Microsoft.DotNet.Wpf\src\PresentationFramework\System\Windows\Controls\Grid.cs(4152,21): error CS0246: 未能找到类型或命名空间名“RowDefinitionCollection”(是否缺少 using 指令或程序集引用?) [f
:\Code\wpf\src\Microsoft.DotNet.Wpf\src\PresentationFramework\PresentationFramework.csproj]
```

我进入了 WPF 仓库里面，想要看看 ColumnDefinitionCollection 和 RowDefinitionCollection 等的定义，但是在 VisualStudio 里面实际上是找不到这几个类的代码的

原因是在 WPF 中，上古的开发者觉得 RowDefinitionCollection 和 ColumnDefinitionCollection 的代码差不多，而 ColumnDefinition 和 RowDefinition 的代码也差不多，于是就想用黑科技，通过配置生成这些类型。可以在 WPF 仓库的 `src\Microsoft.DotNet.Wpf\src\PresentationFramework\MS\Utility` 文件夹看到很多有趣的逻辑，在此文件夹可以看到如下的几个文件

```
    ColumnDefinition.ti
    GridContentElementCollection.tb
    GridContentElementCollection.th
    RowDefinition.ti
```

打开 `GridContentElementCollection.tb` 文件，可以看到这里面的代码定义就特别有趣，以下是删减的部分

```csharp
namespace System.Windows.Controls
{
    /// <summary>
    /// A <<COLLECTIONTYPE>> is an ordered, strongly typed, non-sparse 
    /// collection of <<ITEMTYPE>>s. 
    /// </summary>
    /// <remarks>
    /// <<COLLECTIONTYPE>> provides public access for <<ITEMTYPE>>s 
    /// reading and manipulation. 
    /// </remarks>
    public sealed class <<COLLECTIONTYPE>> : IList<<<ITEMTYPE>>> , IList
    {
        //------------------------------------------------------
        //
        //  Constructors
        //
        //------------------------------------------------------

        #region Constructors

        /// <summary>
        ///     Default ctor.
        /// </summary>
        internal <<COLLECTIONTYPE>>(<<OWNERTYPE>> owner)
        {
            _owner = owner;
            PrivateOnModified();
        }

        #endregion Constructors

    }

    /// <summary>
    ///     <<ITEMTYPE>> is a FrameworkContentElement used by Grid 
    ///     to hold column / row specific properties.
    /// </summary>
    public class <<ITEMTYPE>> : DefinitionBase
    {
        //------------------------------------------------------
        //
        //  Constructors
        //
        //------------------------------------------------------

        #region Constructors

        /// <summary>
        ///     Default ctor.
        /// </summary>
        public <<ITEMTYPE>>()
            : base(DefinitionBase.ThisIs<<ITEMTYPE>>)
        {
        }

        #endregion Constructors

        //------------------------------------------------------
        //
        //  Public Properties
        //
        //------------------------------------------------------

        #region Public Properties 

        /// <summary>
        ///     Sets specified <<WIDTHHEIGHT>> value for the <<ITEMTYPE>>.
        ///     Returns current <<WIDTHHEIGHT>> value for the <<ITEMTYPE>>. 
        /// </summary>
        public GridLength <<WIDTHHEIGHT>>
        {
            get { return (base.UserSizeValueCache); }
            set { SetValue(<<WIDTHHEIGHT>>Property, value); }
        }

        /// <summary>
        ///     Sets specified Min<<WIDTHHEIGHT>> value for the <<ITEMTYPE>>.
        ///     Returns current Min<<WIDTHHEIGHT>> value for the <<ITEMTYPE>>.
        /// </summary>
        [TypeConverter(typeof(LengthConverter))]
        public double Min<<WIDTHHEIGHT>>
        {
            get { return (base.UserMinSizeValueCache); }
            set { SetValue(Min<<WIDTHHEIGHT>>Property, value); }
        }

        /// <summary>
        ///     Sets specified Max<<WIDTHHEIGHT>> value for the <<ITEMTYPE>>.
        ///     Returns current Max<<WIDTHHEIGHT>> value for the <<ITEMTYPE>>.
        /// </summary>
        [TypeConverter(typeof(LengthConverter))]
        public double Max<<WIDTHHEIGHT>>
        {
            get { return (base.UserMaxSizeValueCache); }
            set { SetValue(Max<<WIDTHHEIGHT>>Property, value); }
        }

        /// <summary>
        ///     Returns calculated device independent pixel value of <<WIDTHHEIGHT>> for the <<ITEMTYPE>>.
        /// </summary>
        public double Actual<<WIDTHHEIGHT>>
        {
            get
            {
                double value = 0.0;

                if (base.InParentLogicalTree)
                {
                    value = ((Grid)base.Parent).GetFinal<<ITEMTYPE>><<WIDTHHEIGHT>>(base.Index);
                }

                return (value);
            }
        }
    }
}
```

可以看到，如果将上面代码的 `<<COLLECTIONTYPE>>` 等内容替换掉，那不就是实际上的类型定义了？实际上就是如此，还请打开一下 `ColumnDefinition.ti` 和 `RowDefinition.ti` 文件看一下，以下是 `ColumnDefinition.ti` 文件的内容

```
::BEGIN_TEMPLATE
COLLECTIONTYPE:ColumnDefinitionCollection
ITEMTYPE:ColumnDefinition
OWNERTYPE:Grid
WIDTHHEIGHT:Width
::END_TEMPLATE

::END
```

从上面代码可以看到，从某个 `<<Foo>>` 替换的规则也就可以猜到了。如将 `<<COLLECTIONTYPE>>` 根据 `COLLECTIONTYPE:ColumnDefinitionCollection` 的规则，替换为 `ColumnDefinitionCollection` 即可。同理替换其他的逻辑

其实在 WPF 里面，即使在 VisualStudio 2022 也是有自动生成的，不需要咱做什么科技

还请看看如下两个文件

```
f:\lindexi\Code\wpf\artifacts\obj\PresentationFramework\Debug\net6.0\ColumnDefinition.cs

f:\lindexi\Code\wpf\artifacts\obj\PresentationFramework\Debug\net6.0\RowDefinition.cs
```

那为什么我在本文开始依然构建失败呢？那就是需要问问神奇的 VisualStudio 2022 啦，因为在 VisualStudio 2022 预览版在生成了如上两个文件之前，就先跑去构建 Grid.cs 文件啦

那另一个问题是，是哪个逻辑负责生成以上的文件的？请打开 `src\Microsoft.DotNet.Wpf\src\PresentationFramework\template.pl` 文件，这是由古老的 perl 提供的黑科技。相信 Perl 只有上古的开发者才知道这是什么啦。本文不想去聊 Perl 的内容，原因是我也不知道，也不想去学

更多 WPF 框架构建相关，请看

- [手把手教你如何构建 WPF 官方开源框架源代码](https://blog.lindexi.com/post/%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E5%A6%82%E4%BD%95%E6%9E%84%E5%BB%BA-WPF-%E5%AE%98%E6%96%B9%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E6%BA%90%E4%BB%A3%E7%A0%81.html ) 
- [手把手教你构建 WPF 框架的私有版本](https://blog.lindexi.com/post/%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E6%9E%84%E5%BB%BA-WPF-%E6%A1%86%E6%9E%B6%E7%9A%84%E7%A7%81%E6%9C%89%E7%89%88%E6%9C%AC.html)
- [WPF 框架全构建环境虚拟机硬盘分享](https://blog.lindexi.com/post/WPF-%E6%A1%86%E6%9E%B6%E5%85%A8%E6%9E%84%E5%BB%BA%E7%8E%AF%E5%A2%83%E8%99%9A%E6%8B%9F%E6%9C%BA%E7%A1%AC%E7%9B%98%E5%88%86%E4%BA%AB.html)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。