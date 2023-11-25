# WPF 什么时候 VisualTreeHelper.GetDescendantBounds 将返回无穷大

本文将和大家介绍在什么情况下 WPF 将会在调用 VisualTreeHelper.GetDescendantBounds 方法时，返回一个无穷大的范围尺寸

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

在 WPF 的容器控件的里层元素的 RenderTransform 包含 NaN 将会导致对上层容器调用 VisualTreeHelper.GetDescendantBounds 返回无穷大

返回的矩形范围是 -∞,-∞,∞,∞ 的正负无穷大

复现代码如下

```xml
        <Grid x:Name="Grid">
            <Border x:Name="Border">
                <Grid>
                    <Grid.RenderTransform>
                        <TranslateTransform X="NaN"></TranslateTransform>
                    </Grid.RenderTransform>
                </Grid>
            </Border>
        </Grid>
```

此时对以上代码的 Grid 或 Border 控件调用 VisualTreeHelper.GetDescendantBounds 返回的 Rect 就是一个包含正负无穷大的范围，如以下代码

```csharp
    public MainWindow()
    {
        InitializeComponent();

        Loaded += MainWindow_Loaded;
    }

    private void MainWindow_Loaded(object sender, RoutedEventArgs e)
    {
        var rect = VisualTreeHelper.GetDescendantBounds(Grid);
    }
```

以上代码里面存在 RenderTransform 的 TranslateTransform 的 X 或 Y 属性包含了 NaN 值，从逻辑上讲就是具备无穷大的描述范围

本文以上的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/b50873dca1d5f4d118df253f1e7feb326ec1f87e/BeewhearfulidiLodiwena) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/b50873dca1d5f4d118df253f1e7feb326ec1f87e/BeewhearfulidiLodiwena) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin b50873dca1d5f4d118df253f1e7feb326ec1f87e
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin b50873dca1d5f4d118df253f1e7feb326ec1f87e
```

获取代码之后，进入 BeewhearfulidiLodiwena 文件夹