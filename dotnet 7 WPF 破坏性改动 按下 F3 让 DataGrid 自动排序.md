# dotnet 7 WPF 破坏性改动 按下 F3 让 DataGrid 自动排序

本文记录在 dotnet 7 下的 WPF 的一个破坏性改动。在 dotnet 7 下的 WPF 支持 DataGrid 在按下 F3 键的时候，自动按照当前所选列进行列自动排序。这将会让原本采用 F3 键进行其他业务逻辑的代码，工作起来有些非预期

<!--more-->
<!-- CreateTime:2022/11/17 8:43:19 -->

<!-- 博客 -->
<!-- 发布 -->

此破坏改动是在此需求提出的： [https://github.com/dotnet/wpf/issues/6737](https://github.com/dotnet/wpf/issues/6737)

在此代码提交里面更改的： [https://github.com/dotnet/wpf/pull/6873](https://github.com/dotnet/wpf/pull/6873)

行为上就是在 DataGrid 获取选中和键盘焦点时，按下 F3 键，将会根据当前选中的列作为排序依据，进行排序。内核实现代码也非常简单，从 [https://github.com/dotnet/wpf/pull/6873](https://github.com/dotnet/wpf/pull/6873) 更改里面可以看到只有几句代码

```csharp
                else if(e.Key == Key.F3)
                {
                    if (Column.CanUserSort)
                    {
                        Column.DataGridOwner.PerformSort(Column);
                        e.Handled = true;
                        return;
                    }
                }
```

此行为是在 dotnet 7 引入的，可以写一点测试代码来确认。先创建一个 WPF 的 dotnet 7 项目，再编辑 csproj 项目文件，设置为支持 dotnet 6 和 dotnet 7 两个框架。多框架的设置详细请看 [让一个 csproj 项目指定多个开发框架 - walterlv](https://blog.walterlv.com/post/configure-projects-to-target-multiple-platforms.html )

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFrameworks>net6.0-windows;net7.0-windows</TargetFrameworks>
    <Nullable>enable</Nullable>
    <UseWPF>true</UseWPF>
  </PropertyGroup>

</Project>
```

接着写一点后台代码用来生成测试数据，代码如下

```csharp
public partial class MainWindow : Window
{
    public MainWindow()
    {
        for (int i = 0; i < 100; i++)
        {
            ModelList.Add(new Model());
        }

        InitializeComponent();
    }

    public ObservableCollection<Model> ModelList { get; } = new ObservableCollection<Model>();
}

public class Model
{
    public Model()
    {
        Name = "Name_" + _count;
        Description = "Description_" + _count;
        Number = _count;

        _count++;
    }

    public string Name { get; set; }
    public string Description { get; set; }
    public int Number { get; set; }

    private static int _count;
}
```

接着在 XAML 上新建一个 DataGrid 使用数据

```xml
<Window x:Class="ChehicemkeNedearfabulemni.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:ChehicemkeNedearfabulemni"
        mc:Ignorable="d"
        Title="MainWindow" Height="450" Width="800"
        x:Name="Root">
    <Grid>
        <DataGrid ItemsSource="{Binding ElementName=Root,Path=ModelList}">
         
        </DataGrid>
    </Grid>
</Window>
```

尝试切换到 dotnet 7 框架，然后运行项目，接着随意选中一项，按下 F3 键，可以看到 DataGrid 被自动排序

尝试切换到 dotnet 6 框架，执行以上步骤，可以看到按下 F3 键，啥都没有发生

这就是 dotnet 7 在 WPF 引入的一个破坏性变更

如果不想要此功能，可以自己通过路由事件吃掉 F3 键，从而不让 DataGrid 排序

```csharp
    protected override void OnPreviewKeyDown(KeyEventArgs e)
    {
        if (e.Key == Key.F3)
        {
            // 自己的业务

            e.Handled = true;
            return;
        }

        base.OnPreviewKeyDown(e);
    }
```

值得一说的是，在 WPF 里面的这个改动本来是为了保持 Windows 的统一性行为。然而在此更改合入 dotnet 7 发布之后，不出意外，有大佬来开喷了

> I understand the change, but why would you add such stuff that reworks the normal usage of F3. F3 is for search in windows platform, if you wanted to add something like this, you could have added a property to disable that behavior.
>
> We also have tri-level multi-column sort on the grid implemented and of course we do not want a keyboard shortcut for it. We have a button and a UI for it.
>
> How about you also apply CTRL + S to close the grid or print preview the grid? Will that make sense, when common use case on all windows applications for CTRL + S is to save?
>
> Please tell me know can I trap this, but bubble it up the chain so main window's F3 will work correctly?

更多请看 [Wpf DataGrid in .NET7 takes away F3 and automatically sorts. - Breaking change. · Issue #7288 · dotnet/wpf](https://github.com/dotnet/wpf/issues/7288 )

我认为在 WPF 这么大的体量下，功能性改动，还是需要谨慎一些的，毕竟众口难调。能使用外部对接的，就尽量不要直接加在框架内。但也有一群人想着在框架内加入各种原本可以在第三方库简单就能实现的功能… 这些都是难以抉择的。因为很难有一些功能让大家都喜欢，特别是一些有选择性的变更，选了 A 一定就会让期望 B 的开发者伤心

现在的 WPF 开发团队还是很能听进话的，在经过了一场激烈的战斗之后，大家都同意这个功能在下个更改版本里面，使用开关控制打开。默认是打开，可以通过开关关闭，而不需要通过本文如此 Hack 的方法关闭

详细请看 [https://github.com/dotnet/wpf/pull/7297](https://github.com/dotnet/wpf/pull/7297)

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/7324f2b12ce498736917f08c9301c31fec455d54/ChehicemkeNedearfabulemni) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/7324f2b12ce498736917f08c9301c31fec455d54/ChehicemkeNedearfabulemni) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 7324f2b12ce498736917f08c9301c31fec455d54
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 7324f2b12ce498736917f08c9301c31fec455d54
```

获取代码之后，进入 ChehicemkeNedearfabulemni 文件夹

更多 WPF 相关博客，请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )