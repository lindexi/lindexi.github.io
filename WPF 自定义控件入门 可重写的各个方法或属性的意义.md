# WPF 自定义控件入门 可重写的各个方法或属性的意义

本文属于 WPF 自定义控件入门系列博客。本文整理在 WPF 里面，自定义控件，非用户控件时，可以重写基类的许多方法和属性，这些方法和属性的作用和含义。方便让大家了解到自定义控件时，有哪些方法或属性可以被重写，重写时的正确实现以及其影响是什么

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

这是有伙伴问我，他在自定义控件时，发现了自己的自定义控件里面的子控件的 Loaded 事件不触发，命中测试不进入，以及测量布局方法没有被调用等问题。我开始无法快速帮助他定位到问题所在，于是在解决完问题之后，我就准备记录下来这篇博客，期望能够让大家有更好的思路去解决自定义控件时，所遇到的问题

在开始之前，期望大家对以下知识点有一个大概的了解，至少是需要听过：逻辑树，可视化树（又被我称为视觉树），控件，布局，元素，依赖属性，附加属性

本文将使用直接继承 FrameworkElement 的自定义控件类型为例子，由于在 WPF 里面有着新手比较友好的设计，在自己定义的一层（视觉树概念上的层级）控件上，各个事件或方法基本都能被符合预期正常触发。更底层的原因是在 WPF 里面，一个控件元素的布局或框架相关的事件和方法时由控件的父级控件所决定的，一个自定义的控件如果加入的是原生 WPF 自带的容器控件上，自然由于原生 WPF 自带的容器控件是正确实现了各个机制，于是自定义的控件的事件或方法都能正常被执行

换句话说就是，一个自定义的控件，加入到 WPF 自带的容器控件，如 Grid 等这些上面时。由于 WPF 自带的容器控件，如 Grid 等，是正确实现了机制，于是自定义的控件就抱了 WPF 自带的容器控件大腿，啥都不用干，各个事件和方法都是符合预期触发的

比如说自己定义一个名为 F1 的继承 FrameworkElement 的控件，代码如下

```csharp
class F1 : FrameworkElement
{
    public F1()
    {
        Width = 500;
        Height = 500;

        Loaded += F1_Loaded;
    }

    protected override Size MeasureOverride(Size availableSize)
    {
        Debug.WriteLine("F1 MeasureOverride");
        return base.MeasureOverride(availableSize);
    }

    private void F1_Loaded(object sender, RoutedEventArgs e)
    {
        Debug.WriteLine(nameof(F1_Loaded));
    }
}
```

将以上的 F1 加入到 Grid 里面，代码如下

```xml
    <Grid>
        <local:F1></local:F1>
    </Grid>
```

运行时，将会发现 F1 的 `MeasureOverride` 和 `F1_Loaded` 都会被触发。证明了 Loaded 事件符合预期被触发，且重写的 MeasureOverride 方法也符合预期被调用

```
F1 MeasureOverride
F1_Loaded
```

这就给了许多新手开发者一个误导，误以为自己定义的控件写对了。这里值得一提的是，如果只是单纯一层控件只是用来展示的话，那真的就是啥机制都不需要实现，就可以了。但是如果自定义的控件需要有复杂的交互或布局，比如包含子控件等，那就有一些机制需要正确实现

为了更好的说明，这里我需要用到放入到 F1 这个自定义控件里面的 F2 子控件来进一步和大家说明。这里的 F2 子控件，也是一个继承 FrameworkElement 的类型，代码定义如下

```csharp
class F2 : FrameworkElement
{
    public F2()
    {
        Width = 500;
        Height = 500;

        Loaded += F2_Loaded;
    }

    protected override Size MeasureOverride(Size availableSize)
    {
        Debug.WriteLine("F2 MeasureOverride");
        return base.MeasureOverride(availableSize);
    }

    private void F2_Loaded(object sender, RoutedEventArgs e)
    {
        Debug.WriteLine(nameof(F2_Loaded));
    }
}
```

可以看到几乎和 F1 一摸一样。这个 F2 子控件是从界面层级关系上，作为 F1 的子控件，也就是 F2 被包含在 F1 里面。以下是 F1 里面包含 F2 的代码

```csharp
class F1 : FrameworkElement
{
    public F1()
    {
        Width = 500;
        Height = 500;

        F2 = new F2();

        Loaded += F1_Loaded;
    }

    private F2 F2 { get; }

    protected override Size MeasureOverride(Size availableSize)
    {
        Debug.WriteLine("F1 MeasureOverride");
        return base.MeasureOverride(availableSize);
    }

    private void F1_Loaded(object sender, RoutedEventArgs e)
    {
        Debug.WriteLine(nameof(F1_Loaded));
    }
}
```

可以看到，此时 F1 仅仅只是将 F2 作为一个属性而已，没有附加任何机制。相信此时大家也能猜到 F2 的 Loaded 事件和 MeasureOverride 方法，肯定是不能符合预期的被调用

以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/e24633ab99ebc5a1def7204d4d4595bc582c7d1d/KearkemnerwhayneqiChaywibelfo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/e24633ab99ebc5a1def7204d4d4595bc582c7d1d/KearkemnerwhayneqiChaywibelfo) 欢迎访问

可以通过如下方式获取本文以上的源代码，先创建一个名为 KearkemnerwhayneqiChaywibelfo 的空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin e24633ab99ebc5a1def7204d4d4595bc582c7d1d
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin e24633ab99ebc5a1def7204d4d4595bc582c7d1d
```

获取代码之后，进入 KearkemnerwhayneqiChaywibelfo 文件夹


回顾一下，正常开发 WPF 应用的时候，如果一个控件元素将会包含多个子控件，大部分情况下这个控件元素会被咱写为一个继承自 Panel 的类型，表示这是一个容器控件。但有些情况，例如这个控件元素仅仅只包含一个子级，一个子控件且是固定的类型，而且从业务逻辑上也不是一个容器的概念。这个时候咱依然可以继承 FrameworkElement 来进行自己编写。本文也着重告诉大家这个方法，而不是采用比较上层封装的 Panel 容器类型，从而让大家能够了解更多的细节

十分符合预期的 F2 类型如果只是作为 F1 的一个 CLR 属性，是不能让 F2 加入到 WPF 机制里面的，无法让 F2 的事件和重写的方法被符合预期的调用

接下来咱来修改一下 F1 类型，重写 VisualChildrenCount 属性和 GetVisualChild 方法

修改 F1 的代码如下

```csharp
class F1 : FrameworkElement
{
    ... // 忽略其他代码

    private F2 F2 { get; }

    protected override int VisualChildrenCount => 1;
    protected override Visual GetVisualChild(int index) => F2;
}
```

修改之后的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/ad3fe86708a297ea8058b44bb576d51a858349b7/KearkemnerwhayneqiChaywibelfo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/ad3fe86708a297ea8058b44bb576d51a858349b7/KearkemnerwhayneqiChaywibelfo) 欢迎访问

获取这一步骤的代码，可以在上文获取源代码的基础上，在 KearkemnerwhayneqiChaywibelfo 文件夹里面继续输入以下代码进行获取

```
git pull origin ad3fe86708a297ea8058b44bb576d51a858349b7
```

运行代码，可以看到输出如下

```
F1 MeasureOverride
F1_Loaded
F2_Loaded
```

也就是说 F2 的 Loaded 事件被触发了。但也仅仅只是 Loaded 事件被触发，而 `MeasureOverride` 方法没有被调用

通过以上的例子即可说明，想要让子自定义控件的 Loaded 事件能够正常被触发，是需要在 GetVisualChild 里返回子自定义控件

接下来继续测试其他的重写方法，比如命中测试和 OnRender 方法。先在以上代码的基础上，添加 HitTestCore 和 OnRender 方法，同时为了展现效果，也在 OnRender 里面绘制一个圆形，代码如下

```csharp
class F2 : FrameworkElement
{
    ... // 忽略其他代码

    protected override void OnRender(DrawingContext drawingContext)
    {
        drawingContext.DrawEllipse(Brushes.Red, null, new Point(30, 30), 20, 20);
    }

    protected override HitTestResult HitTestCore(PointHitTestParameters hitTestParameters)
    {
        Debug.WriteLine($"F2 HitTestCore");
        return base.HitTestCore(hitTestParameters);
    }
}
```

加上以上代码之后，继续运行程序。可以看到无论鼠标怎么晃，都不会进入 F2 的 HitTestCore 命中测试方法。同时 F2 绘制的圆形也无法在界面上看到。也就是说仅仅只有 重写 VisualChildrenCount 属性和 GetVisualChild 方法对此需求来说还是不够。在有需要将子自定义控件的 OnRender 方法的内容打到界面上以及让子自定义控件参与命中测试时，还需要加上更多的代码

先分析一下为什么 F2 的 OnRender 方法没有在界面打出来绘制的圆形。在 OnRender 方法上打断点，运行代码，可以看到断点没有进来

根据 [dotnet 读 WPF 源代码笔记 布局时 Arrange 如何影响元素渲染坐标](https://blog.lindexi.com/post/dotnet-%E8%AF%BB-WPF-%E6%BA%90%E4%BB%A3%E7%A0%81%E7%AC%94%E8%AE%B0-%E5%B8%83%E5%B1%80%E6%97%B6-Arrange-%E5%A6%82%E4%BD%95%E5%BD%B1%E5%93%8D%E5%85%83%E7%B4%A0%E6%B8%B2%E6%9F%93%E5%9D%90%E6%A0%87.html ) 博客，可以了解到，在 WPF 里面是会在 Arrange 方法里面调用 OnRender 方法的。换句话说就是，想要 OnRender 方法被调用，那就需要调用 Arrange 方法


<!-- 核心原因就是 F2 控件没有尺寸，尽管 F2 控件是已经输出了绘制指导指令，但是可惜由于 F2 没有布局尺寸，导致了完全被裁剪，整个 F2 控件被裁剪为不可见 -->

<!-- 了解了这个问题之后，解决方法也就自然知道了，既然没有布局尺寸，既然 WPF 的布局尺寸是通过 `UIElement.Arrange` 方法给的，那就不妨调用一下。修改之后的代码如下 -->

了解了这个问题之后，解决方法也就自然知道了，既然没有调用 Arrange 方法，那就不妨调用一下。修改之后的代码如下


```csharp
    public F1()
    {
        Width = 500;
        Height = 500;

        F2 = new F2();

        F2.Arrange(new Rect(new Point(), new Size(100, 100)));
    }
```

修改完成之后，运行代码，即可看到 F2 的内容可以打到界面上了

以上代码是在 F1 里面调用 `F2.Arrange` 方法，那直接在 F2 里面自己调用自己呢？其实也是可以的，尽管这样不太符合设计。因为 WPF 框架设计上 Arrange 就是专门给上一级控件在布局时调用的。尽管不符合设计，但是也是能解决问题

```csharp
    public F2()
    {
        Width = 500;
        Height = 500;

        Loaded += F2_Loaded;

        Arrange(new Rect(new Point(), new Size(100, 100)));
    }
```

这里还存在另一个问题，那就是布局裁剪问题。默认 WPF 在 FrameworkElement 将会自动裁剪超过布局传入尺寸的画面。比如 Arrange 方法的 Size 参数是 100x100 时，实际渲染的 RenderSize 却是 200x200 尺寸，默认行为下，只有 100x100 的界面内容可见

可以通过重写 GetLayoutClip 方法重新设置布局裁剪，如此即可方便让渲染内容超过实际画布大小。对于继承 FrameworkElement 元素的控件来说，默认 WPF 将会自动裁剪超过布局传入尺寸的画面，除非重写 GetLayoutClip 修改行为。对于继承 UIElement 元素的控件来说，取决于 ClipToBounds 属性，默认此 ClipToBounds 属性是 false 值，意味着不会自动裁剪，如果设置 true 的值，将会返回裁剪大小为 RenderSize 尺寸。以下是 UIElement 的源代码

```csharp
public class UIElement
{
    ... // 忽略其他代码
        protected virtual Geometry GetLayoutClip(Size layoutSlotSize)
        {
            if(ClipToBounds)
            {
                RectangleGeometry rect = new RectangleGeometry(new Rect(RenderSize));
                rect.Freeze();
                return rect;
            }
            else
                return null;
        }
}
```

无论如何，重写 GetLayoutClip 都可以实现绘制界面超过布局尺寸，重写 GetLayoutClip 方法可以返回一个几何裁剪，如果无需任何裁剪，则返回 null 值，如以下代码

```csharp
class F2 : FrameworkElement
{
    public F2()
    {
        Width = 500;
        Height = 500;

        Loaded += F2_Loaded;

        Arrange(new Rect(new Point(), new Size(1, 1)));
    }

    protected override Geometry GetLayoutClip(Size layoutSlotSize)
    {
        return null;
    }

    ... // 忽略其他代码
}
```

尽管 Arrange 传入是 1x1 尺寸，但是通过重写 GetLayoutClip 返回 null 从而让 F2 绘制的内容可以绘制到界面

命中测试也是依存于布局的功能，命中测试需要在元素具备布局尺寸才会被调用。同时可参与命中测试的元素也要求是在视觉树上的元素，为了让一个元素能够参与命中测试，也就是让控件的 HitTestCore 方法被触发，就需要让控件加入到视觉树上。可以通过调用 AddVisualChild 方法让控件加入到视觉树上，代码如下

```csharp
class F1 : FrameworkElement
{
    public F1()
    {
        Width = 500;
        Height = 500;

        F2 = new F2();

        AddVisualChild(F2);

        F2.Arrange(new Rect(new Point(), new Size(100, 100)));
    }

    ... // 忽略其他代码
}
```

修改之后的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/383ccb0c09f41ab676feae36fe5085898255b347/KearkemnerwhayneqiChaywibelfo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/383ccb0c09f41ab676feae36fe5085898255b347/KearkemnerwhayneqiChaywibelfo) 欢迎访问

获取这一步骤的代码，可以在上文获取源代码的基础上，在 KearkemnerwhayneqiChaywibelfo 文件夹里面继续输入以下代码进行获取

```
git pull origin 383ccb0c09f41ab676feae36fe5085898255b347
```

运行代码，然后晃动鼠标，在 F2 的 HitTestCore 方法上打断点，可以看到进入断点，证明 F2 的 HitTestCore 被调用

如果发现自己自定义的控件里面，子自定义控件的 HitTestCore 命中测试没有被触发，除了看 IsHitTestVisible 属性之外，还需要关注一下控件元素是否已经被布局了，且布局尺寸符合预期，同时控件元素也加入到视觉树上

以上就是通过简单的代码告诉大家 WPF 自定义控件的多个可重写方法的用法和意义

更多博客，请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
