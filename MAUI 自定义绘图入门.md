# MAUI 自定义绘图入门

在2022的5月份，某软正式发布了 MAUI 跨平台 UI 框架。我本来想着趁六一儿童节放假来写几篇关于 MAUI 入门的博客，可惜发现我不擅长写很入门的博客。再加上 MAUI 似乎是为了赶发布日期而发布，只能勉强说能开发了，能用了。于是我就来开始假定大家是一个成熟的 MAUI 开发者了，开始进入复杂控件自绘的自定义绘图渲染的博客

<!--more-->

<!-- 标签：MAUI -->

<!-- 博客 -->
<!-- 发布 -->

在 MAUI 里面，默认将会在 Microsoft.Maui.Dependencies 引用 Microsoft.Maui.Graphics 的负载。在 Microsoft.Maui.Graphics 里，提供了跨平台的独立绘图能力，在 GitHub 上作为独立的开源项目，开源在 [https://github.com/dotnet/Microsoft.Maui.Graphics](https://github.com/dotnet/Microsoft.Maui.Graphics)

也如 Microsoft.Maui.Graphics 在它开源项目里面描述的一样，使用 Microsoft.Maui.Graphics 不会被局限于 MAUI 框架上，可以在任何的应用框架下使用上 Microsoft.Maui.Graphics 库，就和其他的 NuGet 包一样去使用。换句话说，我可以在 WPF 或 WinForms 或者是纯控制台里面使用 Microsoft.Maui.Graphics 进行绘图

另外，我也可以自己注入 Microsoft.Maui.Graphics 的实现定义，扩展其他渲染引擎或框架作为绘图的基础支持

回到主题，本文将告诉大家如何在 MAUI 里面使用 Microsoft.Maui.Graphics 提供的绘图能力进自绘。对于任何的 UI 框架来说，只要能实现好的自绘，就能扩展出超级多炫酷的界面效果，同时也可以方便将旧技术积累迁移到此 UI 框架上。无疑，在MAUI上就实现了这一点

这部分的内容，在当前是 2022.06 还没有多少文档，官方的文档里面都说 MAUI 还是预览版，别停官方说的，在5月就发布了。发布在 [6.0.312](https://github.com/dotnet/maui/releases/tag/6.0.312) 的 dotnet 版本上

在 MAUI 里面接入 Microsoft.Maui.Graphics 从而实现自绘是有框架层的支持的，只是实现的方式稍微有点绕

先安装 VisualStudio 2022 预览版用于新建 MAUI 项目。由于 MAUI 的发布和 VisualStudio 的发布日期对不上，现在只能通过预览版本了，不过后续会合入到正式版本

在新建的项目里面，新建一个类型，让这个类型继承 `Microsoft.Maui.Graphics.IDrawable` 接口。于是此类型即可通过实现 Draw 方法，被框架层调用到，从而在 Draw 方法里面执行绘图。例如和官方的例子一样，将此类型命名为 GraphicsDrawable 如以下代码

```csharp
public class GraphicsDrawable : IDrawable
{
    public void Draw(ICanvas canvas, RectF dirtyRect)
    {
    }
}
```

在 Draw 里面通过 DrawLine 画出一段线条。为了让线条可见，再加上设置线条的颜色和粗细值的代码

```csharp
    public void Draw(ICanvas canvas, RectF dirtyRect)
    {
        canvas.StrokeColor = Colors.Red;
        canvas.StrokeSize = 6;
        canvas.DrawLine(10, 10, 90, 100);
    }
```

完成了这一步之后，还需要将 GraphicsDrawable 接入到 MAUI 框架里面

在 MAUI 框架里提供了 GraphicsView 元素用来对接 Microsoft.Maui.Graphics 的绘图功能。在 GraphicsView 的 Drawable 属性里面，就是用来传入 IDrawable 的对象的

对接的第一步是将咱写的 `GraphicsDrawable` 类型定义成资源，方便后续代码都在 XAML 里面实现。为了演示方便，以下代码都写在 MainPage.xaml 里

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:local="clr-namespace:HejawrawceaJurheakelerela"
             x:Class="HejawrawceaJurheakelerela.MainPage">
    <ContentPage.Resources>
        <local:GraphicsDrawable x:Key="GraphicsDrawable"></local:GraphicsDrawable>
    </ContentPage.Resources>

</ContentPage>
```

还请将以上代码的 `local` 的命名空间更换为你的项目对应的命名空间

接着在 GraphicsView 里使用以上定义的资源，如以下代码

```xml
    <GraphicsView x:Name="GraphicsView" WidthRequest="100" HeightRequest="100" Drawable="{StaticResource GraphicsDrawable}"></GraphicsView>
```

运行程序，即可看到界面画出一条线

![](http://image.acmx.xyz/lindexi%2F202265124335331.jpg)

可以看到调用堆栈如下

![](http://image.acmx.xyz/lindexi%2F20226512326813.jpg)

也就是实际的实现是由 Win2D 提供的

以上是在 Windows 平台上运行的，那既然 MAUI 宣称是跨平台的，那在其他的平台上又是如何

接下来在安卓平台上跑一下

![](http://image.acmx.xyz/lindexi%2F2022651159458266.jpg)

同样也看一下调用堆栈

![](http://image.acmx.xyz/lindexi%2F2022651158524428.jpg)

可以看到调用堆栈和 Windows 平台上，符合预期的不同，也就是说 Microsoft.Maui.Graphics 根据不同的平台选用不同的绘制底层技术

这就是 MAUI 自绘的开始，如何绘制出漂亮的界面就靠大家发挥

试用了几天的 MAUI 发现了比上次我用预览版本有了很大的进步，比如 Windows 端的调试部署极大的提升。当然，这不是 MAUI 团队的努力，而是 Windows App SDK 团队的努力，将原本的 UWP 很多逗比逻辑和交互给优化的。自然，不足之处也是有的，那就是 MAUI 团队还是太小了，好多东西还没磨完，不过这会随着开发的投入逐步完善。现在的 MAUI 已经达到了 Demo 级，小工具级可用的状态。推荐大家要是有什么小工具，选择一下 MAUI 试试水也不错

本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/2da0315302ae504f50c4c3baa47fe3f45d0cdc26/HejawrawceaJurheakelerela) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/2da0315302ae504f50c4c3baa47fe3f45d0cdc26/HejawrawceaJurheakelerela) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 2da0315302ae504f50c4c3baa47fe3f45d0cdc26
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 HejawrawceaJurheakelerela 文件夹

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、 使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
