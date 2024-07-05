
# UWP WinUI 制作一个路径矢量图标按钮样式入门

本文将告诉大家如何在 UWP 或 WinUI3 或 UNO 里，如何制作一个路径按钮。路径按钮就是使用几何路径轮廓表示内容的按钮，常见于各种图标按钮，或 svg 系贴图矢量图按钮

<!--more-->


<!-- 草稿 -->

在网上有非常多矢量图库，其中免费的图库也非常多，比如 <https://www.iconfont.cn/> 等等。在咱的应用程序里面，可以使用这些矢量图作为按钮的图标，从而更好的进行表意，让界面有更好的设计。使用矢量图还自然带有缩放时依然清晰的功能

最为简单的制作方式就是在按钮里面存放一个 Path 作为内容，比如做一个简单的路径矢量图标按钮

```xml
    <Button HorizontalAlignment="Center" VerticalAlignment="Center">
      <Path Stroke="Black" StrokeThickness="1" Data="M5,5L15,15L5,25"></Path>
    </Button>
```

<!-- ![](image/UWP WinUI 制作一个路径矢量图标按钮样式入门/UWP WinUI 制作一个路径矢量图标按钮样式入门0.png) -->
![](http://image.acmx.xyz/lindexi%2F20247520401696.jpg)

可以看到简单的几行代码就可以实现一个图标按钮的功能了。不过哩作为有追求的开发者，可不能像在树上的小猫一样，咱还需要多加一些需求。比如我希望鼠标移动到按钮上的时候，按钮可以变色，比如说我感觉上面的重复代码多了，即我有多个图标按钮都有大量相似的代码，能不能做一个样式实现这些功能？当然是可以的啦

先在一个资源里面定义按钮的样式，资源可以放在自己的应用业务代码 xaml 文件里面，也可以单独做一个资源字典。本文为了简单，就放在 MainPage.xaml 里面了。如果大家想要放在资源字典里面，别忘了引用资源字典哦

```xml
    <Style x:Key="Style.Button.PathButtonStyle" TargetType="Button">
       ...
    </Style>
```

如上面代码，就定义了一个名为 `Style.Button.PathButtonStyle` 的代码样式。这样的样式命名方法是我习惯用的，因为如此可以方便一级级点下去，特别在有 ReSharper 的帮助下，会更加好用，在样式特别多的时候，这样写能够和 ReSharper 更好的进行配合

这样的样式，可以应用到按钮代码上，如下面代码

```xml
   <Button Style="{StaticResource Style.Button.PathButtonStyle}"
   	 .../>
```

此样式都是给路径图标按钮制作，可以制作非常明确的按钮样式实现。对于 xaml 的界面样式实现的编码思路有些会和 C# 不一样，即不追求抽象性，有很多界面逻辑都是越具体越好，且允许有一些代码是重复的。核心追求就是让界面代码在看的时候可以更好的和界面效果联系起来，按照界面组织的方式走而不是按照逻辑的组织方式走。且有些界面效果是追求界面像，而不追求逻辑合理，即只要界面像就好更重要，当然，能两者都兼顾那是最好的。放心，本文提供的方法还是两者都兼顾的

```xml
    <Style x:Key="Style.Button.PathButtonStyle" TargetType="Button">
      <Setter Property="Template">
        <Setter.Value>
          <ControlTemplate TargetType="Button">
            <Grid x:Name="RootGrid" Background="{TemplateBinding Background}">
              <Path x:Name="ButtonContentPath" StrokeThickness="1" Stroke="Black" Data="M5,5L15,15L5,25"></Path>
            </Grid>
          </ControlTemplate>
        </Setter.Value>
      </Setter>
    </Style>
```

如上面代码，即定义了一个图标按钮的样式，也写明了图标按钮的内容，应用此样式的按钮即可显示出也如上图的效果

样式自然是追求一定的通用性的，上面代码只能显示固定的路径图标，自然不符合咱的需求。能否让 ButtonContentPath 的 Data 参与业务定制？自然是可以的，接下来咱使用简单的附加属性来解决此问题

通过附加属性的方式，既可以用在 UWP 等框架上，同样在 WPF 里面也是可以使用的，毕竟都是相同系列的框架

在后台 cs 代码里面定义一个名为 ButtonHelper 的类型，将在这个类型里面定义附加属性，实现代码如下

```csharp
public class ButtonHelper
{
    public static readonly DependencyProperty ButtonPathProperty = DependencyProperty.RegisterAttached(
        "ButtonPath", typeof(string), typeof(ButtonHelper), new PropertyMetadata(default(string)));
    
    public static void SetButtonPath(DependencyObject element, string value)
    {
        element.SetValue(ButtonPathProperty, value);
    }
    
    public static string GetButtonPath(DependencyObject element)
    {
        return (string)element.GetValue(ButtonPathProperty);
    }
    
    ... // 忽略其他代码
}
```

如此即可在样式里面进行绑定 Data 的内容，核心代码如下

```xml
<Path x:Name="ButtonContentPath" StrokeThickness="2" Stroke="#FF666666" Data="{Binding  RelativeSource={RelativeSource TemplatedParent},Path=(local:ButtonHelper.ButtonPath)}"></Path>
```

如果大家看了本文的内容不知道代码写在哪，可以到本文末尾获取所有代码的下载方法，拉取我的代码跑跑看

通过以上代码，可以看到使用 `(local:ButtonHelper.ButtonPath)` 将 Data 绑定到 ButtonHelper 的 ButtonPath 附加属性上，属性源是通过 `RelativeSource={RelativeSource TemplatedParent}` 指定的，在这里就是按钮本身。以上代码的 `local:` 的 local 表示的 xaml 命名空间，这是因为我将 ButtonHelper 放在和 MainPage 相同的命名空间上，于是就刚好就是 local 的值，如果大家放在其他命名空间，还请在 VisualStudio 的帮助下进行命名空间引用 。以上代码的细节在于必须通过 `RelativeSource` 和 TemplatedParent 指定，且使用 Binding 进行绑定，不能通过 TemplateBinding 和 Source 指定绑定

应用以上样式的按钮，需要在按钮上给 ButtonHelper 的 ButtonPath 附加属性进行赋值，如以下代码

```csharp
    <Button Style="{StaticResource Style.Button.PathButtonStyle}"

local:ButtonHelper.ButtonPath="M5,5L15,15L5,25"
     .../>
```

运行代码也可以看到大概如上图的效果，也就是本文以上提供的三个方式都是实现相同的一个按钮效果。可以看到第一个代码最简单，最后一个代码最有通用性，可以将更多的图标按钮使用样式减少重复的代码

那接下来给样式提出更多的要求，如鼠标移动到按钮上方时，修改按钮的图标颜色

对于 Path 元素来说，可以通过 Stroke 和 StrokeThickness 分别修改轮廓颜色画刷和轮廓线条粗细，可以使用 Fill 属性修改填充画刷

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/2eb5da7c4a63d65e1a2424ca40e2ae94f5da7549/UnoDemo/PathButtonDemo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/2eb5da7c4a63d65e1a2424ca40e2ae94f5da7549/UnoDemo/PathButtonDemo) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 2eb5da7c4a63d65e1a2424ca40e2ae94f5da7549
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 2eb5da7c4a63d65e1a2424ca40e2ae94f5da7549
```

获取代码之后，进入 UnoDemo/PathButtonDemo 文件夹，即可获取到源代码

更多 UWP 或 WinUI3 或 UNO 开发教程，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。