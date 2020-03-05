# WPF 如何调试 binding

如果是写在 xaml 的绑定，很难看到是那里出错。本文告诉大家如何在 vs 调试 binding 是否绑定上？

<!--more-->
<!-- CreateTime:2019/9/18 10:01:04 -->


<div id="toc"></div>
<!-- csdn -->

<!-- 标签：WPF，WPF调试，调试 -->

在写 Binding 的时候，需要在运行的时候才能知道是否绑定对了，但是可能有时候代码是好的，但是有逗比小伙伴改了一下就不知道是属性名修改了绑定不上，还是转换器没写对

在 WPF 提供了输出绑定的信息，可以在 binding 输出很多关于他如何做的，如何寻找绑定的数据，绑定是如何创建的

本文告诉大家如何使用绑定输出

首先引用命名空间，请在添加 trace 命名空间

```csharp
          xmlns:trace="clr-namespace:System.Diagnostics;assembly=WindowsBase"

```

然后在 binding 里写 `trace:PresentationTraceSources.TraceLevel=High` 设置调试等级

我在一个复杂的界面，很难知道是不是在哪写错，在绑定里面添加输出的方法如下

```
    <TextBlock FontSize="48" HorizontalAlignment="Center" VerticalAlignment="Center"
                               Text="{Binding RelativeSource={RelativeSource AncestorType=ListBoxItem}, Path=(ItemsControl.AlternationIndex),trace:PresentationTraceSources.TraceLevel=High, Mode=OneWay, Converter={StaticResource NumberAddOne}}">
                   
```

如果这个 binding 是在后台代码创建，那么请使用下面代码，这里 BakooteZuroolu 是一个 TextBlock 元素，把他的 Text 绑定到 KasxoujarGayher 一个我也不知道什么的属性，可以看到代码很少。

```csharp
            var binding = new Binding(nameof(ViewModel.KasxoujarGayher));

            PresentationTraceSources.SetTraceLevel(binding, PresentationTraceLevel.High);

            BindingOperations.SetBinding(BakooteZuroolu, TextBlock.TextProperty, binding);
```

运行打开就可以看到输出这些代码


```csharp
    System.Windows.Data Warning: 56 : Created BindingExpression (hash=36771698) for Binding (hash=7954964)
System.Windows.Data Warning: 58 :   Path: '(0)'
System.Windows.Data Warning: 61 : BindingExpression (hash=36771698): Default update trigger resolved to PropertyChanged
System.Windows.Data Warning: 56 : Created BindingExpression (hash=62509834) for Binding (hash=7954964)
System.Windows.Data Warning: 58 :   Path: '(0)'
System.Windows.Data Warning: 61 : BindingExpression (hash=62509834): Default update trigger resolved to PropertyChanged
System.Windows.Data Warning: 62 : BindingExpression (hash=62509834): Attach to System.Windows.Controls.TextBlock.Text (hash=8340425)
System.Windows.Data Warning: 66 : BindingExpression (hash=62509834): RelativeSource (FindAncestor) requires tree context
System.Windows.Data Warning: 65 : BindingExpression (hash=62509834): Resolve source deferred
System.Windows.Data Warning: 56 : Created BindingExpression (hash=55365254) for Binding (hash=37966223)
System.Windows.Data Warning: 58 :   Path: '(0)'
System.Windows.Data Warning: 61 : BindingExpression (hash=55365254): Default update trigger resolved to PropertyChanged
System.Windows.Data Warning: 56 : Created BindingExpression (hash=28525238) for Binding (hash=37966223)
System.Windows.Data Warning: 58 :   Path: '(0)'
System.Windows.Data Warning: 61 : BindingExpression (hash=28525238): Default update trigger resolved to PropertyChanged
System.Windows.Data Warning: 62 : BindingExpression (hash=28525238): Attach to System.Windows.Controls.TextBlock.Text (hash=48957711)
System.Windows.Data Warning: 66 : BindingExpression (hash=28525238): RelativeSource (FindAncestor) requires tree context
System.Windows.Data Warning: 65 : BindingExpression (hash=28525238): Resolve source deferred
System.Windows.Data Warning: 56 : Created BindingExpression (hash=16291455) for Binding (hash=40797849)
System.Windows.Data Warning: 58 :   Path: '(0)'
System.Windows.Data Warning: 61 : BindingExpression (hash=16291455): Default update trigger resolved to PropertyChanged
System.Windows.Data Warning: 56 : Created BindingExpression (hash=12405375) for Binding (hash=40797849)
System.Windows.Data Warning: 58 :   Path: '(0)'
System.Windows.Data Warning: 61 : BindingExpression (hash=12405375): Default update trigger resolved to PropertyChanged
System.Windows.Data Warning: 62 : BindingExpression (hash=12405375): Attach to System.Windows.Controls.TextBlock.Text (hash=34359256)
System.Windows.Data Warning: 66 : BindingExpression (hash=12405375): RelativeSource (FindAncestor) requires tree context
System.Windows.Data Warning: 65 : BindingExpression (hash=12405375): Resolve source deferred
System.Windows.Data Warning: 56 : Created BindingExpression (hash=6720246) for Binding (hash=42336695)
System.Windows.Data Warning: 58 :   Path: '(0)'
System.Windows.Data Warning: 61 : BindingExpression (hash=6720246): Default update trigger resolved to PropertyChanged
System.Windows.Data Warning: 56 : Created BindingExpression (hash=60482217) for Binding (hash=42336695)
System.Windows.Data Warning: 58 :   Path: '(0)'
System.Windows.Data Warning: 61 : BindingExpression (hash=60482217): Default update trigger resolved to PropertyChanged
System.Windows.Data Warning: 62 : BindingExpression (hash=60482217): Attach to System.Windows.Controls.TextBlock.Text (hash=64356400)
System.Windows.Data Warning: 66 : BindingExpression (hash=60482217): RelativeSource (FindAncestor) requires tree context
System.Windows.Data Warning: 65 : BindingExpression (hash=60482217): Resolve source deferred
System.Windows.Data Warning: 56 : Created BindingExpression (hash=29858833) for Binding (hash=64163473)
System.Windows.Data Warning: 58 :   Path: '(0)'
System.Windows.Data Warning: 61 : BindingExpression (hash=29858833): Default update trigger resolved to PropertyChanged
System.Windows.Data Warning: 56 : Created BindingExpression (hash=294043) for Binding (hash=64163473)
System.Windows.Data Warning: 58 :   Path: '(0)'
System.Windows.Data Warning: 61 : BindingExpression (hash=294043): Default update trigger resolved to PropertyChanged
System.Windows.Data Warning: 62 : BindingExpression (hash=294043): Attach to System.Windows.Controls.TextBlock.Text (hash=59325057)
System.Windows.Data Warning: 66 : BindingExpression (hash=294043): RelativeSource (FindAncestor) requires tree context
System.Windows.Data Warning: 65 : BindingExpression (hash=294043): Resolve source deferred
System.Windows.Data Warning: 67 : BindingExpression (hash=62509834): Resolving source 
System.Windows.Data Warning: 70 : BindingExpression (hash=62509834): Found data context element: <null> (OK)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried Grid (hash=64665535)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried Grid (hash=22098140)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried ContentPresenter (hash=9911889)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried Border (hash=8557861)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried ListViewItem (hash=48873995)
System.Windows.Data Warning: 72 :   RelativeSource.FindAncestor found ListViewItem (hash=48873995)
System.Windows.Data Warning: 78 : BindingExpression (hash=62509834): Activate with root item ListViewItem (hash=48873995)
System.Windows.Data Warning: 108 : BindingExpression (hash=62509834):   At level 0 - for ListViewItem.(ItemsControl.AlternationIndex) found accessor DependencyProperty(AlternationIndex)
System.Windows.Data Warning: 104 : BindingExpression (hash=62509834): Replace item at level 0 with ListViewItem (hash=48873995), using accessor DependencyProperty(AlternationIndex)
System.Windows.Data Warning: 101 : BindingExpression (hash=62509834): GetValue at level 0 from ListViewItem (hash=48873995) using DependencyProperty(AlternationIndex): '0'
System.Windows.Data Warning: 80 : BindingExpression (hash=62509834): TransferValue - got raw value '0'
System.Windows.Data Warning: 82 : BindingExpression (hash=62509834): TransferValue - user's converter produced '1'
System.Windows.Data Warning: 84 : BindingExpression (hash=62509834): TransferValue - implicit converter produced '1'
System.Windows.Data Warning: 89 : BindingExpression (hash=62509834): TransferValue - using final value '1'
System.Windows.Data Warning: 67 : BindingExpression (hash=28525238): Resolving source 
System.Windows.Data Warning: 70 : BindingExpression (hash=28525238): Found data context element: <null> (OK)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried Grid (hash=42455531)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried Grid (hash=56913064)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried ContentPresenter (hash=13780214)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried Border (hash=23900756)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried ListViewItem (hash=37212772)
System.Windows.Data Warning: 72 :   RelativeSource.FindAncestor found ListViewItem (hash=37212772)
System.Windows.Data Warning: 78 : BindingExpression (hash=28525238): Activate with root item ListViewItem (hash=37212772)
System.Windows.Data Warning: 107 : BindingExpression (hash=28525238):   At level 0 using cached accessor for ListViewItem.(ItemsControl.AlternationIndex): DependencyProperty(AlternationIndex)
System.Windows.Data Warning: 104 : BindingExpression (hash=28525238): Replace item at level 0 with ListViewItem (hash=37212772), using accessor DependencyProperty(AlternationIndex)
System.Windows.Data Warning: 101 : BindingExpression (hash=28525238): GetValue at level 0 from ListViewItem (hash=37212772) using DependencyProperty(AlternationIndex): '1'
System.Windows.Data Warning: 80 : BindingExpression (hash=28525238): TransferValue - got raw value '1'
System.Windows.Data Warning: 82 : BindingExpression (hash=28525238): TransferValue - user's converter produced '2'
System.Windows.Data Warning: 84 : BindingExpression (hash=28525238): TransferValue - implicit converter produced '2'
System.Windows.Data Warning: 89 : BindingExpression (hash=28525238): TransferValue - using final value '2'
System.Windows.Data Warning: 67 : BindingExpression (hash=12405375): Resolving source 
System.Windows.Data Warning: 70 : BindingExpression (hash=12405375): Found data context element: <null> (OK)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried Grid (hash=23387474)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried Grid (hash=2598608)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried ContentPresenter (hash=45027976)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried Border (hash=57198891)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried ListViewItem (hash=66479500)
System.Windows.Data Warning: 72 :   RelativeSource.FindAncestor found ListViewItem (hash=66479500)
System.Windows.Data Warning: 78 : BindingExpression (hash=12405375): Activate with root item ListViewItem (hash=66479500)
System.Windows.Data Warning: 107 : BindingExpression (hash=12405375):   At level 0 using cached accessor for ListViewItem.(ItemsControl.AlternationIndex): DependencyProperty(AlternationIndex)
System.Windows.Data Warning: 104 : BindingExpression (hash=12405375): Replace item at level 0 with ListViewItem (hash=66479500), using accessor DependencyProperty(AlternationIndex)
System.Windows.Data Warning: 101 : BindingExpression (hash=12405375): GetValue at level 0 from ListViewItem (hash=66479500) using DependencyProperty(AlternationIndex): '2'
System.Windows.Data Warning: 80 : BindingExpression (hash=12405375): TransferValue - got raw value '2'
System.Windows.Data Warning: 82 : BindingExpression (hash=12405375): TransferValue - user's converter produced '3'
System.Windows.Data Warning: 84 : BindingExpression (hash=12405375): TransferValue - implicit converter produced '3'
System.Windows.Data Warning: 89 : BindingExpression (hash=12405375): TransferValue - using final value '3'
System.Windows.Data Warning: 67 : BindingExpression (hash=60482217): Resolving source 
System.Windows.Data Warning: 70 : BindingExpression (hash=60482217): Found data context element: <null> (OK)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried Grid (hash=47357306)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried Grid (hash=35088084)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried ContentPresenter (hash=3898676)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried Border (hash=22802807)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried ListViewItem (hash=61444595)
System.Windows.Data Warning: 72 :   RelativeSource.FindAncestor found ListViewItem (hash=61444595)
System.Windows.Data Warning: 78 : BindingExpression (hash=60482217): Activate with root item ListViewItem (hash=61444595)
System.Windows.Data Warning: 107 : BindingExpression (hash=60482217):   At level 0 using cached accessor for ListViewItem.(ItemsControl.AlternationIndex): DependencyProperty(AlternationIndex)
System.Windows.Data Warning: 104 : BindingExpression (hash=60482217): Replace item at level 0 with ListViewItem (hash=61444595), using accessor DependencyProperty(AlternationIndex)
System.Windows.Data Warning: 101 : BindingExpression (hash=60482217): GetValue at level 0 from ListViewItem (hash=61444595) using DependencyProperty(AlternationIndex): '3'
System.Windows.Data Warning: 80 : BindingExpression (hash=60482217): TransferValue - got raw value '3'
System.Windows.Data Warning: 82 : BindingExpression (hash=60482217): TransferValue - user's converter produced '4'
System.Windows.Data Warning: 84 : BindingExpression (hash=60482217): TransferValue - implicit converter produced '4'
System.Windows.Data Warning: 89 : BindingExpression (hash=60482217): TransferValue - using final value '4'
System.Windows.Data Warning: 67 : BindingExpression (hash=294043): Resolving source 
System.Windows.Data Warning: 70 : BindingExpression (hash=294043): Found data context element: <null> (OK)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried Grid (hash=45699530)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried Grid (hash=5077725)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried ContentPresenter (hash=564191)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried Border (hash=62687)
System.Windows.Data Warning: 73 :     Lookup ancestor of type ListBoxItem:  queried ListViewItem (hash=16130451)
System.Windows.Data Warning: 72 :   RelativeSource.FindAncestor found ListViewItem (hash=16130451)
System.Windows.Data Warning: 78 : BindingExpression (hash=294043): Activate with root item ListViewItem (hash=16130451)
System.Windows.Data Warning: 107 : BindingExpression (hash=294043):   At level 0 using cached accessor for ListViewItem.(ItemsControl.AlternationIndex): DependencyProperty(AlternationIndex)
System.Windows.Data Warning: 104 : BindingExpression (hash=294043): Replace item at level 0 with ListViewItem (hash=16130451), using accessor DependencyProperty(AlternationIndex)
```

参见 [http://jimmangaly.blogspot.com/2009/04/debugging-data-binding-in-wpf.html](http://jimmangaly.blogspot.com/2009/04/debugging-data-binding-in-wpf.html)

如果是想要在 VisualStudio 输出所有绑定的详细信息请看 [WPF 笔刷绑定不上可能的原因](https://blog.lindexi.com/post/WPF%20%E7%AC%94%E5%88%B7%E7%BB%91%E5%AE%9A%E4%B8%8D%E4%B8%8A%E5%8F%AF%E8%83%BD%E7%9A%84%E5%8E%9F%E5%9B%A0.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 