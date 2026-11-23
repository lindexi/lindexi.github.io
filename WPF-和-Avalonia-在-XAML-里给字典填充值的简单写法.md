
# WPF 和 Avalonia 在 XAML 里给字典填充值的简单写法

本文告诉大家一个在 WPF 和 Avalonia 里都很好用的小技巧，在 XAML 里直接给一个字典填充值，把写在条目上的键当作字典的键，把元素本身当作字典的值。借助这个技巧，可以做到不改一行 C# 代码就注册新的界面模板

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

本文内容由人类主导 AI 辅助编写

## 背景

假设我们在做聊天界面，不同工具需要显示不同的卡片：`ReadFileLines` 工具要显示文件内容，其他工具先继续用通用卡片

最直接的写法是在代码里写一大堆 if 判断，但这样每加一个工具就要改一次代码。比较舒服的做法是给模板选择器准备一个字典，键是工具名，值是模板。选择器只负责按工具名去字典里找模板，找不到就回退到通用模板

而字典里的内容，用 XAML 填写比写 C# 更直观，具体布局和绑定都可以直接写在字典的条目上

## 准备数据

演示用的数据模型非常简单，只需要工具名和输出文本

```csharp
public sealed record ToolItem(string ToolName, string OutputText);
```

接下来看 WPF 和 Avalonia 各自的选择器与 XAML 写法，先看 WPF

## WPF 的选择器和字典

WPF 里的选择器继承 `DataTemplateSelector`，重写 `SelectTemplate` 方法，按工具名在字典里查找模板

```csharp
public sealed class WpfSelector : Wpf.DataTemplateSelector
{
    public Dictionary<string, System.Windows.DataTemplate> ToolItemTemplates { get; }
        = new(StringComparer.Ordinal);

    public override System.Windows.DataTemplate? SelectTemplate(object item, System.Windows.DependencyObject container)
        => item is ToolItem tool ? ToolItemTemplates.GetValueOrDefault(tool.ToolName) : null;
}
```

这里的代码做了三件事：定义一个公开的字典属性、初始化字典、在 `SelectTemplate` 里按工具名查字典。其中 `GetValueOrDefault` 是 `Dictionary` 上的扩展方法，查不到键时返回 null，刚好可以交给外面回退到通用模板

需要敲黑板的是这个属性**只有 getter 没有 setter**。初学者容易认为没有 setter 就无法在 XAML 里写入，其实 XAML 并不是给属性赋值，而是拿到 getter 返回的那个字典实例，然后往里面 `Add` 条目。所以要求是 getter 每次都返回同一个已经初始化好的字典实例，千万不能写成每次访问都 `new` 一个新字典

另外，`StringComparer.Ordinal` 决定了键的比较方式，它是区分大小写的，因此 `ReadFileLines` 和 `readfilelines` 在这里是两个不同的键

## WPF 在 XAML 里填充字典

有了上面的属性，就可以在 XAML 里这样填充

```xml
<UserControl.Resources>
    <demo:WpfSelector x:Key="MessageParts">
        <demo:WpfSelector.ToolItemTemplates>
            <DataTemplate x:Key="ReadFileLines">
                <TextBlock Text="{Binding OutputText}" />
            </DataTemplate>
        </demo:WpfSelector.ToolItemTemplates>
    </demo:WpfSelector>
</UserControl.Resources>
```

`<demo:WpfSelector.ToolItemTemplates>` 是属性元素写法，意思是从这里开始，里面的内容都是给 `ToolItemTemplates` 属性的。放在里面的每个 `DataTemplate` 都会变成字典里的一条记录，其中 `x:Key` 提供键，`DataTemplate` 对象本身就是值

换句话说，上面这段 XAML 在逻辑上等价于下面这行 C# 代码

```csharp
selector.ToolItemTemplates["ReadFileLines"] = 那个 DataTemplate 实例;
```

这就是整套写法的核心，能这样写的前提只有两个：属性可以被 XAML 取到（有公开 getter），且属性返回的集合实现了 `IDictionary`。至于值类型是 `DataTemplate` 还是字符串、画刷，都不影响这个规律

顺手提一个 WPF 的坑：WPF 的 `DataTemplate` 里没有 `x:DataType`，`Binding` 是运行时反射绑定，属性名写错了不会在编译时报错，只有运行起来看到一片空白时才会发现

使用的时候，把资源里的选择器挂到 `ContentPresenter` 或 `ItemsControl` 的模板选择器上就可以了

```csharp
var selector = (WpfSelector)probe.Resources["MessageParts"];
var presenter = new Wpf.ContentPresenter
{
    Content = new ToolItem("ReadFileLines", "WPF tool output"),
    ContentTemplateSelector = selector,
};
```

## Avalonia 的选择器和字典

Avalonia 没有 `DataTemplateSelector` 这个类型，它的模板统一抽象为 `IDataTemplate` 接口，由模板自己决定要不要处理某份数据，以及怎么构建界面

```csharp
public sealed class AvaloniaSelector : IDataTemplate
{
    public Dictionary<string, IDataTemplate> ToolItemTemplates { get; }
        = new(StringComparer.Ordinal);

    public bool Match(object? data) => data is ToolItem;

    public Control? Build(object? param) => param is ToolItem tool
        ? ToolItemTemplates.GetValueOrDefault(tool.ToolName)?.Build(param)
        : null;
}
```

这里的字典和 WPF 那边几乎是双胞胎，只是值类型换成了 `IDataTemplate` 接口。`Match` 决定这份数据归不归自己管，`Build` 负责生成界面。因为 Avalonia 的 `DataTemplate` 本身就实现了 `IDataTemplate`，所以 XAML 里写的 `DataTemplate` 可以直接塞进这个字典

在 `Build` 里面，我们并没有自己写布局，而是把活转交给字典里找到的那个模板的 `Build` 方法，这也是选择器与字典分工的体现

这里还有一个把 WPF 和 Avalonia 放进同一个项目才会遇到的小麻烦：两边都有 `Control`、`UserControl`、`DataTemplate` 这些同名类型，写代码时需要用 using 别名区分，比如让 `Ava` 指向 Avalonia、让 `Wpf` 指向 WPF，否则编译器会提示类型有歧义

## Avalonia 在 XAML 里填充字典

Avalonia 这边的 XAML 写法和 WPF 几乎一样

```xml
<UserControl.DataTemplates>
    <demo:AvaloniaSelector>
        <demo:AvaloniaSelector.ToolItemTemplates>
            <DataTemplate x:Key="ReadFileLines" x:DataType="demo:ToolItem">
                <TextBlock Text="{Binding OutputText}" />
            </DataTemplate>
        </demo:AvaloniaSelector.ToolItemTemplates>
    </demo:AvaloniaSelector>
</UserControl.DataTemplates>
```

同样的属性元素写法，同样的 `x:Key` 提供键、`DataTemplate` 提供值。区别在于这里放在 `UserControl.DataTemplates` 集合里，Avalonia 在给内容找模板的时候，会依次询问集合里的每个模板，`Match` 返回 true 的那个就被采用。这就是上面 `AvaloniaSelector` 会被调用的原因

还有一个区别是 `x:DataType="demo:ToolItem"`。Avalonia 支持编译绑定，项目里开启 `AvaloniaUseCompiledBindingsByDefault` 之后，加上 `x:DataType` 就能让 `Binding` 在编译期检查，属性名写错直接编译报错，比 WPF 的运行时绑定更早发现问题。WPF 没有这个标记，所以上面 WPF 的那段 XAML 不需要写

## 这样写真的能跑

需要用到的只有两个包，演示项目里 WPF 和 Avalonia 是放在同一个 net10.0-windows 项目里的

```xml
<TargetFramework>net10.0-windows</TargetFramework>
<UseWPF>true</UseWPF>
<AvaloniaUseCompiledBindingsByDefault>true</AvaloniaUseCompiledBindingsByDefault>
```

```xml
<PackageReference Include="Avalonia" Version="12.1.0" />
```

为了确认这不是只在某种加载方式下才成立的写法，测试项目分别用 WPF 的 STA 线程和 Avalonia 的 Headless 平台加载 XAML，然后从字典里按 `ReadFileLines` 找到模板、渲染出来、读回文本

```csharp
var probe = new WpfProbe();
var selector = (WpfSelector)probe.Resources["MessageParts"];
var item = new ToolItem("ReadFileLines", "WPF tool output");

var presenter = new Wpf.ContentPresenter { Content = item, ContentTemplateSelector = selector };
probe.Content = presenter;
probe.Measure(new System.Windows.Size(400, 200));
probe.Arrange(new System.Windows.Rect(0, 0, 400, 200));
```

Avalonia 那边则直接调 `Build` 拿到控件再看文本

```csharp
var probe = new AvaloniaProbe();
var selector = (AvaloniaSelector)probe.DataTemplates.Single();
var item = new ToolItem("ReadFileLines", "Avalonia tool output");
var content = (Ava.TextBlock)selector.Build(item)!;
content.DataContext = item;
```

两边都能拿到字典里注册的模板并渲染出文本，说明编译后的 XAML 一样会往字典里填值

## 几个小提醒

- 属性只有 getter 不影响 XAML 填充，但 getter 必须返回同一个已初始化的字典实例

- 字典的键不能重复，重复的 `x:Key` 会直接抛异常，而不是后写覆盖前写

- 键的比较方式由字典决定，用了 `StringComparer.Ordinal` 就区分大小写

- 注册模板只是在加载 XAML 时填了一次字典，运行过程中再改字典，已经渲染出来的界面不会自动重新选模板

## 代码获取

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/91994147265ee5c35942e0a373f478995ba37151/WPFDemo/TemplateDictionaryDemo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/91994147265ee5c35942e0a373f478995ba37151/WPFDemo/TemplateDictionaryDemo) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 91994147265ee5c35942e0a373f478995ba37151
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 91994147265ee5c35942e0a373f478995ba37151
```

获取代码之后，进入 WPFDemo/TemplateDictionaryDemo 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。