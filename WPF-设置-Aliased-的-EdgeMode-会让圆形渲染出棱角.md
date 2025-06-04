
# WPF 设置 Aliased 的 EdgeMode 会让圆形渲染出棱角

本文记录 WPF 的 RenderOptions EdgeMode 的 Aliased 选项时的效果

<!--more-->


<!-- CreateTime:2025/05/31 07:26:00 -->

<!-- 发布 -->
<!-- 博客 -->

在后台代码设置时，使用的是 `RenderOptions.SetEdgeMode(this, EdgeMode.Aliased);` 的方式

在 XAML 代码设置时，使用的是 `RenderOptions.EdgeMode="Aliased"` 的方式

按照 [官方文档所述](https://learn.microsoft.com/en-us/dotnet/api/system.windows.media.edgemode?view=windowsdesktop-9.0 ) 正常文本采用 Aliased 方式显示，但如椭圆或曲线多边形等就应该是采用 Unspecified 方式

测试代码分别如下

```csharp
public class Foo : Control
{
    public Foo()
    {
        RenderOptions.SetEdgeMode(this, EdgeMode.Aliased);
        Width = 300;
        Height = 300;
    }

    protected override void OnRender(DrawingContext drawingContext)
    {
        var size = 6.35;
        drawingContext.DrawEllipse(Brushes.Black, null, new Point(100, 100), size, size);
    }
}
```

```xml
<Ellipse Margin="10,10,10,10" Fill="Black" Width="10" Height="10" RenderOptions.EdgeMode="Aliased" HorizontalAlignment="Left" VerticalAlignment="Top"/>
```

可以尝试运行一下我的代码，试试效果

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/3b7d260174b02542e887ca2354cc41b4b8599486/WPFDemo/QarwicukeNehifakajaycair) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/3b7d260174b02542e887ca2354cc41b4b8599486/WPFDemo/QarwicukeNehifakajaycair) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 3b7d260174b02542e887ca2354cc41b4b8599486
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 3b7d260174b02542e887ca2354cc41b4b8599486
```

获取代码之后，进入 WPFDemo/QarwicukeNehifakajaycair 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。