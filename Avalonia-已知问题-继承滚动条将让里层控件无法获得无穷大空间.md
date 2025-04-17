
# Avalonia 已知问题 继承滚动条将让里层控件无法获得无穷大空间

本文记录 Avalonia 的一个已知问题，如果有代码里面编写一个类型继承 ScrollViewer 类型，然后这个类型里面啥都不做。那将会导致所有放在此滚动条里面的控件无法获取无穷大的空间，其宽高无法撑开，被限定为上层容器尺寸

<!--more-->


<!-- CreateTime:2025/03/12 07:07:26 -->

<!-- 发布 -->
<!-- 博客 -->

此问题能够在 11.2 版本复现

复现步骤如下：

1. 新建一个 FooScrollViewer 类型，让其继承 ScrollViewer 控件
2. 在 FooScrollViewer 里面放入一个 Grid 和一个按钮，点击按钮时设置撑开 Grid 控件

具体代码如下，以下是 FooScrollViewer 类型的代码

```csharp
public class FooScrollViewer : ScrollViewer
{
}
```

以下是 MainView.axaml 界面代码

```xml
    <Grid x:Name="RootGrid" Background="Transparent">
        <views:FooScrollViewer>
            <StackPanel x:Name="RootStackPanel" Margin="0 50 0 0">
                <Grid x:Name="TranslationPanel">
                    <Border x:Name="FooBorder" Height="10" Background="#50565656"></Border>
                    <Button VerticalAlignment="Top" Content="点击" Click="Button_OnClick"></Button>
                </Grid>
            </StackPanel>
        </views:FooScrollViewer>
    </Grid>
```

点击按钮的时候的代码如下

```csharp
    private void Button_OnClick(object? sender, RoutedEventArgs e)
    {
        FooBorder.Height += 200;
    }
```

尝试运行项目，然后不断点击按钮，可以看到 TranslationPanel 这个 Grid 高度已经到达一千六百多。但 RootStackPanel 依然是高度 600 多的值，且此时滚动条不出来

尝试将 `views:FooScrollViewer` 换成 ScrollViewer 控件，则 RootStackPanel 空间可以被撑开，且滚动条出现

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/1479d0e9004e3f0648c9c5ead464227a963b6cb7/AvaloniaIDemo/QeajewiyawarkogearLiqilela) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/1479d0e9004e3f0648c9c5ead464227a963b6cb7/AvaloniaIDemo/QeajewiyawarkogearLiqilela) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 1479d0e9004e3f0648c9c5ead464227a963b6cb7
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 1479d0e9004e3f0648c9c5ead464227a963b6cb7
```

获取代码之后，进入 AvaloniaIDemo/QeajewiyawarkogearLiqilela 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。