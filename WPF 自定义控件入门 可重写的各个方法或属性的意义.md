# WPF 自定义控件入门 可重写的各个方法或属性的意义

本文属于 WPF 自定义控件入门系列博客。本文整理在 WPF 里面，自定义控件，非用户控件时，可以重写基类的许多方法和属性，这些方法和属性的作用和含义。方便让大家了解到自定义控件时，有哪些方法或属性可以被重写，重写时的正确实现以及其影响是什么

<!--more-->
<!-- 草稿 -->

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

回顾一下，正常开发 WPF 应用的时候，如果一个控件元素将会包含多个子控件，大部分情况下这个控件元素会被咱写为一个继承自 Panel 的类型，表示这是一个容器控件。但有些情况，例如这个控件元素仅仅只包含一个子级，一个子控件且是固定的类型，而且从业务逻辑上也不是一个容器的概念。这个时候咱依然可以继承 FrameworkElement 来进行自己编写。本文也着重告诉大家这个方法，而不是采用比较上层封装的 Panel 容器类型，从而让大家能够了解更多的细节

十分符合预期的 F2 类型如果只是作为 F1 的一个 CLR 属性，是不能让 F2 加入到 WPF 机制里面的，无法让 F2 的事件和重写的方法被符合预期的调用

接下来咱来修改一下 F1 类型，重写 VisualChildrenCount 属性和 GetVisualChild 方法