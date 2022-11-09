# WPF 已知问题 Separator 无法应用 ContextMenu 定义的默认样式

本文记录一个 WPF 已知问题，在 ContextMenu 的 Resources 里定义 Separator 的默认样式，在 ContextMenu 里面的 Separator 将应用不上，或者说不会找到此默认的样式。需要明确给 Style 一个 Key 且在 Separator 写明此 Key 才能应用上

<!--more-->

<!-- 博客 -->
<!-- 发布 -->

如下面的例子，在 ContextMenu 的 Resources 资源里面定义了 Separator 的默认样式

```xml
            <ContextMenu>
                <ContextMenu.Resources>
                    <Style TargetType="Separator">
                        <Setter Property="Margin" Value="10,10,10,10"></Setter>
                    </Style>
                </ContextMenu.Resources>
            </ContextMenu>
```

接着在 ContextMenu 里面存放一个 Separator 元素，可以看到此 Separator 元素没有使用或者说找到定义的样式，视觉上就是 Margin 没有生效

```xml
            <ContextMenu>
                <ContextMenu.Resources>
                    <Style TargetType="Separator">
                        <Setter Property="Margin" Value="10,10,10,10"></Setter>
                    </Style>
                </ContextMenu.Resources>
                <Separator></Separator>
            </ContextMenu>
```

而如果给 Separator 的 Style 加上 Key 且在 Separator 写明了此 Key 那么将可以成功应用上，如下面代码

```xml
            <ContextMenu>
                <ContextMenu.Resources>
                    <Style x:Key="SeparatorStyle" TargetType="Separator">
                        <Setter Property="Margin" Value="10,10,10,10"></Setter>
                    </Style>
                </ContextMenu.Resources>
                <Separator Style="{StaticResource SeparatorStyle}"></Separator>
            </ContextMenu>
```

此问题是在 WPF 代码里面写了特殊判断逻辑，预计是有我没有理解的坑才如此做。感谢 [少珺](https://blog.sdlsj.net/) 工具人帮我找到了在 WPF 框架里面的问题

为了方便说明问题，我将给出可以运行的测试代码，此测试代码可以在本文末尾找到项目的下载

新建一个 WPF 项目，编辑主窗口，添加以下代码

```xml
    <Grid Background="White" MouseDown="Grid_OnMouseDown">
        <Grid.ContextMenu>
            <ContextMenu>
                <ContextMenu.Resources>
                    <Style TargetType="Separator">
                        <Setter Property="Margin" Value="10,10,10,10"></Setter>
                    </Style>
                    <Style x:Key="SeparatorStyle" TargetType="Separator">
                        <Setter Property="Margin" Value="10,10,10,10"></Setter>
                    </Style>
                </ContextMenu.Resources>
                <Menu>1</Menu>
                <Menu>2</Menu>
                <Menu>3</Menu>
                <Separator></Separator>
                <Menu>1</Menu>
                <Menu>2</Menu>
                <Menu>3</Menu>
                <Separator Style="{StaticResource SeparatorStyle}"></Separator>
                <Menu>1</Menu>
                <Menu>2</Menu>
                <Menu>3</Menu>
            </ContextMenu>
        </Grid.ContextMenu>
    </Grid>
```

对应的后台代码如下

```csharp
    private void Grid_OnMouseDown(object sender, MouseButtonEventArgs e)
    {
        var grid = (Grid)sender;
        grid.ContextMenu.IsOpen = true;
    }
```

此时点击窗口内容，即可看到弹出了菜单

弹出的菜单的两条分割线的 Margin 是不相同的

根本原因是在 WPF 里面，对于在 Menu 里面的 Separator 采用的是如下逻辑，以下代码可以从 WPF 官方开源仓库 [https://github.com/dotnet/wpf/blob/1aab9e3f42dbf550797bff97a32f2dbfb61a3198/src/Microsoft.DotNet.Wpf/src/PresentationFramework/System/Windows/Controls/MenuItem.cs#L1344-L1353](https://github.com/dotnet/wpf/blob/1aab9e3f42dbf550797bff97a32f2dbfb61a3198/src/Microsoft.DotNet.Wpf/src/PresentationFramework/System/Windows/Controls/MenuItem.cs#L1344-L1353) 找到

```csharp
                Separator separator = item as Separator;
                if (separator != null)
                {
                    bool hasModifiers;
                    BaseValueSourceInternal vs = separator.GetValueSource(StyleProperty, null, out hasModifiers);
                    if (vs <= BaseValueSourceInternal.ImplicitReference)
                        separator.SetResourceReference(StyleProperty, SeparatorStyleKey);

                    separator.DefaultStyleKey = SeparatorStyleKey;
                }
```

从上面代码可以看到，判断如果样式是小于等于 ImplicitReference 优先级的，那就采用默认的 SeparatorStyleKey 作为样式属性。如果没有在代码里面明确给定资源的 Key 内容，那以上代码的 vs 就是 ImplicitReference 优先级，于是样式就被修改为默认的主题样式

这是在 WPF 里面特别给定的代码，也许是大佬们为了修复某个我理解不了的坑

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/b820847a1af20370de28a1e73e32df9561a98ecc/HayhachujedaKikunayreefee) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/b820847a1af20370de28a1e73e32df9561a98ecc/HayhachujedaKikunayreefee) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin b820847a1af20370de28a1e73e32df9561a98ecc
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin b820847a1af20370de28a1e73e32df9561a98ecc
```

获取代码之后，进入 HayhachujedaKikunayreefee 文件夹

更多 WPF 已知问题请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )