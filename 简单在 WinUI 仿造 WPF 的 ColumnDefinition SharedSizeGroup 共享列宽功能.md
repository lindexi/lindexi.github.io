# 简单在 WinUI 仿造 WPF 的 ColumnDefinition SharedSizeGroup 共享列宽功能

本文将告诉大家如何在 WinUI 3 或 UNO 里面，仿造 WPF 的 ColumnDefinition SharedSizeGroup 共享列宽功能

<!--more-->
<!-- CreateTime:2024/08/11 07:25:46 -->

<!-- 发布 -->
<!-- 博客 -->

本文的实现代码是大量从 <https://github.com/Qiu233/WinUISharedSizeGroup> 抄的，感谢大佬提供的代码。我在此基础上简化了对 Behavior 的依赖，在本文末尾放上了全部代码的下载方法

实现效果如下：

在界面放入两个 Grid 容器，这两个 Grid 容器分别都有两列，其中第零个 Grid 里面的首列放入一个带背景的 Border 控件，默认情况下宽度被压缩，期望能通过 SharedSizeGroup 的能力共享其他 Grid 的列宽而被撑开。第一个 Grid 里面的首列放入一个按钮，按钮点击的时候修改按钮的宽度，代码如下

```xml
  <Grid local:ColumnSharedSizeHelper.IsSharedSizeScope="true">
    <Grid.RowDefinitions>
      <RowDefinition Height="100"></RowDefinition>
      <RowDefinition Height="100"></RowDefinition>
    </Grid.RowDefinitions>

    <Grid x:Name="Grid1">
      <Grid.ColumnDefinitions>
        <ColumnDefinition></ColumnDefinition>
        <ColumnDefinition Width="*"></ColumnDefinition>
      </Grid.ColumnDefinitions>
      <Border Background="Blue" local:ColumnSharedSizeHelper.SharedSizeGroup="S1"></Border>
    </Grid>
    <Grid Grid.Row="1">
      <Grid.ColumnDefinitions>
        <ColumnDefinition></ColumnDefinition>
        <ColumnDefinition Width="*"></ColumnDefinition>
      </Grid.ColumnDefinitions>
      <Button Width="100" local:ColumnSharedSizeHelper.SharedSizeGroup="S1" Click="Button_OnClick"/>
    </Grid>
  </Grid>
```

如以上代码可以看到添加了名为 ColumnSharedSizeHelper 的辅助类用来提供 IsSharedSizeScope 和 SharedSizeGroup 附加属性，这两个附加属性和在 WPF 中有一点不一样的是不能放入在 ColumnDefinition 里面。现实中我也确实没有想到什么办法可以附加到 ColumnDefinition 里面实现功能。这也就让我仿造的功能比 WPF 弱

在后台代码里面的 `Button_OnClick` 只修改按钮宽度，代码如下

```csharp
    private void Button_OnClick(object sender, RoutedEventArgs e)
    {
        var button = (Button) sender;
        button.Width += 100;
    }
```

运行代码的界面效果如下图

<!-- ![](image/简单在 WinUI 仿造 WPF 的 ColumnDefinition SharedSizeGroup 共享列宽功能/简单在 WinUI 仿造 WPF 的 ColumnDefinition SharedSizeGroup 共享列宽功能0.gif) -->
![](http://cdn.lindexi.site/lindexi%2F%25E7%25AE%2580%25E5%258D%2595%25E5%259C%25A8%2520WinUI%2520%25E4%25BB%25BF%25E9%2580%25A0%2520WPF%2520%25E7%259A%2584%2520ColumnDefinition%2520SharedSizeGroup%2520%25E5%2585%25B1%25E4%25BA%25AB%25E5%2588%2597%25E5%25AE%25BD%25E5%258A%259F%25E8%2583%25BD0.gif)

核心代码是 ColumnSharedSizeHelper 类型，其实现逻辑如下

```csharp
public static class ColumnSharedSizeHelper
{
    // Copy From https://github.com/Qiu233/WinUISharedSizeGroup

    public static readonly DependencyProperty IsSharedSizeScopeProperty =
        DependencyProperty.RegisterAttached("IsSharedSizeScope", typeof(bool), typeof(UIElement), new PropertyMetadata(false));

    private static readonly DependencyProperty SharedSizeGroupProperty =
        DependencyProperty.RegisterAttached("SharedSizeGroup", typeof(string), typeof(UIElement), new PropertyMetadata(null));

    public static void SetIsSharedSizeScope(DependencyObject o, bool group) => o.SetValue(IsSharedSizeScopeProperty, group);
    public static bool GetIsSharedSizeScope(DependencyObject o) => (bool) o.GetValue(IsSharedSizeScopeProperty);

    public static void SetSharedSizeGroup(DependencyObject o, string group)
    {
        o.SetValue(SharedSizeGroupProperty, group);

        if (o is FrameworkElement framework)
        {
            framework.Loaded -= FrameworkOnLoaded;
            framework.Loaded += FrameworkOnLoaded;

            void FrameworkOnLoaded(object sender, RoutedEventArgs e)
            {
                TrySetSize(framework);

                framework.SizeChanged -= Framework_SizeChanged;
                framework.SizeChanged += Framework_SizeChanged;
            }
        }
    }

    private static void Framework_SizeChanged(object sender, SizeChangedEventArgs args)
    {
        if (sender is not FrameworkElement currentFrameworkElement)
        {
            return;
        }

        TrySetSize(currentFrameworkElement);
    }

    private static void TrySetSize(FrameworkElement currentFrameworkElement)
    {
        var sharedSizeGroup = GetSharedSizeGroup(currentFrameworkElement);

        if (string.IsNullOrEmpty(sharedSizeGroup))
        {
            return;
        }

        if (currentFrameworkElement.Parent is not Grid grid)
        {
            throw new InvalidOperationException();
        }

        FrameworkElement p = currentFrameworkElement;
        while (!ColumnSharedSizeHelper.GetIsSharedSizeScope(p))
        {
            if (VisualTreeHelper.GetParent(p) is not FrameworkElement fe)
            {
                return;
            }
            else
            {
                p = fe;
            }
        }

        if (p == currentFrameworkElement)
        {
            return;
        }

        if (!ColumnSharedSizeHelper.GetIsSharedSizeScope(p))
        {
            return;
        }

        var group = p.GetValue(GroupsProperty) as Dictionary<string, ColumnSharedSizeGroup>;
        if (group == null)
        {
            group = new Dictionary<string, ColumnSharedSizeGroup>();
            p.SetValue(GroupsProperty, group);
        }

        if (!group.TryGetValue(sharedSizeGroup, out var columnSharedSizeGroup))
        {
            columnSharedSizeGroup = new ColumnSharedSizeGroup();
            group.Add(sharedSizeGroup, columnSharedSizeGroup);
        }

        columnSharedSizeGroup.Update(currentFrameworkElement);
    }

    public static string GetSharedSizeGroup(DependencyObject o)
    {
        return (string) o.GetValue(SharedSizeGroupProperty);
    }

    public static readonly DependencyProperty GroupsProperty =
        DependencyProperty.RegisterAttached(nameof(ColumnSharedSizeGroup), typeof(Dictionary<string, ColumnSharedSizeGroup>), typeof(UIElement),
            new PropertyMetadata(null));

    class ColumnSharedSizeGroup
    {
        public void Update(FrameworkElement currentFrameworkElement)
        {
            var grid = (Grid) currentFrameworkElement.Parent;
            var value = (int) currentFrameworkElement.GetValue(Grid.ColumnProperty);

            var column = grid.ColumnDefinitions[value];
            if (!_columns.Contains(column))
            {
                _columns.Add(column);
            }
            var adjustments = new List<ColumnDefinition>();
            var width = currentFrameworkElement.ActualWidth + currentFrameworkElement.Margin.Left + currentFrameworkElement.Margin.Right;
            if (width > _columnSize)
            {
                _columnSize = width;
                adjustments.AddRange(_columns);
            }
            else
            {
                adjustments.Add(column);
            }

            foreach (var columnDefinition in adjustments)
            {
                columnDefinition.Width = new GridLength(_columnSize);
            }
        }

        private readonly List<ColumnDefinition> _columns = [];
        private double _columnSize = 0.0;
    }
}
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/48c6e653a28a5f5609738a288b9b34b31f37c18c/UnoDemo/JeawehonawbuWhaikeregaryere) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/48c6e653a28a5f5609738a288b9b34b31f37c18c/UnoDemo/JeawehonawbuWhaikeregaryere) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 48c6e653a28a5f5609738a288b9b34b31f37c18c
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 48c6e653a28a5f5609738a288b9b34b31f37c18c
```

获取代码之后，进入 UnoDemo/JeawehonawbuWhaikeregaryere 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

